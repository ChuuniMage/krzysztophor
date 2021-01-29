# Two Argument Commands

[<< Back to Project Overview](../defenderIndex.md)

[< Back to Zero argument commmands](zeroArgs.md)

For the commands that take two arguments, we have three functions:

- `replaceall`, which replaces all of a particular role X that users have, with another role Y
- `msg`, which instructs the bot to send a message to a particular channel
- `dmsg`, which instructs the bot to directly message a particular user

First, let's look at the initial commit's `replaceall` command

- The `(!firstArgID)` check we find in other functions has another parameter - `(!args[1])` which tests if there is a second argument given or not. These were replaced by the switch statement covered in [Bot Commands](../botCommands.md).
- `replacedRoleId` and `newRoleId` are defined from the `args` posted in the message
- `hasRoleToReplace` wraps up `memberHasAllRolesById`, referencing an out-of-scope variable `replacedRoleId`. This
- `replaceTheRole` wraps up `removeRoleByIdFromUser` and `addRoleByIdToUser` into a single function, and clumsily accepts a dummy void value in order to fit into `iterateOverMembersAndReturnData`
- Just like `whois` and `howmanyare`, this function used `iterateOveRembersAndReturnData`, but took in `undefined` as its data, since I wasn't returning any data, but altering member data.

```typescript
case "replaceall":
        if (!firstArgId || !args[1]) {
          return;
        }
        let replacedRoleId = extractNumbersForId(args[0]);
        let newRoleId = extractNumbersForId(args[1]);

        let hasRoleToReplace = (inputMember: Discord.GuildMember) => {
          return memberHasAllRolesById(inputMember, [replacedRoleId]);
        };

        let replaceTheRole = (
          inputMember: Discord.GuildMember,
          dummyData: void
        ) => {
          removeRoleByIdFromUser(
            replacedRoleId,
            inputMember.id,
            currentGuildObject
          );
          applyRoleByIdToUser(
          newRoleId,
          inputMember.id,
          currentGuildObject);
        };

        iterateOverMembersAndReturnData(
          undefined,
          currentGuildObject,
          hasRoleToReplace,
          replaceTheRole
        );
```

The functionality of the `replaceTheRole` callback is put into into the main body of `replaceAllCommand`, and applies it to each each user indiscriminately.

```typescript
export let replaceAllRolesCommand = async (
  inputGuildObject: Discord.Guild,
  inputReplacedRoleId: string,
  inputNewRoleId: string
) => {
  let checkedMembers = await inputGuildObject.members.fetch();
  checkedMembers.forEach((member) => {
    if (memberHasAllRolesById(member, [inputReplacedRoleId])) {
      updateUserRole.removeRole.byId(
        inputGuildObject,
        member.id,
        inputReplacedRoleId
      );
      updateUserRole.addRole.byId(inputGuildObject, member.id, inputNewRoleId);
    }
  });
};
```

The `msg` command from the initial commit is as follows:

- The boilerplate `if (!firstArgId)` check, which tests if the first argument given to the bot even exists
- A superfluous extraction of the channel id from the first argument, given `firstArgId` already is this ID extraction string
- The definition of a new constant, `targetChannel`, which is a new `Discord.GuildChannel` object.
- A check if, after being fetched, this channel actually exists. If it does not exist, then the command's execution is aborted.
- The `.send` method on the `Discord.Guildmember` channel is called to send a message to this channel. For some mysterious reason, the typescript compiler complains, saying that this method does not exist on the object. According to the `Discord.js` API, this does, and in fact this code when executed, functions correctly. I have added `// #ts-ignore` over this line, in order to force the typescript compiler to ignore this supposed error.

```typescript
      case `msg`: // =msg #channel message
        if (!firstArgId) {
          return;
        }
        let extractedChannelId = extractNumbersForId(args[0]);
        const targetChannel = client.channels.cache.get(extractedChannelId);
        if (!targetChannel) {
          return;
        }
        // @ts-ignore
        targetChannel.send(reasonMessage); // intellisense complains but this works
        break;
```

The production release `msg` command is optimised as suchs:

- The assignment of the `targetChannel` constant is the same
- The `postInNamedChannel` utility is called, which allows is to implement a `.catch()` block in case a channel is not found with the given argument's input.

The compiler doesn't complain, and the code is neatly packaged away in the `botCommands.ts` file.

```typescript
export let msgCommand = async (
  inputGuildObject: Discord.Guild,
  inputChannelId: string,
  inputMessage: Discord.Message,
  inputPost: string
) => {
  const targetChannel = inputGuildObject.channels.cache.get(inputChannelId)
    .name;
  postInNamedChannel(targetChannel)(inputGuildObject, inputPost).catch(() =>
    inputMessage.channel.send("No channel found")
  );
};
```

The initial commit's `dmsg` command, which sends a message directly to a user, is identical in structure to the previous command, except the `.send` method is called on the `GuildMember` object.

```typescript
      case `dmsg`: // =dmsg @user message
        if (!firstArgId) {
          return;
        }
        let messagedUser = await currentGuildObject.members.fetch(firstArgId);
        if (!messagedUser) {
          return;
        }
        messagedUser.send(reasonMessage);
        break;
```

Similarly, the production release `dmsgCommand` is identical in structure to the `msgCommand`.

```typescript
export let dmsgCommand = async (
  inputGuildObject: Discord.Guild,
  inputUserId: string,
  inputMessage: Discord.Message,
  inputPost: string
) => {
  await inputGuildObject.members
    .fetch(inputUserId)
    .then((messagedUser) => {
      messagedUser.send(inputPost);
    })
    .catch(() => inputMessage.channel.send("No user found."));
};
```

These three functions are called in their corresponding switch cases in the `twoArgumentBotCommands` function, which parses the inputs to the functions from the arguments given to the bot, prior to feeding them into the functions.

```typescript
async function twoArgumentBotCommands(
  inputGuildObject: Discord.Guild,
  commandInput
) {
  switch (commandInput) {
    case `msg`: // =msg #channel message
      let targetChannelId: string = extractNumbersForId(args[0]);
      msgCommand(
        inputGuildObject,
        targetChannelId,
        currentMessage,
        reasonForModeration
      );
      break;

    case `dmsg`: // =dmsg @user message
      let targetUserId: string = firstArgId;
      dmsgCommand(
        inputGuildObject,
        targetUserId,
        currentMessage,
        reasonForModeration
      );
      break;

    case "replaceall": // =replaceall @roleA @roleB
      let replacedRoleId: string = extractNumbersForId(args[0]);
      let newRoleId: string = extractNumbersForId(args[1]);
      replaceAllRolesCommand(inputGuildObject, replacedRoleId, newRoleId);
      break;
  }
}
```

In the next post, we will cover the commands which take a single argument.

[>> One Argument Commmands](oneArg.md)
