# One Argument Commands

[<< Back to Project Overview](../defenderProject.md)

One argument commands blah blah

This is old join command

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
