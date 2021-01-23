# The story of iterateOverMemebersAndReturnData

[<< Back to Project Overview](../defenderProject.md)

[< Back to roleUtils.ts - The Role Utilities](roleUtils.md)

I was very proud of this function from the initial commit. However, it didn't make the cut for the release of the bot. I had a common task - Iterating over members, and returning data, and so I had defined a function that did exactly this.

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


[>> chanUtils.ts - The Channel Utilities](chanUtils.md)
