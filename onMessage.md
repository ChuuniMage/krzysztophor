## Initialisation

[<< Back to Project Overview](defenderProject.md)

The old initialisation process:

```typescript
console.log("The swans have been released!");

const client = new Discord.Client();
client.login(config.BOT_TOKEN);
const prefix = "=";
const serverId = "780615987857850368"; // Swan Hatchery ID
const fetchCurrentGuildObject = client.guilds.fetch(serverId);

client.on("message", function (message) {
  if (message.author.bot) return;
  if (!message.content.startsWith(prefix)) return;

  let messagerIsModOrAdminOrJayDyer: boolean = memberHasAnyRoleByName(
    message.member,
    ["Moderator", "Administrator", "Jay Dyer"]
  );

  if (!messagerIsModOrAdminOrJayDyer) return;

  const commandBody = message.content.slice(prefix.length);
  const args = commandBody.split(" ");
  const command = args.shift().toLowerCase();

  let reasonMessage = args.slice(1, 9999).join(" ");
  let firstArgId = extractNumbersForId(args[0]);

  //remake to be generic?
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

```typescript
const client = new Discord.Client();
client.login(config.BOT_TOKEN);
const prefix = "=";
const serverId = config.SERVER_ID; // Swan Hatchery ID
// const fetchCurrentGuildObject = client.guilds.fetch(serverId);

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
