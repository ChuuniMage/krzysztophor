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

The next three were in the production release

```typescript
export let postInWarned = postInNamedChannel("🚨-warned-members")

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


[> argUtils.ts - The Argument Utilities](argUtils.md)
