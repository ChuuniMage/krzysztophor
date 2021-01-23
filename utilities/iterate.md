# The iterateOverMembersAndReturnData Function

[<< Back to Project Overview](../defenderProject.md)

[< Back to roleUtils.ts - The Role Utilities](roleUtils.md)

I was very proud of this function from the initial commit. However, it didn't make the cut for the release of the bot. I had a common task - Iterating over members, and returning data, and so I had defined a function that did exactly this.

The structure of this function is rather complex, starting from its inputs:
- `dataToUpdate`, intentionally an `any` type in order to accomodate for any sort of data that might be fed through the function
- `inputGuild`, which is the Discord guild the members are part of
- `fulfillConditionToUpdateData`, a boolean check on the members to determine whether to execute `dataUpdater` or not
- `dataUpdater`, a callback function, which takes two arguments - the Discord member, and the data that requires updating.

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

The theory behind this function was to have a flexible, pre-structured function that any data could be fed through, and callbacks could be designed to work with. It was implemented on four bot commands, `whois`, `howmanyare`, `checkPFP`, and `replaceall`. The details of the implementations of these functions, both in the initial commit and in the production release, will be covered in future posts. 

For now, I will just make note of inputs I was feeding into this function.

```typescript
replaceall:
   iterateOverMembersAndReturnData(
     undefined,
     currentGuildObject,
     hasRoleToReplace, // Boolean to check if the user has a role - 
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
     whoHasRoles, // This is an empty array
     currentGuildObject, // The current server
     memberHasRolesFromArgs, // The function we covered in argUtils.ts
     updateArrayForHowManyAre // New array update callback, specifically for this command
   )
     
 howmanyare:
   iterateOverMembersAndReturnData(
    returnUsers, // This is an empty array of names
    currentGuildObject,
    memberHasRolesFromArgs,
    updateArrayForWhoIs // Another update array callback, specifically for the data of this command
)

```

- For `replaceall` and `checkpfp`, I took advantage of the function's structure by feeding in `undefined` into the `dataInput` field, in what I *thought* was a clever way to do effects with a function that supposed to return data, but the longer that I worked on the project, the more absurd and superfluous the solution seemed, and the more problems this caused.
- In the `replaceall` command, the problem with `replaceTheRole` was absurd - since the `dataUpdater` callback function parameters are different, I have to wrap `removeRoleByIdFromUser` and `AddRoleByIdToUser` in a new `replaceTheRole` function, making superfluous functions for a single use case. Not only that, but one of the inputs fed into this function must be undefined! 
- An identical issue plagued the `checkpfp` command, where `applyCheckPFPRole` is just a wrapper on `addRoleByNameToUser`, which takes an undefined input as an argument and does nothing with it.
- For `whois` and `howmanyare`, the `memberHasRolesFromArgs` function was defined in the Message Initialisation portion of the program, and relied on global state. This was the only way to prevent me from having to write identical custom callbacks for the scope of each function. The solution to this problem was covered in the [Argument Utilities](utilities/argUtils.md) post.
- Two custom array updating functions, `updateArrayForHowManyAre` and `updateArrayForWhoIs`, which are only used once, in the context of both of these functions. This is superfluous function creation, and just increased the boilerplate needed in order to get these functions off of the ground.


Ultimately, this function was scrapped, since instead of providing flexibility and abstraction, it introduced unnecessary rigidity and verbosity in the implementation of each solution, especially since in half of the use cases I was just feeding it null data input, just to iterate over the members! 

In the next post, I cover the beginning of the program where the bot commands are executed.

[>> The Bot Commands](../botCommands.md)
