# Zero argument commands

[<< Back to Project Overview](../defenderProject.md)

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
