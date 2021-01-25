# Zero argument commands

[<< Back to Project Overview](../defenderProject.md)

[< Back to The Warn Command](warnCommand.md)

There are three commands which take no arguments
- The `boostinfo` command, which fetches information about the Nitro Booster status of the server, which are paid & subscribing Discord members who invest into increasing the bandwidth and capacities of the server.
- The `checkpfp` command, which tests if a user's profile picture is the default Discord profile picture, and keeps the user in a containment channel if they do.

The initial commit's `boostinfo` command is straightforward:
- Define the `boosterRole` role object, fetching it with `getRoleByName`
- Fetch the cache of all  members who have this role, by calling the `.members` property on the `Discord.Role` object
- Send a message to the channel this command was called, and in this message posting the following values
  - The Discord Server's `.premiumTier` number number
  - The Discord Server's `.premiumSubscriptionCount` property, since Discord users can purchase multiple subscriptions and assign these subscriptions arbitrarily to boost each discord server
  - After converting the cache map of all members to an array, returning the length of the array for the number of users.

```typescript
      case "boostinfo": // =boostinfo
        let boosterRole = getRoleByName("Server Booster", currentGuildObject);
        let boosterRoleMembers = (await boosterRole).members;

        message.channel.send(
      `Server is currently at tier: ${currentGuildObject.premiumTier}
      Total number of Nitro Boosts: ${currentGuildObject.premiumSubscriptionCount}
      Total number of boosters: ${boosterRoleMembers.array().length}`
        );
        break;
```

The production release `postBoostInfoCommand` is functionally idential to the previous switch case, except for two details:
- The required `await` for the `boosterRole` value has been implemented to make this function work in an extracted context
- I had discovered the `.size` method the map object, which is analogous in function to the `.length` function for arrays, so I no longer needed to convert the map into an array.


```typescript
export let postBoostInfoCommand = async (
  inputGuildObject:Discord.Guild, 
  inputMessage:Discord.Message) => {
  let boosterRole:Discord.Role = await getRoleByName(inputGuildObject, "Nitro Booster");
  let boosterRoleMembers = (await boosterRole).members;
  
  inputMessage.channel.send(
  `Server is currently at tier: ${inputGuildObject.premiumTier}
  Total number of Nitro Boosts: ${inputGuildObject.premiumSubscriptionCount}
  Total number of boosters: ${boosterRoleMembers.size}`
  );
}
```

```typescript
      case "checkpfp": // =checkpfp
        let defaultAvatarCheck = (inputMember: Discord.GuildMember) => {
          let defaultAvatar = inputMember.user.defaultAvatarURL;
          let displayAvatar = inputMember.user.displayAvatarURL();
          return defaultAvatar === displayAvatar;
        };

        let applyCheckPFPRole = (inputUser: Discord.Guild, dummyData: void) => {
          applyRoleByNameToUser(
            "HasDefaultPFP",
            inputUser.id,
            currentGuildObject
          );
        };

        iterateOverMembersAndReturnData(
          undefined,
          currentGuildObject,
          defaultAvatarCheck,
          applyCheckPFPRole
        );
```

The production release implementation was greatly improved
```typescript
export let checkPFPCommand = async (inputGuildObject:Discord.Guild) => {
  let checkedMembers = await inputGuildObject.members.fetch();
  checkedMembers.forEach((member) => {
  if (hasDefaultPFP(member)) {
    updateUserRole.addRole.byName(
    member.guild,
    member.id,
    "Change PFP")
  }
  })
}
```

In the production release, I have added the `help` command, which posts a message to the channel detailing the commands of the bot, and the arguments it takes. I have not abstracted this command into the `botCommands.ts` file, since it would just be a wrapper on the `message.channel.send` method.

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

[>> Summary](../summary.md)

