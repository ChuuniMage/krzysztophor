## Initialisation

```const client = new Discord.Client();
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

console.log("The swans have been released!");```
