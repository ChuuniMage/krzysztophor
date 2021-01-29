## Orthodox Defender Project Overview

[<< Return to Main Page](../index.md)

This a blog detailing my first projects learning Typescript – a Discord bot, implementing the the Discord.js API. These posts showcase the code's development from my initial commit to github, up to the production release.

The purpose of the discord bot is to provide custom quality-of-life commands for the Orthodox Christian discord server’s Administrators & Moderators, and to get practical experience developing a program that has practical use.

If you want to look at the code in its public release, the github repository is [here](https://github.com/ChuuniMage/OrthodoxDefender).

[Introduction](introduction.md)

[Folder Structure, Imports and Dependencies](importsSection.md)

[Initialisation & Message Initialisation](initialisationAndOnMessage.md)

[Utilities](utilities.md)

- [chanUtils.ts - The Channel Utilities](utilities/chanUtils.md)
- [argUtils.ts - The Argument Utilities](utilities/argUtils.md)
- [roleUtils.ts - The Role Utilities](utilities/roleUtils.md)
  - [The iterateOverMembersAndReturnData Function](utilities/iterate.md)

[The Bot Commands](botCommands.md)

- [Arbitrary argument commmands](commandDev/arbitraryArgs.md)
- [Two argument commmands](commandDev/twoArgs.md)
- [One argument commmands](commandDev/oneArg.md)
  - [The Warn Command](commandDev/warnCommand.md)
- [Zero argument commmands](commandDev/zeroArgs.md)

[Summary](summary.md)
