# argUtils.ts - the Argument Utilities

[<< Back to Project Overview](../defenderProject.md)

[< Back to Utilities](../utilities.md)

One of the three utilities files the bot implements is argUtils.ts, which contains two utility functions for parsing arguments.

The first function, `extractNumbersForId` is unchanged from the initial commit:
- The function returns undefined if given an undefined input, to prevent crash if given an undefined input.
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

In the previous post, we had the `memberHasRolesFromArgs` function extracted from the message event. This function, used for the `howmanyare` and `whois` bot Commands, which both take in an arbitrary amount of roles from arguments, and checks if a member has all of those roles, works in the following way:
- Takes a Discord member as an argument
- The length of the global `args` variable array defines the amount of roles expected to be found on the member in `fullRolesCount`, and initialises a running counter `hasRolesCount` to match against this expected amount.
- A `for` loop is defined to iterate over every role in `args`, and increment the counter for every role of those that is found on the user.
- Finally, function returns a boolean: true if the user has all of the roles given to the function, false otherwise.

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

This was clutter in the main index file, and relied too heavily on referencing global variables, so I had to find away to extract and change it. Ultimately, the function wasn't retained - this is because I already had an utility function that performs an almost identical task, the `memberHasAllRolesById` function, in `roleUtils.ts`

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
All this function needs is the tested Discord member, and an array of IDs to see if they have the roles.

With this in mind, the answer to the problem is not a custom function that reads global state, but a function of smaller scope that takes an array of arguments and returns an array of IDs extracted from the arguments. This is very easy to do, since we use our previously defined `extractNumbersForId` function and place it here.

```typescript
export let returnIdArrayFromArgs = (inputArgs:string[]):string[] => {
  let outputIdArray:string[] = [];
  for (let i = 0; i < inputArgs.length; i++) {
    outputIdArray.push(extractNumbersForId(inputArgs[i]))
  }
  return outputIdArray;
}
```

In the next post, we will cover the setup of the Bot Commands.

[>> The Bot Commands](botCommands.md)
