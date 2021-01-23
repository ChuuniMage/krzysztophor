# roleUtils.ts - The Role Utilities

[<< Back to Project Overview](../defenderProject.md)

[< back to argUtils.ts - The Argument Utilities](argUtils.md)

We have already seen `memberHasAllRolesById`, which has remained unchanged since the initial commit to production release.

```typescript
    export const memberHasAllRolesById = (testedMember:Discord.GuildMember, roleIdArray:string[]):boolean => {
      let matchGoal = roleIdArray.length;
      let runningCounter = 0;
      let rolesArray = testedMember.roles.cache.array();
        for (let i = 0; i < roleIdArray.length; i++){
        let currentRoleNameTested = roleIdArray[i];
        for (let a = 0; a < rolesArray.length; a++){
          let testedRoleName = rolesArray[a].id;
            if (testedRoleName === currentRoleNameTested){
              runningCounter++;
              }
            }
          }
          return runningCounter === matchGoal;
        }
 ```
In the initial commit, we have two similar functions:
- `memberHasAnyRoleByName`, which tests if a server member has any one of the given input roles by their name, not their ID. The function fetchies an array of all of the roles that the tested member has. This function is already utilised in the [Message Initialisation](../initialisationAndOnMessage.md) process to test if the member has permissions to use the bot.
- `memberHasAllRolesByName`, which tests if the user had all of the roles given. This function was written for completeness sake, but was never called. In the production release, this function was removed.

```typescript
export const memberHasAnyRoleByName = (
testedMember:Discord.GuildMember, 
roleNameArray:string[]):boolean => {
  let runningCounter = false;
  let rolesArray = testedMember.roles.cache.array();
    for (let i = 0; i < roleNameArray.length; i++){
    let currentRoleNameTested = roleNameArray[i];
     for (let a = 0; a < rolesArray.length; a++){
       let testedRoleName = rolesArray[a].name;
        if (testedRoleName === currentRoleNameTested){
          runningCounter = true;
          }
        }
      }
      return runningCounter;
    }

export const memberHasAllRolesByName = (
testedMember:Discord.GuildMember, 
roleNameArray:string[]):boolean => {
  let matchGoal = roleNameArray.length;
  let runningCounter = 0;
  let rolesArray = testedMember.roles.cache.array();
    for (let i = 0; i < roleNameArray.length; i++){
    let currentRoleNameTested = roleNameArray[i];
    for (let a = 0; a < rolesArray.length; a++){
      let testedRoleName = rolesArray[a].name;
        if (testedRoleName === currentRoleNameTested){
          runningCounter++;
          }
        }
      }
      return runningCounter === matchGoal;
    }
 ```
 
In the initial commit, the next two functions, `applyRoleByIdToUser` and `removeRoleByIdFromUser` applied and removed roles from a user respectively. These were key functions since a applying roles and removing roles from users is the primary task of the bot. These functions are wrappers on the `roles.add()` and `roles.remove()` methods on the Guildmember object, so that we can pass
 
 ```typescript
export const applyRoleByIdToUser = async (
  inputRoleId:string, 
  inputUserId:string, 
  inputGuildObject:Discord.Guild):Promise<void> => {
  let currentGuildMemberUser = await inputGuildObject.members.fetch(inputUserId);
  if (memberHasAllRolesById(currentGuildMemberUser, [inputRoleId])) {return}
  currentGuildMemberUser.roles.add(inputRoleId);
};

// removeRoleByIdFromUser is identical, except for the last two lines:
  if (!memberHasAllRolesById(currentGuildMemberUser, [inputRoleId])) {return}
  currentGuildMemberUser.roles.remove(inputRoleId);

 ```
 
We needed functions exactly like these, but which took role names as parameters. However, the `roles.add()` and `roles.remove()` methods only took Role IDs as erguments - so the intermediary function, `getRoleByName` was written, which asynchronously fetches the `Discord.Role` object according to a name given to.
 
 ```typescript
 export const getRoleByName = async (
 roleName: string, 
 guildObject:Discord.Guild):Promise<Discord.Role> => {

    let roleToBeReturned = await guildObject.roles.fetch().then(roles => {
      return roles.cache.find((_role) => _role.name == roleName ) as Discord.Role;
    });
  
    if (roleToBeReturned === undefined) {
      console.log('Cannot get role!')
    } else {
      return roleToBeReturned;
      }
  }
 ```
 
 So, after this was written, implementing `applyRoleByNameToUser` and `RemoveRoleByNameFromUser` was straightforward.
 
```typescript
// applyRoleByNameTouser
  if (memberHasAllRolesByName(currentGuildMemberUser, [inputRoleName])) {return}
  currentGuildMemberUser.roles.add((await getRoleByName(inputRoleName, inputGuildObject)));
  
// removeRoleByNameFromUser
  if (!memberHasAllRolesByName(currentGuildMemberUser, [inputRoleName])) {return}
  currentGuildMemberUser.roles.remove((await getRoleByName(inputRoleName, inputGuildObject)));

```
 
For the production release, I had figured that impornt four very similar functions, and needing to remember all of them was inefficient. So, I rewrote them with the following structure:
- Just like how writing the same function four times isn't ideal, writing the same function twice is also not ideal, so I wrote `updateUserRoleCurry` to give a branching path to the applyRole and removeRole. `Add` and `Remove` are fed into the function to produce `addRoleToUser` and `removeRoleFromUser`

 ```typescript
   let updateUserRoleCurry = 
  (addOrRemove:"Add"|"Remove") => 
  async (inputGuildObject:Discord.Guild, 
    inputUserId:string, 
    inputRoleId:string):Promise<void> => {

    let currentGuildMemberUser:Discord.GuildMember = await inputGuildObject.members.fetch(inputUserId)
    
    switch (addOrRemove){
      case 'Add':
        currentGuildMemberUser.roles.add(inputRoleId);
      break;
      case 'Remove':
        currentGuildMemberUser.roles.remove(inputRoleId)
      break;
    }
  }
  
  export let addRoleToUser = updateUserRoleCurry('Add');
  export let removeRoleFromUser = updateUserRoleCurry('Remove');
  ```
Next, I defined two objects: `addRole` and `removeRole`, with two methods on each of them: `byId` and `byName`. Now, all `byName` does is await the Role ID, and call the same `addRoleToUser` function as the `byId` method.
  
```typescript
let addRole = {
  byId: function(inputGuildObject:Discord.Guild, inputUserId:string, inputRoleId:string){
    addRoleToUser(inputGuildObject, inputUserId, inputRoleId);
  },
  byName: async function(inputGuildObject:Discord.Guild, inputUserId:string, inputRoleName:string){
    let inputRoleId = (await getRoleByName(inputGuildObject, inputRoleName)).id;
    addRoleToUser(inputGuildObject, inputUserId, inputRoleId);
  }
}
// the removeRole 
```

These two objects are then put into a final object, `updateUserRole`, making this object a carrier for my four functions.

```typescript
export const updateUserRole = {
  addRole,
  removeRole,
}
```

Now, with full intellisense support, I can simply refer to `updateUserRole.addRole.byName`, to do the same job as `applyRoleToUserByName`, but with the intellisense doing more of the cognitive heavy lifting for me, and collecting the functions into the `updateUserRole` object.



The story of this last function deserves its own page: iterateOverMembersAndReturnData

[>> The story of iterateOverMembersAndReturnData](iterate.md)
