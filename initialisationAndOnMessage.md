## Initialisation & Message Initialisation

[<< Back to Project Overview](defenderProject.md)

[< Back to Folder Structure, Imports and Dependencies](importsSection.md)

Programs will need to initialise a set of variables, constants, and function definitions when they are executed. This program is no exception - indeed, there are two stages of initialisation:
1. The first initialisation process when the bot starts up
2. The bot's next stage of initialisation whenever it reads a message in Discord

The initial commit for the bot didn't organise these two initialisation processes as optimally as it could have, so this post covers the initial commit for the initialisation process & message initialisation

The initial commit's initialisation process does the following:
- Instantiates a new `Discord.Client()` object, so we can access the associated methods on the object.
- Log into the Discord as a bot user, with the bot token specified in the config.json.
- Initialise the prefix being used to parse user messages for bot commands.
- Provide the ID for the server the bot is to join into.
- Define a constant for the Promise of a Discord Guild object cache.

```typescript
console.log("The swans have been released!"); // console log to declare the bot has been initialised

const client = new Discord.Client();
client.login(config.BOT_TOKEN);
const prefix = "=";
const serverId = "780615987857850368"; // Swan Hatchery(test server) ID
const fetchCurrentGuildObject = client.guilds.fetch(serverId);
```

After this initialisation, the `client.on` method is called, which defines a behavior for every time the specified event occurs. In this case, the specified event is a discord message

```typescript
client.on("message", function (message) {// insert rest of function here
```

The bot first makes a serise of boolean tests:
- If the author of the message is a bot; and if so, terminate the operation.
- If the message starts with the prefix `=`, and if not, terminate the operation.
- If the member who made the post has any of three roles, Moderator, Administrator, or Jay Dyer(the owner of the server). The details of the `memberHasAnyRoleByName` function will be covered in a later post.

```typescript
//client.on("message", function (message) { // part 2
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
- `command`, which is removed from the first element of the args array to determine the command our bot performs
- `reasonMessage`, which stores a string in case a command such as `kick` is called, to append the reason for usage of the command, (ie, `=kick @User For being a nuisance.`)
- `firstArgId`, which extracts the ID string from the first argument remaining in `args`. The `extractNumbersForId` function returns a string parsed with a regular expression to exclude all characters except for numbers.
  -In a Discord message, what looks like `@Username` or `#channel-name` to a regular user, looks like`<!@12345678912345678>` or `<!#12345678912345678>` under the hood to use this ID number, it must be extracted from the message.

```typescript
//client.on("message"... part 3
  const commandBody = message.content.slice(prefix.length);
  const args = commandBody.split(" ");
  const command = args.shift().toLowerCase();

  let reasonMessage = args.slice(1, 9999).join(" ");
  let firstArgId = extractNumbersForId(args[0]);
```

The next three components are:
- The `memberHasRolesFromArgs` function, tests a member's roles against the roles provided in `args`. The details of this function will be covered in the [next post](argUtils.md). 
- The `VIPRoleList` array, an array of role names granted immunity to commands such as `kick` or `ban`
- The `isMemberVIP` function, which tests whether the user has a VIP role

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

For the production release, I optimised the initialisation process and the message initialisation behavior the following ways:
- `Server ID`, `botPermissionsRoleList`, and `VIPRoleList`, are all moved to the config.json, to decouple the details of the server itself from the main index file.
- The `memberHasRolesFromArgs` has been abstracted and changed. The [following post](argUtils.md) will explicate the details around its previous implementation, and how it has changed.
- `isMemberVIP` function is defined prior, so that it isn't redefiend every time a message event is sent
- The bot permissions checks have been extracted into the `messageHasBotPermissions` function, and moved into the setup initialisation, to declutter the Message initialisation
- The console log, `The swans have been released!"` is moved to the end of the setup initialisation, rather than the start. Now it's a useful message in the console to verify it's initialisation.

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

After this cleanup and reorganisation, this is what the Message Initialisation code looks like.

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
One other thing has changed: `const fetchCurrentGuildObject = client.guilds.fetch(serverId)` has been relocated here, to ensure that the bot is always fetching the most recent cache. Since it was defining the cache fetch prior to the message event, there was no guarantee that it was fetching the most up-to-date cache possible every time. Overall, the release structure is far cleaner than the initial commit.

Before we look at the commands themselves,  we will look at the Argument-Parsing Utility functions.

[>> The Argument Utilities](argUtils.md)
