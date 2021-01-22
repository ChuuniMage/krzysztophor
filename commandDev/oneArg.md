# One Argument Commands

[<< Back to Project Overview](../defenderProject.md)

In this post, we'll cover the commands which take one argument as their input. The first one we cover is the `join` command.

- The purpose of the 'join' command is to test when a user has joined Discord, and when they have joined the server. This is useful when discerning if the user's account is freshly created, as another piece of information to help discern if the user is new to the server, or if the account is a freshly made troll account.
- The first thing the command does is check if there is in fact a first argument fed into the command. If there is no argument, or it is not parsable as an ID, then the command isn't carried out.
- The `firstArgId` variable is fed into the `Discord.Guild` object's `members.fetch()` method asynchronously, to get the cache for that Discord member
- Utilising the fetched `Discord.Guildmember` object, we call its `user.createdAt()` method for the date the user joined Discord, and the `.joinedAt()` method for the date they joined the server. These are both passed through the `toDateString()` method to render these times as Javascript date strings.
- The final portion of this function, is the `message.channel.send()` method, which instructs the bot to. 
  - The first element of the `args` array, which is the raw input for the user being tested.
  - The user's Discord join date
  - The Server's name, and the user's join date to this Server.

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

The new `join` command is split into two asynchronous functions, and a type definition:

```typescript
type joinDateObjectType = {
  discordJoinDate:string,
  serverJoinDate:string
  memberName:string
}

export let returnJoinDates = async (inputGuildObject:Discord.Guild, inputUser:string):Promise<joinDateObjectType> => {
  let testedMember:Discord.GuildMember = await inputGuildObject.members.fetch(inputUser);
  let joinDateObject:joinDateObjectType = {
    discordJoinDate: testedMember.user.createdAt.toDateString(),
    serverJoinDate: testedMember.joinedAt.toDateString(),
    memberName: testedMember.displayName
  }
  return joinDateObject;
  }
```



```typescript
export let joinCommand = async (inputGuildObject:Discord.Guild, 
  inputUserId:string, 
  inputMessage:Discord.Message) =>
{
  await returnJoinDates(inputGuildObject,inputUserId).then(async (joinDateObject) => {
  inputMessage.channel.send(
    `${joinDateObject.memberName} joined:
   - Discord on ${joinDateObject.discordJoinDate}
   - ${inputGuildObject.name} on ${joinDateObject.serverJoinDate}`)}
   ).catch(() => {
    inputMessage.channel.send('No such user found.')
   }
  )
}
```

