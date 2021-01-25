# argUtils.ts - the Argument Utilities

[<< Back to Project Overview](../defenderProject.md)

[< Back to chanUtils.ts - The Channel Utilities](chanUtils.md)

One of the three utilities files the bot implements is argUtils.ts, which contains two utility functions for parsing arguments.

The first function, `extractNumbersForId` is unchanged from the initial commit:
- The function returns undefined if given an undefined input, to prevent crash if given an undefined input.
- `regExForDigits`, is defined, a regular expression is defined to match numerals from 0-9.
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

This was clutter in the main index file, and relied too heavily on referencing global variables, so I had to find away to extract and change it. Ultimately, the function wasn't retained - I already had a utility function that performs an almost identical task, `memberHasAllRolesById`. All this function needs is the tested Discord member, and an array of IDs to see if they have the roles. I will go into detail how this function works in the following post.

```typescript
export const memberHasAllRolesById = (
  testedMember:Discord.GuildMember, 
  roleIdArray:string[]):boolean => {

  let matchGoal:number = roleIdArray.length;
  let runningCounter:number = 0;

  testedMember.roles.cache.forEach(memberRole => {
    roleIdArray.forEach(inputRoleTested => {
      if (memberRole.id === inputRoleTested){
        runningCounter++;
        }
    })
  });
      return runningCounter === matchGoal;
    }
```

With this in mind, the answer to the problem is not a custom function that reads global state, but a function of smaller scope that takes an array of arguments and returns an array of IDs extracted from the arguments. This is very easy to do, with the `.map()` method, since we just map `extractNumbersForId` over each inputARg in the array fed into `returnIdArrayFromArgs`.

```typescript
export let returnIdArrayFromArgs = (inputArgs:string[]):string[] => {
  return inputArgs.map((inputArg) => extractNumbersForId(inputArg))
}
```

Since we've already gotten a sneak peek at the Role Utilities with `memberHasAllRolesById`, we will cover the rest of the Role Utilities in the next post.

[>> roleUtils.ts - The Role Utilities](roleUtils.md)
