# Bot commands & their development

[<< Back to Project Overview](defenderProject.md)

Optimised bot commands:

```typescript
let executeBotCommands = (command:string) => {
  let lengthOfArgs = args.length
  switch(lengthOfArgs){
  case(0):
    zeroArgumentBotCommands(command)
    break;
  case(1):
    oneArgumentBotCommands(command)
    arbitraryArgumentBotCommands(command);
    break;
  case(2): 
    oneArgumentBotCommands(command) // To account for reasonmessage
    twoArgumentBotCommands(command)
    arbitraryArgumentBotCommands(command);
    break;
  default:
    arbitraryArgumentBotCommands(command);
    break;
  }
}
```


```typescript
async function zeroArgumentBotCommands(commandInput) {
  let currentGuildObject = await fetchCurrentGuildObject

  switch(commandInput){
      case "help":
        currentMessage.channel.send(`=help | Show list of commands
        =boostinfo | Show server's boost info
        =checkpfp | Applies 'Default PFP' role to users with default PFP
        =join @user | Show when user joined Discord, and this Server
        =kick @user | Kick a user
        =ban @user | Ban a user
        =quarantine @user | Applies quarantine role
        =unquarantine @user | Removes quarantine role
        =warn @user | Warn a user. If warned twice, user is banned.
        =msg #channel message | Sends message to channel
        =dmsg @user message | DM message to user
        =replaceall @roleA @roleB | Replaces all roleA with roleB
        =howmanyare @role @role .. | Show number of users with listed roles
        =whois @role @role .. | Lists users by name with listed roles`)
        break;

      case "boostinfo": // =boostinfo
        postBoostInfoCommand(currentGuildObject,currentMessage);
        break;
        
      case "checkpfp": // =checkpfp
        checkPFPCommand(currentGuildObject);
        break;
    }
}
```

```typescript
async function oneArgumentBotCommands(commandInput) {
  let currentGuildObject = await fetchCurrentGuildObject;
  switch (commandInput) {
  case "join": // =join @user
    let joinTestUser = firstArgId;
    joinCommand(currentGuildObject,joinTestUser,currentMessage)
    break;

  case "kick": // =kick @user
    let kickedUserId = firstArgId;
    kickUserCommand(currentGuildObject, kickedUserId, currentMessage, reasonForModeration)
    break;

  case "ban": // =ban @user
    let bannedUserId = firstArgId;
    banUserCommand(currentGuildObject, bannedUserId, currentMessage, reasonForModeration)
    break;

  case "quarantine": // =quarantine @user
    let quarantinedUserId:string = firstArgId;
    quarantineCommand(currentGuildObject, quarantinedUserId, currentMessage, reasonForModeration)
    break;

  case "unquarantine": // =unquarantine @user
    let unquarantinedUserId:string = firstArgId;
    unQuarantineCommand(currentGuildObject, unquarantinedUserId, currentMessage, reasonForModeration)
    break;

  case "warn": // =warn @user
// Three-stage warning system. 
// If no warned role, add "Warned" role. 
// If has "Warned", remove "Warned", add "Warned Twice". 
// If "Warned Twice", ban.
    let warnedUserId:string = firstArgId
    warnCommand(currentGuildObject,warnedUserId,currentMessage,reasonForModeration)
    break;
  }

}
```

```typescript
async function twoArgumentBotCommands(commandInput) {
  let currentGuildObject = await fetchCurrentGuildObject;
  switch (commandInput) {

    case `msg`: // =msg #channel message
      let targetChannelId = extractNumbersForId(args[0]);
      msgCommand(currentGuildObject,targetChannelId,reasonForModeration).catch(
        () => currentMessage.channel.send('No channel found')
      )
      break;

    case `dmsg`: // =dmsg @user message
      let targetUserId = firstArgId;
      dmsgCommand(currentGuildObject,targetUserId,currentMessage,reasonForModeration)
      break;

    case "replaceall":
      let replacedRoleId = extractNumbersForId(args[0]);
      let newRoleId = extractNumbersForId(args[1]);
      replaceAllRolesCommand(currentGuildObject, replacedRoleId, newRoleId)
      break;
    }
}
```

```typescript
async function arbitraryArgumentBotCommands(commandInput) {
  let currentGuildObject = await fetchCurrentGuildObject;

  switch(commandInput){

    case "howmanyare": // =howmanyare @role @role .. etc
    howManyAreCommand(currentGuildObject, args, currentMessage)
      break;

    case "whois": // =whois @role @role .. etc
    whoIsCommand(currentGuildObject, args, currentMessage)
      break;

    default:
      break;
    }
  }
```

As an example, here is the start of the previous `executebotCommand` function, with the first command in it

```typescript
  async function executeBotCommand(commandInput) {
    let currentGuildObject = await fetchCurrentGuildObject;

    switch (commandInput) {
      case "join": // =join @user
        if (!firstArgId) {
          return;
        }
        let testedMember = await currentGuildObject.members.fetch(firstArgId);
        let discordJoinDate = testedMember.user.createdAt.toDateString();
        let serverJoinDate = testedMember.joinedAt.toDateString();
        message.channel.send(
          `${args[0]} joined:
         - Discord on ${discordJoinDate}
         - ${currentGuildObject.name} on ${serverJoinDate}`
        );
        break;
       ```
