# roleUtils.ts - The Role Utilities

[<< Back to Project Overview](../defenderProject.md)

[< back to argUtils.ts - The Argument Utilities](argUtils.md)

We have already seen the initial commit's `memberHasAllRolesById`.

What this function does, is:
- Set the `matchGoal`, which is the amount of roles that need to be found on the `testedMember`
- Initialise the `runningCounter` to zero 
- Convert the mapped member role objects into a `rolesArray`, since at the time of coding this, I was confident with manipulating arrays
- Implement two `for` loops, one through each role ID in the `roleIdArray` to test each one of them against against all of the roles the member has. The second `for` loop iterates over each of those member's roles.
- If there is a matching role name, the `runningCounter` is icremented.
- The function returns true if the member has all of the roles fed into the function, and false if they do not.

```typescript
    export const memberHasAllRolesById = (
    testedMember:Discord.GuildMember, 
    roleIdArray:string[]):boolean => {
    
      let matchGoal = roleIdArray.length;
      let runningCounter = 0;
      let rolesArray = testedMember.roles.cache.array();
        for (let i = 0; i < roleIdArray.length; i++){
        let currentRoleIdTested = roleIdArray[i];
        for (let a = 0; a < rolesArray.length; a++){
          let testedRoleId = rolesArray[a].id;
            if (testedRoleId === currentRoleIdTested){
              runningCounter++;
              }
            }
          }
          return runningCounter === matchGoal;
        }
 ```
In the initial commit, we have two similar functions:
- `memberHasAnyRoleByName`, which tests if a server member has any one of the given input roles by their name, not their ID. For this logic check, the `runningCounter` is changed to a `runningBoolean`, and if any of the input roles are found, that boolean is set to true.  This function is used in [Message Initialisation](../initialisationAndOnMessage.md) to verify if the member has permissions to use the bot.
- `memberHasAllRolesByName`, which tests if the user had all of the roles given by name. This function was written for completeness sake, but was never called. In the production release, this function was removed.

```typescript
export const memberHasAnyRoleByName = (
testedMember:Discord.GuildMember, 
roleNameArray:string[]):boolean => {
  let runningBoolean = false;
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
      return runningBoolean;
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
`memberHasAllRolesById` and `MemberHasAnyRoleByName` have been retained, with the following improvements:
- Letting the map of user Roles stay a map, instead of converting to an array.
- Instead of nested `for` loops, instead we use two `.forEach()` method calls, which are identical for our purposes. This has reduced the amount of clutter & boilerplate needed to implement both functions.
 
 
 ```typescript
 export const memberHasAllRolesById = (
  testedMember:Discord.GuildMember, 
  roleIdArray:string[]):boolean => {

  let matchGoal:number = roleIdArray.length;
  let runningCounter:number = 0;

  testedMember.roles.cache.forEach(memberRole => {
    roleIdArray.forEach(inputRoleTested => {
      if (memberRole.id === inputRoleTested){
        runningCounter++;
        }
    })
  });
      return runningCounter === matchGoal;
    }

export const memberHasAnyRoleByName = (
  testedMember:Discord.GuildMember, 
  roleNameArray:string[]):boolean => {

  let memberHasRoleBool:boolean = false;

  testedMember.roles.cache.forEach(memberRole => {
    roleNameArray.forEach(roleNameTested => {
      if (memberRole.name === roleNameTested){
          memberHasRoleBool = true;
          }
        })
      });
      return memberHasRoleBool;
    }
 ```
 
The next two functions in the initial commit, `applyRoleByIdToUser` and `removeRoleByIdFromUser` applied and removed roles from a users. These functions are effectively wrappers on the `roles.add()` and `roles.remove()` methods from the `GuildMember` object, so that we can pass. 

These functions also check if the user already has the role if they are being added a role, or if they already don't have the role if the role is being removed. 
 
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
 
We needed functions exactly like these, but which took role names as parameters. However, the `roles.add()` and `roles.remove()` methods only took Role IDs as erguments - so the intermediary function, `getRoleByName` was written, which asynchronously fetches the `Discord.Role` object according to the `roleName `input.
 
 ```typescript
 export const getRoleByName = async (
 roleName: string, 
 guildObject:Discord.Guild):Promise<Discord.Role> => {

    let roleToBeReturned = await guildObject.roles.fetch().then(roles => {
      return roles.cache.find((testedRole) => testedRole.name == roleName ) as Discord.Role;
    });
  
    if (roleToBeReturned === undefined) {
      console.log('Cannot get role!')
    } else {
      return roleToBeReturned;
      }
  }
 ```
 
Implementing `applyRoleByNameToUser` and `RemoveRoleByNameFromUser` was straightforward after this was written. 
 
```typescript
// applyRoleByNameTouser
  if (memberHasAllRolesByName(currentGuildMemberUser, [inputRoleName])) {return}
  currentGuildMemberUser.roles.add((await getRoleByName(inputRoleName, inputGuildObject)));
  
// removeRoleByNameFromUser
  if (!memberHasAllRolesByName(currentGuildMemberUser, [inputRoleName])) {return}
  currentGuildMemberUser.roles.remove((await getRoleByName(inputRoleName, inputGuildObject)));

```
 
For the production release, I had figured that having four very similar functions, and needing to remember all of them was inefficient. So, I rewrote them with the following structure:
- Instead of explicating that the functions add or remove roles by ID in their role name they simply add or remove roles
- The checks for whether the user already had the rule for add, or already didn't have the role for remove, were removed for the production release. I had discovered the `Discord.js` API is smart enough to handle superfluous additions or removals of roles correctly.

 ```typescript
  export let addRoleToUser = async (
    inputGuildObject:Discord.Guild, 
    inputUserId:string, 
    inputRoleId:string):Promise<void> => {

    let currentGuildMemberUser:Discord.GuildMember = await inputGuildObject.members.fetch(inputUserId)
    currentGuildMemberUser.roles.add(inputRoleId);
 }
    
    //The only change for the removeRoleFromUser function
    currentGuildMemberUser.roles.remove(inputRoleId);
  ```
Next, I defined two objects: `addRole` and `removeRole`, with two methods on each of them: `byId` and `byName`.
   - `byId` is identical to just calling the `addRoleToUser` function
   - `byName` takes the role name as a function, and asynchronously fetches the role's ID, which is then fed into the `addRoleToUser` role.
  
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

Now, with full intellisense support, I can simply refer to this single `updateUserRole` object, and it will show me the `addRole` and `removeRole` properties, and their associated methods `byId` and `byName`. Now, if I wanted to remove a role from a user by name, I would type: `updateUserRole.addRole.byName` with the appropriate arguments.

Two functions were added in the production release:
- `hasDefaultPFP`, which tests if a user's profile picture is the default discord profile picture. This function is used later in the `checkPFP` command, which applies the `Change PFP` role to a user, quarantining them away from the rest of the server until they have a non-default display picture. This is a check against freshly made spam & troll accounts. 
- `returnRoleIdNameArrayToPost`, which utilises the `.map()` method to wrap each role ID given to the function in `<@&` on the left, and `>` on the right, since in discord messages, this is how role IDs are formatted under the hood. To the user, `<@&123456789012345>` typed into a discord chatroom will display as `@ThisIsARole`.

```typescript
export const hasDefaultPFP = (inputMember:Discord.GuildMember) => {
  let defaultAvatar:string = inputMember.user.defaultAvatarURL;
  let displayAvatar:string = inputMember.user.displayAvatarURL();
  
  return defaultAvatar === displayAvatar
} 

export const returnRoleIdNameArrayToPost = (inputIDArray:string[]) => {
  return inputIDArray.map((roleId) => {return "<@&" + roleId + ">"})
}
```

There is one final function that was in the initial commit, but wasn't in the production release, and that's `iterateOverMembersAndReturnData`. The story of this last function deserves its own page.

[>> The iterateOverMembersAndReturnData Function](iterate.md)
