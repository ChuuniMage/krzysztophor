# The iterateOverMemebersAndReturnData function

[<< Back to Project Overview](../defenderProject.md)

[< Back to roleUtils.ts - The Role Utilities](roleUtils.md)

I was very proud of this function from the initial commit. However, it didn't make the cut for the release of the bot. I had a common task - Iterating over members, and returning data, and so I had defined a function that did exactly this.

The structure of this function is rather complex, starting from its inputs:
- `dataToUpdate`, intentionally typed as the `any` type in order to be as flexible as possible
- `inputGuild` which is the Discord guild
- `fulfillConditionToUpdateData`, a boolean check on the members to determine whether to execute `dataUpdater` or not
- The `dataUpdater` callback function, which takes two arguments - the Discord member, and the data that requires updating.

```typescript
export const iterateOverMembersAndReturnData = 
async (dataToUpdate:any, 
inputGuild:Discord.Guild, 
fulfillConditionToUpdateData:Function, 
dataUpdater:Function):Promise<any> => {

 let members = await inputGuild.members.fetch();
 members.forEach((_member) => {
   let isConditionFulfilled: boolean = fulfillConditionToUpdateData(_member);
   if (isConditionFulfilled) {
     dataUpdater(_member, dataToUpdate);
   }
 });
 return dataToUpdate;
}
```

Was feeding in null values, and using it to update user information. While cute, the 

Ultimately, this function was scrapped, since instead of providing flexibility and abstraction, it introduced unnecessary rigidity in the direct implementation of each solution, especially since in half of the use cases I was just feeding it null data input! It's replaced with specific implementations of the `.foreach()` method.

These are the ways the function was being called:
```typescript
replaceall:
   iterateOverMembersAndReturnData(
     undefined,
     currentGuildObject,
     hasRoleToReplace,
     replaceTheRole
   )
   
checkpfp:
  iterateOverMembersAndReturnData(
    undefined,
    currentGuildObject,
    defaultAvatarCheck,
    applyCheckPFPRole
  );
        
whois:
   iterateOverMembersAndReturnData(
     whoHasRoles,
     currentGuildObject,
     memberHasRolesFromArgs,
     updateArrayForHowManyAre
   ).then((returnedRoleArray) => {
     message.channel.send(
       `Number of users with the following roles[${args.join(", ")}] : ${
         returnedRoleArray.length
       }`
     );
     
 howmanyare:
   iterateOverMembersAndReturnData(
    returnUsers,
    currentGuildObject,
    memberHasRolesFromArgs,
    updateArrayForWhoIs
  ).then((returnedArray) => {
    let computedPost = returnedArray.join(", ");
    let tooManyUsersToReturn = computedPost.length > 2000;
    if (!tooManyUsersToReturn) {
      message.channel.send(`${computedPost}`);
    } else {
      fs.writeFileSync("./whoIsFile.txt", computedPost);
      let fileToAttach = new Discord.MessageAttachment("./whoIsFile.txt");
      message.channel.send(
        `Error! There are too many users that match your search! File has been generated with list of users.`,
        fileToAttach
      );


[>> The Bot Commands](../botCommands.md)
