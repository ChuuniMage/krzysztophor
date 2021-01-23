# Two Argument Commands

[<< Back to Project Overview](../defenderProject.md)

[< Back to Zero argument commmands](zeroArgs.md)

For the commands that take two arguments, we have three functions:
- `replaceall`, which replaces all of a particular role X that users have, with another role Y
- `msg`, which instructs the bot to send a message to a particular channel
- `dmsg`, which instructs the bot to directly message a particular user

First, let's look at the initial commit's `replaceall` command
- `hasRoleToReplace` 
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

Since the `discord.js` API doesn't throw any errors, and behaves cleanly when you add a role to a user who already has that role, and when you remove a role from a user that doesn't have that role, this let me heavily streamline the production release of the `replaceall` command; The functionality of `replaceTheRole` is put into into the main body of `replaceAllCommand`, and applies it to each each user indiscriminately. 

```export let replaceAllRolesCommand = async (inputGuildObject:Discord.Guild,inputReplacedRoleId:string,inputNewRoleId:string) => {
  let checkedMembers = await inputGuildObject.members.fetch();
  checkedMembers.forEach((member)=> {
    if (memberHasAllRolesById(member,[inputReplacedRoleId])){
    updateUserRole.removeRole.byId(inputGuildObject,member.id,inputReplacedRoleId);
    updateUserRole.addRole.byId(inputGuildObject,member.id,inputNewRoleId)
      };
  })
}

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

[>> One Argument Commmands](oneArg.md)
