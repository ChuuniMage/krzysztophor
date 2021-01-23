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
  if (memberHasAllRolesById(currentGuildMemberUser, [inputRoleId])) {return}
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
 
 So, after this `applyRoleByNameToUser` and `RemoveRoleByNameFromUser`
 


[>> chanUtils.ts - The Channel Utilities](chanUtils.md)
