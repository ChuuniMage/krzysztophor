# Bot commands & their development

[<< Back to Project Overview](defenderProject.md)

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

However, this is better sorted out in two ways:
1. Set up a bottleneck around the amount of arguments fed into the function, so that the function cannot be called with an invalid number of arguments
2. Implement async catch statements in the functions to return an error if the function parameter is invalid.

Item 1 has been easily implemented in the 1.0 Release:

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

The second item on that list will be addressed in the next page, on one argument commands

[>> Zero argument commmands](commandDev/zeroArgs.md)
