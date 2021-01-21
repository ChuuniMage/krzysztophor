## Initialisation & On Message behavior

[<< Back to Project Overview](defenderProject.md)

This post will cover two topics:
1. The initialisation process for the bot when it starts up
2. The behavior of the bot whenever it reads a message in a Discord server

The reason that, is because a lot of functionality that used to be performed on each message, was either abstracted into a utility, or moved into the initialisation process. It's inefficient to have a program tautologically re-declare a variable or function to be itself over and over.

The old initialisation process

```typescript
console.log("The swans have been released!"); // console log to declare the bot has been initialised

const client = new Discord.Client();
client.login(config.BOT_TOKEN);
const prefix = "=";
const serverId = "780615987857850368"; // Swan Hatchery(test server) ID
const fetchCurrentGuildObject = client.guilds.fetch(serverId);
```

This initiliastion does the following:
- Instantiates a new `Discord.Client()` object, so we can access the associated methods on the object.
- Log into the Discord as a bot user, with the bot token specified in the config.json.
- Initialise the prefix being used to parse user messages for bot commands.
- Provide the ID for the server the bot is to join into.
- Define a constant for the Promise of a Discord Guild object cache.

After this initialisation, the `client.on` method is called, which defines a behavior for every time a specified event occurs from the perspective of the bot. We specify in this function to perform this operation on every Discord message event with:

```typescript
client.on("message", function (message) {
```

The first things the bot does is check if:
1. The author of the message is a bot; and if so, terminate the operation.
2. Check if the message starts with the prefix `=`, and if not, terminate the operation.
3. Define a function to check if members have any of three roles, Moderator, Administrator, or Jay Dyer.

```typescript
  if (message.author.bot) return;
  if (!message.content.startsWith(prefix)) return;

  let messagerIsModOrAdminOrJayDyer: boolean = memberHasAnyRoleByName(
    message.member,
    ["Moderator", "Administrator", "Jay Dyer"]
  );

  if (!messagerIsModOrAdminOrJayDyer) return;
  ```

Next, the bot collects the content of the message, and  parses the message values in different variables for the following purposes:
1. `args`, which is an array of strings, tokens to use for our arguments
2. `command`, which extracts first argument token, to determine the command our bot performs
3. `reasonMessage`, which stores in the case of a command such as `=kick @User For being a nuisance."
4. `firstArgId`, which extracts the user ID string , 
  -In a Discord message, what looks like `@Username` or `#channel-name` to a regular user, is something like `<!@12345678912345678>` under the hood

```typescript
  const commandBody = message.content.slice(prefix.length);
  const args = commandBody.split(" ");
  const command = args.shift().toLowerCase();

  let reasonMessage = args.slice(1, 9999).join(" ");
  let firstArgId = extractNumbersForId(args[0]);
```


```typescript
  let memberHasRolesFromArgs = (
    inputMemberCache: Discord.GuildMember
  ): boolean => {
    let memberRolesCache = inputMemberCache.roles.cache;

    let fullRolesCount = args.length;
    let hasRolesCount = 0;
    for (let i = 0; i < args.length; i++) {
      let currentMemberRoleId = extractNumbersForId(args[i]);
      let hasRoleCheck = memberRolesCache.has(currentMemberRoleId);
      if (hasRoleCheck) {
        hasRolesCount++;
      }
    }
    return hasRolesCount === fullRolesCount;
  };

  let VIPRoleList = [
    "Moderator",
    "Administrator",
    "Jay Dyer",
    "Apologist",
    "Orthodox Deacon",
    "Orthodox Priest",
    "Orthodox Bishop",
  ];

  let isMemberVIP = (inputMember): boolean => {
    return memberHasAnyRoleByName(inputMember, VIPRoleList);
  };

  async function executeBotCommand(commandInput) {...// etc
```

The new initialisation process:

```typescript
const client = new Discord.Client();
client.login(config.BOT_TOKEN);
const prefix = "=";
const serverId = config.SERVER_ID; // Swan Hatchery ID

let botPermissionsRoleList:string[] = [
  "Moderator", 
  "Administrator", 
  "Jay Dyer"]

let VIPRoleList:string[] = botPermissionsRoleList.concat([
  "Apologist",
  "Orthodox Deacon",
  "Orthodox Priest",
  "Orthodox Bishop",]);

export const isMemberVIP = (inputMember:Discord.GuildMember): boolean => {
  return memberHasAnyRoleByName(inputMember, VIPRoleList);
};

let messageHasBotPermissions = (inputMessage:Discord.Message):boolean => {
  let messagerIsBot = inputMessage.author.bot 
  if (messagerIsBot) {
    return false;
  }

  let messagerHasBotPermissions = memberHasAnyRoleByName(inputMessage.member,botPermissionsRoleList)
  if (!messagerHasBotPermissions){
    return false;
  }

  let messageStartsWithCommandPrefix = inputMessage.content.startsWith(prefix)
  if (!messageStartsWithCommandPrefix){
    return false;
  }

  return true;
}

console.log("The swans have been released!");

client.on("message", function (currentMessage) //etc...
```


---

This is what the on message code block is

```typescript
client.on("message", function (currentMessage) {//takes a message object as input
  if (!messageHasBotPermissions(currentMessage)){return}
  const fetchCurrentGuildObject = client.guilds.fetch(serverId);
  const commandBody = currentMessage.content.slice(prefix.length);
  const args = commandBody.split(" ");
  const command = args.shift().toLowerCase();

  let firstArgId = extractNumbersForId(args[0]);
  let reasonForModeration = args.slice(1, 9999).join(" ");
  

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
