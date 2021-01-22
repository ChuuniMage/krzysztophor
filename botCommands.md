# Bot commands & their development

[<< Back to Project Overview](defenderProject.md)

The whole purpose of the bot is to perform commands given to it from Discord messages. These posts will cover the structure of the functions of the commands, as well as the higher level structure around implementing these functions into the bot.

As an example, here is the start of the initial commit's `executebotCommand` function,

- The initialisation of `executeBotCommand` asynchronously fetches the discord server cache, ie the `Discord.Guild` object, defined earlier
- The the top level structure of the function is a switch statement, feeding every valid command into the switch statement for further

```typescript
  async function executeBotCommand(commandInput) {
    let currentGuildObject = await fetchCurrentGuildObject;
```

In this example we'll examine the first function in the switch statement, the `join` command:
- The purpose of the 'join' command is to test when a user has joined Discord, and when they have joined the server. This is useful when discerning if the user's account is freshly created, as another piece of information to help discern if the user is new to the server, or if the account is a freshly made troll account.
- The first thing the command does is check if there is in fact a first argument fed into the command. If there is no argument, or it is not parsable as an ID, then the command isn't carried out.
- The `firstArgId` variable is fed into the `Discord.Guild` object's `members.fetch()` method asynchronously, to get the cache for that Discord member
- Utilising the fetched `Discord.Guildmember` object, we call its `user.createdAt()` method for the date the user joined Discord, and the `.joinedAt()` method for the date they joined the server. These are both passed through the `toDateString()` method to render these times as Javascript date strings.
- The final portion of this function, is the `message.channel.send()` method, which instructs the bot to. 
  - The first element of the `args` array, which is the raw input for the user being tested.
  - The user's Discord join date
  - The Server's name, and the user's join date to this Server.


```typescript
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

There is this particular piece of boilerplate in every single bot command that takes arguments:

```typescript
      case "join":
        if (!firstArgId) {
          return;
        }
```
```typescript
      case "replaceall":
        if (!firstArgId || !args[1]) {
          return;
        }
```
Boilerplate sucks. The purpose of this boilerplate is to test if there is a valid argument fed into the bot command.

To sort this out, we change the `executeBotCommands` function to be a bottleneck for checks like this, so that the bot commands cannot be called with an invalid number of arguments.

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

So where does our `join` command go now? It's the first switch case in the the `oneArgumentBotCommands` function.

```typescript
async function oneArgumentBotCommands(commandInput) {
  let currentGuildObject = await fetchCurrentGuildObject;
  switch (commandInput) {
  case "join": // =join @user
    let joinTestUser = firstArgId;
    joinCommand(currentGuildObject,joinTestUser,currentMessage)
    break;
```

As mentioned in the [Folder Structure, Imports and Dependencies](importsSection.md) post, the `join` function has been abstracted into an imported `joinCommand` function.

In the next post, we'll see how `join`, and other one argument commands, have been changed. 

[>> One argument commmands](commandDev/oneArg.md)
