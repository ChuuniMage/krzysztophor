# The story of iterateOverMemebersAndReturnData

[<< Back to Project Overview](../defenderProject.md)

[< Back to roleUtils.ts - The Role Utilities](roleUtils.md)

I was very proud of this function from the initial commit. However, it didn't make the cut for the release of the bot. I had a common task - Iterating over members, and returning data, and so I had defined a function that did exactly this.

The structure of this function is rather complex, starting from its inputs:
- It has been intentionally designed to take any input as its data to update, explicitly making use of mutating data
- The input discord guild that I am iterating members over
- A boolean condition to fulfill \
- A function that takes in 

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
