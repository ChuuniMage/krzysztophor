# One Argument Commands

[<< Back to Project Overview](../defenderProject.md)

[< Back to The Bot Commands](../botCommands.md)

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

The new `join` command is split into two asynchronous functions, and a type definition. The type definition, and one of those functions, are extracted to the `chanUtils.ts` file, which is a dependency of `botCommands.ts` .

The type definition defines an object that we want to return with , a set of three values
  - `discordJoinDate`, the date the user joined Discord
  - `serverJoinDate`, the date the user joined the server
  - `memberName`, The user's name.
  
The `returnJoinDates` function asynchronously fetches the data we need for our `joinCommand` function, in a similar way to, and returns our `joinDateObject` with the values associated.

```typescript
type joinDateObjectType = {
  discordJoinDate:string,
  serverJoinDate:string
  memberName:string
}

export let returnJoinDates = async (inputGuildObject:Discord.Guild, 
inputUser:string):Promise<joinDateObjectType> => {
  let testedMember:Discord.GuildMember = await inputGuildObject.members.fetch(inputUser);
  let joinDateObject:joinDateObjectType = {
    discordJoinDate: testedMember.user.createdAt.toDateString(),
    serverJoinDate: testedMember.joinedAt.toDateString(),
    memberName: testedMember.displayName
  }
  return joinDateObject;
  }
```

So, there are numerous important differences in the final implementation of the `joinCommand` function:
- Since the function has been abstracted out of the main index file, it now requires three arguments:
  1. The Server's cache, a `Discord.Guild` object
  2. The queried user's ID 
  3. The message `Discord.Message` that told the bot what to do
  
Taking in the `Discord.Guild` object, and the `Discord.Message` object is a common update to all of the extracted functions, in order to make them extractable. Previously, they were both global variables that were referenced within the main body of the function.

The final addition to the function, which is a common adition to all of the extracted functions, is an async catch block, which does a better job. Previously, all  `(!firstArgId)` was doing was checking to see if the value was null, but this doesn't catch the case where a user's name is an invalid input. Now, the function properly handles all failures to fetch the Discord user.

```typescript
export let joinCommand = async (
  inputGuildObject:Discord.Guild, 
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

The next two commands, `kick` and `ban`, are structurally identical:
- Do the boilerplate `(!firstAgId)` check
- Fetch the member to be kicked or banned asynchronously
- Test if the member has a VIP role, and if they do, abort the function, failing to kick or ban them
- Apply the `.kick` and `.ban` methods to the user, providing a `reasonMessage` for their banning if applicable. The `.ban` method also requires specifying a number of days in the past from which the user's messages are to be deleted. Since none of the user's messages are to be deleted, this number is set to zero.
- It feeds the raw input of the first argument into the `message.channel.send` method, so that the kicked or banned user are tagged.

```typescript
      case "kick": // =kick @user
        if (!firstArgId) {
          return;
        }
        let kickedUser = await currentGuildObject.members.fetch(firstArgId);
        if (isMemberVIP(kickedUser)) {
          return;
        }

        kickedUser.kick(reasonMessage);
        message.channel.send(`${args[0]} has been kicked! ${reasonMessage}`);
        break;

// The following are the only parts of the "ban" case that are different from "kick"
        bannedUser.ban({ days: 0, reason: reasonMessage });
        message.channel.send(`${args[0]} has been banned! ${reasonMessage}`);
```

In the production release of the bot, two changes were made to the kick & ban commands:
- The commands have been extracted and made asynchronous so that they can be imported from `botCommands.ts`, and give an error message if the user isn't found.
- The `postInWarned` command is used, instead of `postInNamedMessage`, for brevity's sake

```typescript
export let kickUserCommand = async (
  inputGuildObject:Discord.Guild, 
  inputUserId:string, 
  inputMessage:Discord.Message, 
  inputReasonMessage?:string):Promise<void> => {
    
  await inputGuildObject.members.fetch(inputUserId).then(kickedUser => {
    if (isMemberVIP(kickedUser)) {
      return;
    }
    kickedUser.kick(inputReasonMessage)
    postInWarned(inputGuildObject, `<@!${kickedUser.id}> has been kicked! ${inputReasonMessage}`)
  }).catch(() => {
    inputMessage.channel.send('No such user found.')
   }
  )
}

// The following is the difference between the "banCommand" function and "kickCommand"

    bannedUser.ban({ days: 0, reason: inputReasonMessage });
    postInWarned(inputGuildObject, `<@!${bannedUser.id}> has been banned! ${inputReasonMessage}`)

```

Quarantine and unquarantine are both simple and structurally identical commands
- The `if (!firstArgId)` boilerplate
- `applyRoleByNameToUser` or `removeRoleByNameFromUser` are called, given the `"Quarantine"` role name, the `firstArgId`, and the guild object as inputs.
- `postInNamedChannel` is called, to post a message about the user's quarantine status in the "🚨-warned-members" channel

```typescript
      case "quarantine": // =quarantine @user
        if (!firstArgId) {
          return;
        }
        applyRoleByNameToUser("Quarantine", firstArgId, currentGuildObject);
        postInNamedChannel(
          currentGuildObject,
          "🚨-warned-members",
          `${args[0]} has been quarantined! ${reasonMessage}`
        );
        break;
 ```
 
 In the production release, they were refactored identically to the `kick` and `ban` commands.
 ```typescript
 export let quarantineCommand = async (
  inputGuildObject:Discord.Guild, 
  inputUserId:string, 
  inputMessage:Discord.Message, 
  inputReasonMessage?:string) => {
  await inputGuildObject.members.fetch(inputUserId).then(()=>{
    
    updateUserRole.addRole.byName(inputGuildObject,inputUserId,"Quarantine");
    postInWarned(inputGuildObject, `<@!${inputUserId}> has been quarantined! ${inputReasonMessage}`)
  }).catch(() => {
    inputMessage.channel.send('No such user found.')
   }
  )
}
```

Since `warn`, the last command of the One Argument commands is very large, I'm going to cover it by itself in the next post.

[>> The Warn Command](warnCommand.md)
