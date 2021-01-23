# Two Argument Commands

[<< Back to Project Overview](../defenderProject.md)

[< Back to Zero argument commmands](zeroArgs.md)

For the commands that take two arguments, we have three functions:
- `replaceall`, which replaces all of a particular role X that users have, with another role Y
- `msg`, which instructs the bot to send a message to a particular channel
- `dmsg`, which instructs the bot to directly message a particular user


```typescript
async function twoArgumentBotCommands(inputGuildObject:Discord.Guild, commandInput) {

  switch (commandInput) {

    case `msg`: // =msg #channel message
      let targetChannelId:string = extractNumbersForId(args[0]);
      msgCommand(inputGuildObject,targetChannelId,currentMessage,reasonForModeration)
      break;

    case `dmsg`: // =dmsg @user message
      let targetUserId:string = firstArgId;
      dmsgCommand(inputGuildObject,targetUserId,currentMessage,reasonForModeration)
      break;

    case "replaceall": // =replaceall @roleA @roleB
      let replacedRoleId:string = extractNumbersForId(args[0]);
      let newRoleId:string = extractNumbersForId(args[1]);
      replaceAllRolesCommand(inputGuildObject, replacedRoleId, newRoleId)
      break;
    }
}
```

[>> Arbitrary argument commmands](arbitraryArgs.md)
