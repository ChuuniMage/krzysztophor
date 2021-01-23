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
 
 The `getRoleByName` function gets roles by its name idk what to tell you
 
 ```typescript
 export const getRoleByName = async (roleName: string, guildObject:Discord.Guild):Promise<Discord.Role> => {

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
 


[>> chanUtils.ts - The Channel Utilities](chanUtils.md)
