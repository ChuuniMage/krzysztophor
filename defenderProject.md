## Project Overview

This a blog detailing one of my first projects learning Typescript – a Discord bot, implementing the the Discord.js API. I’m going to showcase the code's development from my initial commit to github, up to the current commit. he code underwent a lot of revisions from early development, and 

The purpose of the discord bot is to provide custom quality-of-life commands for the Orthodox Christian discord server’s Administrators & Moderators. 

[Folder Structure, Imports and Dependencies](importsSection.md)

[Initialisation](initialisation.md)

The main function of Index.ts is a switch statement containing the following commands:
```
=join @User | Show when a user has joined Discord, and the Server
=boostinfo | Show server’s discord boost info
=kick @User | Kick a user
=ban @User | Ban a user
=warn @User | warn a user, two-teired warning system
=quarantine @User | Apply the “Quarantine” role which restricts 
=unquarantine @User | Remove the “unquarantine” role
=msg #channel message | send message to a channel
=dmsg @User message | Send message to channel
=howmanyare @Role @Role… | Return the number of users with all roles enumerated
=whois @Role @Role… | Return list of usernames with all roles enumerated
=replaceall @RoleA @RoleB | Replace all instances of RoleA with RoleB
=checkpfp | If user has default PFP, apply “CheckPFP” role
```

The purpose of further development & refactorisation was to 

The main design flaws I would nut out are:
1. Reliance on higher level context variable, rather than passing in the variable as an argument
2. Repeated boilerplate (Constant repetition of ‘if (!firstArgId) { return }’ and ‘await currentGuildObject.whatever.fetch(variable)’ which clutter up the code. Prior to the initial commit, the design pattern was to fetch the guild object in every Bot Command.
3. Too much setup in the client.on("message"…	   portion of the program
4. Not enough function extraction into other modules to declutter the main index.ts file
