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


[>> chanUtils.ts - The Channel Utilities](chanUtils.md)
