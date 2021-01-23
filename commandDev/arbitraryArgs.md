# Arbitrary number of args commands

[<< Back to Project Overview](../defenderProject.md)

[< Back to Bot Commands](../botCommands.md)

```typescript
async function arbitraryArgumentBotCommands(inputGuildObject:Discord.Guild, commandInput) {
  let inputIDs:string[] = returnIdArrayFromArgs(args)

    switch(commandInput){
      
      case "howmanyare": // =howmanyare @role @role .. etc
        howManyAreCommand(inputGuildObject, inputIDs, currentMessage)
        break;

      case "whois": // =whois @role @role .. etc
        whoIsCommand(inputGuildObject, inputIDs, currentMessage)
        break;
      }
    }
 ```

[> Summary](../summary.md)
