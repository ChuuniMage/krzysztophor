# argUtils.ts - the Argument utilities

The finished utilies are as such:

This function in the initial commit didn't have the `if (userInput == undefined)` check, leading to crashes if it was fed nothing.

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

```typescript
export let returnIdArrayFromArgs = (inputArgs:string[]):string[] => {
  let outputIdArray:string[] = [];
  for (let i = 0; i < inputArgs.length; i++) {
    outputIdArray.push(extractNumbersForId(inputArgs[i]))
  }
  return outputIdArray;
}
```
