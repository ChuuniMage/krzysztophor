# argUtils.ts - the Argument Utilities

[<< Back to Project Overview](defenderProject.md)

One of the three utilities files the bot implements is argUtils.ts, which contains two utility functions for parsing arguments.

The first function, `extractNumbersForId` is unchanged from the initial commit, to the production release.
- The function returns undefined if given an undefined input, to prevent crash in this event.
- A regular expression is defined to match digits, numbers from 0-9.
- The `match` method on string objects returns an array of characters that fit the regular expression's parameters
- If the array is not null, then it returns a string formed from the `parsedInput`, with the `join()` method.

```typescript
export const extractNumbersForId = (userInput:string|undefined):string|undefined => {
  if (userInput == undefined) {
    return undefined;
  }
    let regExForDigits = /\d+/g;
    let parsedInput = userInput.match(regExForDigits);
    if(parsedInput != null) {
    return parsedInput.join('');
  }
}
```

In the previous post, we had the `memberHasRolesFromArgs` function extracted from the message event. 

```typescript
  let memberHasRolesFromArgs = (
    inputMemberCache: Discord.GuildMember
  ): boolean => {
    let memberRolesCache = inputMemberCache.roles.cache;

    let fullRolesCount = args.length;
    let hasRolesCount = 0;
    for (let i = 0; i < args.length; i++) {
      let currentMemberRoleId = extractNumbersForId(args[i]);
      let hasRoleCheck = memberRolesCache.has(currentMemberRoleId);
      if (hasRoleCheck) {
        hasRolesCount++;
      }
    }
    return hasRolesCount === fullRolesCount;
  };
```

However, this function wasn't retained - instead, 


```typescript
export let returnIdArrayFromArgs = (inputArgs:string[]):string[] => {
  let outputIdArray:string[] = [];
  for (let i = 0; i < inputArgs.length; i++) {
    outputIdArray.push(extractNumbersForId(inputArgs[i]))
  }
  return outputIdArray;
}
```
[>> The Bot Commands](botCommands.md)
