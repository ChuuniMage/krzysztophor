# The story of iterateOverMemebersAndReturnData

[<< Back to Project Overview](../defenderProject.md)

[< Back to roleUtils.ts - The Role Utilities](roleUtils.md)

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
