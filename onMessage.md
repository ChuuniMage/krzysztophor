This is what the on message code block is

```client.on("message", function (currentMessage) {//takes a message object as input
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
