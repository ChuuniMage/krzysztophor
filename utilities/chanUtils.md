# chanUtils.ts - The Channel Utilities

[<< Back to Project Overview](../defenderProject.md)

[< Back to Utilities](../utilities.md)

The only channel utility in the initial commit was `postInNamedChannel`

```typescript
export const postInNamedChannel = (
inputGuildObject:Discord.Guild,
namedChannel:string,
inputPost:string) => {
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

Two changes were made to this function in the production release: 
- Changing `namedChannel` to be the first input, making the `postInNamedChannel` function's first input curryable with the fat arrow `=>` syntax. 
- Making the function asyncronous, so that the whole program doesn't crash if it fails to find the named channel.

```typescript
export const postInNamedChannel = 
(namedChannel:string) => 
async (
inputGuildObject:Discord.Guild, 
inputPost:string) => {
  inputGuildObject.channels.cache.forEach(testedChannel => {
    if (testedChannel.name === namedChannel){
      // @ts-ignore
      return testedChannel.send(inputPost)
      }
  });
}
```

With the arrow function implementation for the `namedChannel` parameter, I can now define a new function `postInWarned`, which takes the `inputGuildObject` and `inputPost` arguments as normal, and but automatically them in the `🚨-warned-members` channel.

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

export let returnJoinDates = async (
inputGuildObject:Discord.Guild, 
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

The final `chanUtil.ts` function in the production release is `appendTxtFileIfPostTooBig`. This was the only function that needed the node File System library, `fs`, imported, and only a single function at that - `writeFileSync`.
- The bot accepts three parameters:
  - An `inputPost`, to be tested for length
  - The `inputMessage`, of the user that is operating the bot
  - The `fileName` of the text message to be appended to the bot

If the bot is in danger of making a post over 2000 characters long, this function writes out the `inputPost` to a text file in the root directory where the bot's proram is stored, and appends the written text file.

```typescript
export let appendTxtFileIfPostTooBig = (
inputPost:string, 
inputMessage:Discord.Message,
fileName:string) => {
    if (inputPost.length < 2000){
      inputMessage.channel.send(inputPost)
    } else {
      writeFileSync(`./${fileName}.txt`, inputPost);
      let fileToAttach = new Discord.MessageAttachment(`./${fileName}.txt`);
      inputMessage.channel.send(
        `Error! The post is too big. Attacking text file with message contents.`,
        fileToAttach)
  }
}
```

The next post covers the argument utilities.

[> argUtils.ts - The Argument Utilities](argUtils.md)
