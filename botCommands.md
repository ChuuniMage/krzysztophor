# Bot Commands

[<< Back to Project Overview](defenderProject.md)

[< Back to argUtils.ts - The Argument Utilities](utilities/argUtils.md)

The whole purpose of the bot is to perform commands given to it from Discord messages. These posts will cover the structure of the functions of the commands, as well as the higher level structure around implementing these functions into the bot.

As an example, here is the start of the initial commit's `executebotCommand` function,

- The initialisation of `executeBotCommand` asynchronously fetches the discord server cache, ie the `Discord.Guild` object
- The main body of the function is a switch statement evaluating the `commandInput` fed into it

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

We'll leave examination of this command for the next post. For now, what interests us is this piece of biolerplate code:

```typescript
        if (!firstArgId) {
          return;
        }
```
This code is in 11 of the 13 bot command functions in this file. The purpose of this code is to test if an argument was fed into the bot command at all, and to abort the function if no argument was fed into the function.

Boilerplate sucks, so to fix this problem, we change the `executeBotCommands` function to feed the length of the `args` array into a switch statement, executing our commands depending on how many inputs our commands accept. This way, the logic check is performed upstream, and this boilerplate has been taken out 11 functions.

```typescript
async function executeBotCommands (command:string) {
  let currentGuildObject:Discord.Guild = await fetchCurrentGuildObject
  
  let lengthOfArgs = args.length
  switch(lengthOfArgs){
  case(0):
    zeroArgumentBotCommands(currentGuildObject, command)
    break;
  case(1):
    oneArgumentBotCommands(currentGuildObject, command)
    arbitraryArgumentBotCommands(currentGuildObject, command);
    break;
  case(2): 
    oneArgumentBotCommands(currentGuildObject, command) // To account for reasonMessage
    twoArgumentBotCommands(currentGuildObject, command)
    arbitraryArgumentBotCommands(currentGuildObject, command);
    break;
  default:
    arbitraryArgumentBotCommands(currentGuildObject, command);
    break;
  }
}
```

So where does our `join` command go now? It's the first switch case in the `oneArgumentBotCommands` function.

```typescript
async function oneArgumentBotCommands(inputGuildObject:Discord.Guild, commandInput) {
  switch (commandInput) {
  case "join": // =join @user
    let joinTestUser = firstArgId;
    joinCommand(inputGuildObject,joinTestUser,currentMessage)
    break;
```

As covered in [Folder Structure, Imports and Dependencies](importsSection.md), the `join` function has been abstracted into an imported `joinCommand` function.

In the next post, we'll see how `join`, and other one argument commands, have been updated to support this abstraction.

[>> One argument commmands](commandDev/oneArg.md)
