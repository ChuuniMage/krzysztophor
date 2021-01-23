# chanUtils.ts - The Channel Utilities

[<< Back to Project Overview](../defenderProject.md)

[< Back to Utilities](../utilities.md)

The only channel utility in the initial commit was `postInNamedChannel`

```typescript
export const postInNamedChannel = (inputGuildObject:Discord.Guild,namedChannel:string,inputPost:string) => {
  let channelsArray = inputGuildObject.channels.cache.array()
  for (let i = 0; i < channelsArray.length; i++){
    let testedChannel = channelsArray[i].name;

      if (testedChannel === namedChannel){
            const targetChannel = inputGuildObject.channels.cache.get(channelsArray[i].id);
            if (!targetChannel) {return}
            // @ts-ignore
            targetChannel.send(inputPost);
            return;}
      }
}
```

Only one change was made to this function in the production release:

```typescript
export const postInNamedChannel = 
(namedChannel:string) => 
(inputGuildObject:Discord.Guild, inputPost:string) => {.../
```
The `namedChannel` input was made curryable, in order to define the `postInWarned` function, since posting in this specific channel is a function of many of the bot commands.

```typescript
export let postInWarned = postInNamedChannel("🚨-warned-members")
```

Two more functions and one type definition were in the production release of this bot:
- `joinDateObjectType`, which defines the output of the `returnJoinDatesFunction`
- `returnJoinDatesFunction`, which fetches the tested member's date they joined discord, the date they joined the server, and their name.

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

The final function in the production release is `appendTxtFileIfPostTooBig`, the use case being if bot intends on posting a message potentially over 2000 characters long. The only command that fits this criterea is the `whois` command, which will be covered in a future post.

```typescript
export let appendTxtFileIfPostTooBig = (inputPost:string, inputMessage:Discord.Message) => {
    if (inputPost.length < 2000){
      inputMessage.channel.send(inputPost)
    } else {
      writeFileSync("./whoIsFile.txt", inputPost);
      let fileToAttach = new Discord.MessageAttachment("./whoIsFile.txt");
      inputMessage.channel.send(
        `Error! The post is too big. Attacking text file with message contents.`,
        fileToAttach)
  }
}
```

The next post covers the argument utilities.

[> argUtils.ts - The Argument Utilities](argUtils.md)
