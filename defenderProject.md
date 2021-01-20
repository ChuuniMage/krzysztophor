## Project Overview

This a blog detailing one of my first projects learning Typescript – a Discord bot, implementing the the Discord.js API. I’m going to showcase the code's development from my initial commit to github, up to the current commit. he code underwent a lot of revisions from early development, and 

The purpose of the discord bot is to provide custom quality-of-life commands for the Orthodox Christian discord server’s Administrators & Moderators. 

[Folder Structure, Imports and Dependencies](importsSection.md)

[Initialisation](initialisation.md)

[Behavior on Message](onMessage.md)

[The Bot Commands](botCommands.md)

[Summary & Overview](summary.md)

The purpose of further development & refactorisation was to 

The main design flaws I would nut out are:
1. Reliance on higher level context variable, rather than passing in the variable as an argument
2. Repeated boilerplate (Constant repetition of ‘if (!firstArgId) { return }’ and ‘await currentGuildObject.whatever.fetch(variable)’ which clutter up the code. Prior to the initial commit, the design pattern was to fetch the guild object in every Bot Command.
3. Too much setup in the client.on("message"…	   portion of the program
4. Not enough function extraction into other modules to declutter the main index.ts file
