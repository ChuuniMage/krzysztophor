## Initialisation & On Message behavior

[<< Back to Project Overview](defenderProject.md)

The initialisation process for the main index file of a program prepares the rest of the program for its function.

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
- The author of the message is a bot; and if so, terminate the operation.
- Check if the message starts with the prefix `=`, and if not, terminate the operation.
- Define a boolean to check if the member who made the post has any of three roles, Moderator, Administrator, or Jay Dyer. The details of the `memberHasAnyRoleByName` function will be covered in a later post.

```typescript
//client.on("message"... part 2
  if (message.author.bot) return;
  if (!message.content.startsWith(prefix)) return;

  let messagerIsModOrAdminOrJayDyer:boolean = memberHasAnyRoleByName(
    message.member,
    ["Moderator", "Administrator", "Jay Dyer"]
  );

  if (!messagerIsModOrAdminOrJayDyer) return;
  ```

Next, the bot collects the content of the message, and  parses the message values in different variables for the following purposes:
- `args`, which is an array of strings, tokens to use for our arguments
- `command`, which extracts first argument token, to determine the command our bot performs
- `reasonMessage`, which stores in the case of a command such as `=kick @User For being a nuisance."
- `firstArgId`, which extracts the ID string from the first argument after the command input. The `extractNumbersForId` function returns a string parsed with a regular expression to exclude all characters except for numbers.
  -In a Discord message, what looks like `@Username` or `#channel-name` to a regular user, looks like`<!@12345678912345678>` under the hood to use this ID number, it must be extracted from the message.

```typescript
//client.on("message"... part 3
  const commandBody = message.content.slice(prefix.length);
  const args = commandBody.split(" ");
  const command = args.shift().toLowerCase();

  let reasonMessage = args.slice(1, 9999).join(" ");
  let firstArgId = extractNumbersForId(args[0]);
```

The next three components are:
- The memberHasRolesFromArgs function, used later in the body of the message function, reads arguments fed into the bot for if a member has all of the listed roles. 
- The VIPRoleList array, which is an array of role names that are immune to bot commands such as `kick` or `ban
- The isMemberVIP function, which tests for whether the user has a VIP role

```typescript
//client.on("message"... part 4
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

Clearly, having so many computations being performed every single time a message is posted is not ideal. 

In the 1.0 Release, I optimised the initialisation process and the on message behavior the following ways:
- `Server ID`, `botPermissionsRoleList`, and `VIPRoleList`, are all moved to the config.json, to decouple the details of the server itself from the main index file.
- The `memberHasRolesFromArgs` has been abstracted and changed. The following post will explicate the details around its previous implementation, and how it has changed.
- `isMemberVIP` function is defined prior, so that it isn't redefiend every time a message event is sent
- The bot permissions checks have been extracted into the `messageHasBotPermissions` function, to declutter the `on message` event code
- The initialisation confirmation console log, "The swans have been released!" is moved to the end of the initialisation, rather than the start. Now it's a useful message in the console.

```typescript
const client = new Discord.Client();
client.login(config.BOT_TOKEN);
const prefix = "=";
const serverId = config.SERVER_ID; // Swan Hatchery ID
const VIPRoleList = config.VIPRoleList
const botPermissionsRoleList = config.botPermissionsRoleList

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

After this cleanup and reorganisation, this is what the `client.on("message"...` now code looks like.

```typescript
client.on("message", function (currentMessage) {//takes a message object as input
  if (!messageHasBotPermissions(currentMessage)){return}
  const fetchCurrentGuildObject = client.guilds.fetch(serverId);
  
  const commandBody = currentMessage.content.slice(prefix.length);
  const args = commandBody.split(" ");
  const command = args.shift().toLowerCase();

  let firstArgId = extractNumbersForId(args[0]);
  let reasonForModeration = args.slice(1, 9999).join(" ");
  
async function executeBotCommands (command:string) {...
```  
One other thing has changed: `const fetchCurrentGuildObject = client.guilds.fetch(serverId)` has been relocated here, to ensure that the bot is always fetching the most recent cache. Since it was defining the cache fetch prior to the message event, there was no guarantee that it was fetching the most up-to-date cache possible every time. Overall, this structure is far cleaner and introduces less performance issues.

Before we look at the commands themselves,  we will look at the Argument-Parsing Utility functions.

[>> The Argument Utilities](argUtils.md)
