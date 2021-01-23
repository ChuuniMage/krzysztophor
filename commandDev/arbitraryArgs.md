# Arbitrary Argument Commmands

[<< Back to Project Overview](../defenderProject.md)

[< Back to Bot Commands](../botCommands.md)

In this post we'll cover two commands, `howmanyare` and `whois`. You may remember them from the [iterateOverMembersAndReturnData](../utilities/iterate.md) post. 

The purpose of `howmanyare` is to return a number of how many users have all of the roles given to the bot as arguments. The user types in `=howmanyare @Role1 @Role2 @Role3...` for as many roles as they wish to test, and the bot returns this number. 
 - For example, if there are 5 members with the role `@Lawyer`, telling the bot `=howmanyare @Lawyer` would return `Number of users with the following roles[@Lawyer] : 5`
 - If there are 4 members who had the role `@Doctor`, and 2 have the role `@Lawyer`, then typing `=howmanyare @Doctor @Lawyer` would return  `Number of users with the following roles[@Doctor, @Lawyer] : 2`
 
 The purpose of `whois` is similar to `howmanyare`, but instead of returning a raw number, the purpose is to return a list of usernames, their unique discriminator, and nickname, ie `"DiscordUser#1234 (Nickname)"`
  - For example, if there are 5 members with the role `@Lawyer`, telling the bot `=whois @Lawyer` would return `The users with the roles[@Lawyer] are: Jack#1234 (Blackjack), Jill#5656 (UpTheHill), Gary#6479 (GaryOaks), Tamara#9999 (T4m4r4), Clint#8887 (Westwood)`
 - If there are 4 members who had the role `@Doctor`, and 2 have the role `@Lawyer`, then typing `=howmanyare @Doctor @Lawyer` would return  `The users with the roles[@Doctor, @Lawyer] are : Jack#1234 (Blackjack), Jill#5656 (UpTheHill)`
 
First, we'll see `howmanyare`'s initial implementation. 
- The `!firstArgId` boilerplate is present, taken care of in the [previous post](../botCommands.md)
- The `updateArrayForHowManyAre` function is defined, and only used once, in the scope of this function. This pushes a user's name into the array, to be used as a token to track the amount of users who are in the array.
- `whohasRoles`, an empty array is initialised
- These newly declared variables are fed into the `iterateOverMembersAndReturnData` function, which was covered in [relevant post]
- The asynchronous `.then` block defines a callback function once `iterateOverMembersAndReturnData` has returned its values
  - The `returnedRoleArray` is given to the callback function as an input
  - The `message.channel.send` method sends a message to the channel that the user gave the bot a command in.
  - The roles, which are still in the `args` array, are joined to display them in this post
  - The length of `returnedRoleArray` is calculated, to display the number of users who have the associated roles
  
```typescript
case "howmanyare": //howmanyare @role @role .. etc
        if (!firstArgId) {
          return;
        }
        let updateArrayForHowManyAre = (
          inputMember: Discord.GuildMember,
          outputArray: string[]
        ): void => {
          outputArray.push(inputMember.user.username);
        };

        let whoHasRoles = [];

        iterateOverMembersAndReturnData(
          whoHasRoles,
          currentGuildObject,
          memberHasRolesFromArgs,
          updateArrayForHowManyAre
        ).then((returnedRoleArray) => {
          message.channel.send(
            `Number of users with the following roles[${args.join(", ")}] : ${
              returnedRoleArray.length
            }`
          );
        });
        break;
```

The production release implementation of `howmanyare` is much cleaner:
- Three parameters are accepted
  - The Discord server cache, aka the GuildObject
  - The `inputIDs` being tested on the users of this server
  - The `inputMessage` that triggered this command
- The members are fetched asynchronously with `await inputGuildObject.members.fetch();`
- `usersWithRolesArray` is typed as an array of strings and and initialised
- The usernames of each user is pushed into `usersWithRolesArray`,  as tokens to count the length of the array with
- The `roleNameArray` is defined, to post the roles in the message
- A message is posted to the channel of the `inputMessage`, using the `roleNameArray` to post the tested role names, and the length of the `UsersWithRolesArray`

```typescript
export let howManyAreCommand = async (
inputGuildObject:Discord.Guild, 
inputIDs:string[], 
inputMessage:Discord.Message) => {

  let checkedMembers = await inputGuildObject.members.fetch();
  let usersWithRolesArray:string[] = []
  
  checkedMembers.forEach((member) => {
    if (memberHasAllRolesById(member, inputIDs)){
      usersWithRolesArray.push(member.user.username);
    }
  })
  let roleNameArray = returnRoleIdNameArrayToPost(inputIDs)
  
  inputMessage.channel.send(
    `Number of users with the following roles[${roleNameArray.join(", ")}] : ${usersWithRolesArray.length}`)
}
```

The initial commit's `whois` command is similar in structure, so I will mention the relevant differences:
- The `updateArrayForWhoIs` is a similar function to `updateArrayForHowManyAre`, but the array is populated with strings containing information that fit the following format: `"DiscordUser#1234 (Nickname)"`.
- The ternary operator defines the `nickname` variable as the user's nickname if they have one, or their username if they do not have a nickname for the server.
- After `iterateOverMembersAndReturnData` executes, the array of username strings is concatonated into a string.
- Then, we check if that string's length is over 2000 characters long, the Discord post length limit. If it is under the limit, the post is made as usual. If it is over the limit, then the post is rendered as a text file, and attached to the post, along with an appropriate error message.
```typescript
case "whois": // =whois @role @role .. etc
        if (!firstArgId) {
          return;
        }

        let updateArrayForWhoIs = (
          inputMember: Discord.GuildMember,
          outputArray: string[]
        ): void => {
          let nickname = inputMember.nickname
            ? inputMember.nickname
            : inputMember.user.username;
          outputArray.push(
            `${inputMember.user.username}#${inputMember.user.discriminator} (${nickname})`
          );
          // Populates array with: "DiscordUser#1234 (Nickname)"
        };

        let returnUsers = [];

        iterateOverMembersAndReturnData(
          returnUsers,
          currentGuildObject,
          memberHasRolesFromArgs,
          updateArrayForWhoIs
        ).then((returnedArray) => {
          let computedPost = returnedArray.join(", ");
          let tooManyUsersToReturn = computedPost.length > 2000;
          if (!tooManyUsersToReturn) {
            message.channel.send(`${computedPost}`);
          } else {
            fs.writeFileSync("./whoIsFile.txt", computedPost);
            let fileToAttach = new Discord.MessageAttachment("./whoIsFile.txt");
            message.channel.send(
              `Error! There are too many users that match your search! File has been generated with list of users.`,
              fileToAttach
            );
          }
        });
        break;
```
asdf2



```typescript
async function arbitraryArgumentBotCommands(inputGuildObject:Discord.Guild, commandInput) {
  let inputIDs:string[] = returnIdArrayFromArgs(args)

    switch(commandInput){
      
      case "howmanyare": // =howmanyare @role @role .. etc
        howManyAreCommand(inputGuildObject, inputIDs, currentMessage)
        break;

      case "whois": // =whois @role @role .. etc
        whoIsCommand(inputGuildObject, inputIDs, currentMessage)
        break;
      }
    }
 ```

[> Summary](../summary.md)
