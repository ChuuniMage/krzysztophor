# The Warn Command

[<< Back to Project Overview](../defenderProject.md)

[< Back to One argument commmands](oneArg.md)

The `warn` command is a very long command, because it checks for multiple different cases on a single user, and executes different commands for each case. The command is utilised in conjunction with two roles, the `Warned` role, and the `Warned Twice` role, to establish the following system:
- Fetch the User's `Discord.Guildmember` jobect, and test if it had any of the VIP roles. If he did, abort the function.
- If the user has the `Warned Twice` role, ban them.
- If the user has the `Warned` role, remove this role, add the `Warned Twice` and `Quarantined` role, and post an official warning.
- If the user has neither role, add the `Warned` and `Quarantined role`, and post an official warning.
- After the execution of each of these commands, the aprpopriate warning message would be posted in the `🚨-warned-members` channel.

```typescript
      case "warn": // =warn @user
        if (!firstArgId) {
          return;
        }

        let warnedUser = await currentGuildObject.members.fetch(firstArgId);
        if (isMemberVIP(warnedUser)) {
          return;
        }
        let rolesArray = warnedUser.roles.cache.array();

        for (let i = 0; i < rolesArray.length; i++) {
          let testedRoleName = warnedUser.roles.cache.array()[i].name;

          if (testedRoleName === "Warned Twice") {
            let bannedUser = await currentGuildObject.members.fetch(firstArgId);
            bannedUser.ban({ days: 0, reason: reasonMessage });
            postInNamedChannel(
              currentGuildObject,
              "🚨-warned-members",
              `${args[0]} has been banned for being warned three times! ${reasonMessage}`
            );
            return;
          }
          if (testedRoleName === "Warned") {
            removeRoleByNameFromUser("Warned", firstArgId, currentGuildObject);
            applyRoleByNameToUser(
              "Warned Twice",
              firstArgId,
              currentGuildObject
            );

            postInNamedChannel(
              currentGuildObject,
              "🚨-warned-members",
              `${args[0]} has been warned twice! ${reasonMessage}`
            );
            return;
          }
        }
        applyRoleByNameToUser("Warned", firstArgId, currentGuildObject);
        postInNamedChannel(
          currentGuildObject,
          "🚨-warned-members",
          `${args[0]} has been warned! ${reasonMessage}`
        );
          
        break;
 ```
 
The following is the production release implementation of the `warnCommand` function:
- Comments have been added to explicate the warning system
- The asynchronous nature of the function allows us to `.catch` the case for if the user is not found.

No other changes have been made - including a particularly obvious one, the implementation of the `.forEach` method, rather than the `for loop`. 

The reason why the `.forEach` method hasn't been used, is that during the testing of the `.forEach` implementation of this function, a particualarly worrying bug came arose - a user with the `Warned` role would be warned, have it removed, be given the `Warned Twice` role, and then banned for having the newly applied `Warned Twice` role. The `for` implementation allows an early exit from the iterations over the user's roles, keeping them safe from this unintended behavior.

```typescript
export let warnCommand = async (
  inputGuildObject:Discord.Guild, 
  inputUserId:string, 
  inputMessage:Discord.Message,
  inputReasonMessage?:string) => {
  await inputGuildObject.members.fetch(inputUserId).then(warnedUser => {
// Three-stage warning system. 
// If no warned role, add "Warned" role. 
// If has "Warned", remove "Warned", add "Warned Twice". 
// If "Warned Twice", ban.
    if (isMemberVIP(warnedUser)) {
      return;
    }
    let rolesArray = warnedUser.roles.cache.array();

    for (let i = 0; i < rolesArray.length; i++) {
      let testedRoleName = warnedUser.roles.cache.array()[i].name;
      if (testedRoleName === "Warned Twice") {
        warnedUser.ban({ days: 0, reason: inputReasonMessage });
        postInWarned(inputGuildObject,
          `<@!${inputUserId}> has been banned for being warned three times! ${inputReasonMessage}`);
        return;
      }
      if (testedRoleName === "Warned") {
        updateUserRole.removeRole.byName(inputGuildObject,inputUserId,"Warned")
        updateUserRole.addRole.byName(inputGuildObject,inputUserId,"Warned Twice");
        updateUserRole.addRole.byName(inputGuildObject,inputUserId,"Quarantine")
        postInWarned(inputGuildObject,
          `<@!${inputUserId}> has been warned twice! ${inputReasonMessage}`);
        return;
      }
    }
    updateUserRole.addRole.byName(inputGuildObject,inputUserId,"Warned");
    updateUserRole.addRole.byName(inputGuildObject,inputUserId,"Quarantine")
    postInWarned(inputGuildObject,
      `<@!${inputUserId}> has been warned! ${inputReasonMessage}`);
  }).catch(() => {
    inputMessage.channel.send('No such user found.')
   }
  )
}
```

The `oneArgumentBotCommands` function composed of all of these functions, has been implemented as such in the main index file, with the relevant parsed arguments given into each command.

```typescript
async function oneArgumentBotCommands(inputGuildObject:Discord.Guild, commandInput) {

  switch (commandInput) {
  case "join": // =join @user
    let joinTestUser:string = firstArgId;
    joinCommand(inputGuildObject,joinTestUser,currentMessage)
    break;

  case "kick": // =kick @user
    let kickedUserId:string = firstArgId;
    kickUserCommand(inputGuildObject, kickedUserId, currentMessage, reasonForModeration)
    break;

  case "ban": // =ban @user
    let bannedUserId:string = firstArgId;
    banUserCommand(inputGuildObject, bannedUserId, currentMessage, reasonForModeration)
    break;

  case "quarantine": // =quarantine @user
    let quarantinedUserId:string = firstArgId;
    quarantineCommand(inputGuildObject, quarantinedUserId, currentMessage, reasonForModeration)
    break;

  case "unquarantine": // =unquarantine @user
    let unquarantinedUserId:string = firstArgId;
    unQuarantineCommand(inputGuildObject, unquarantinedUserId, currentMessage, reasonForModeration)
    break;

  case "warn": // =warn @user
// Three-stage warning system. 
// If no warned role, add "Warned" role. 
// If has "Warned", remove "Warned", add "Warned Twice". 
// If "Warned Twice", ban.
    let warnedUserId:string = firstArgId
    warnCommand(inputGuildObject,warnedUserId,currentMessage,reasonForModeration)
    break;
  }

}
```

In the final post on the bot commands, we will cover the commands which take no arguments.

[> Zero Argument Commands](zeroArgs.md)
