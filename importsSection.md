# Imports, Dependencies and Folder Structure

[<< Back to Project Overview](defenderProject.md)

[< Back to Introduction](introduction.md)

Every programming project needs an main index file, to define the primary behavior of the program. This post details the imports & dependencies of this main index file, as well as the folder structure of the project. The details of the functions will be addressed in later posts.

For the initial github commit, these are the imports:
- A `config.json` file that contains the bot keys, to hook the bot into the code
- The `fs` node File System library for reading & writing files with node
- The `Discord.js` library, required for a discord bot written in javascript
- A slew of utility functions abstracted during the production of the initial commit, to declutter the main index file

```typescript
const config = require("./config.json");
const fs = require("fs");
import * as Discord from "discord.js";
import { extractNumbersForId 
} from "./Utilities/argUtils";
import { postInNamedChannel 
} from "./Utilities/chanUtils";
import { iterateOverMembersAndReturnData } from "./Utilities/roleUtils";
import {
  memberHasAnyRoleByName,
  applyRoleByNameToUser,
  removeRoleByNameFromUser,
  getRoleByName,
  memberHasAllRolesById,
} from "./Utilities/roleUtils";
import {
  applyRoleByIdToUser,
  removeRoleByIdFromUser,
} from "./Utilities/roleUtils";
```

I noticed these problems needed to be cleared up:
- Importing the entire `fs` library for a single function in the main index file
- Redundant imports of the same `applyRoleByIdToUser` and `RemoveRoleByIdFromUser` functions
- Not enough functions are extracted from the body of the index file into seperate libraries

The production release imports section looks like this:

```typescript
const config = require("./config.json");
import * as Discord from "discord.js";
import { 
  extractNumbersForId,
} from "./botCommands/Utilities/argUtils";
import {
  memberHasAnyRoleByName,
} from "./botCommands/Utilities/roleUtils";
import {
  postBoostInfoCommand,
  joinCommand,
  checkPFPCommand,
  kickUserCommand,
  banUserCommand,
  replaceAllRolesCommand,
  howManyAreCommand,
  whoIsCommand,
  quarantineCommand,
  unQuarantineCommand,
  warnCommand,
  msgCommand,
  dmsgCommand
} from "./botCommands/botCommands";
```

I made two major changes:
1. The `fs` dependency is isolated to the `chanUtils.ts` file, rather than being required for the `index.ts` file.
2. Every command (except for `help`) is abstracted into `botCommands.ts`, which now depends on the lower level components in Utilities.

The folder structure of a project should naturally reflect the internal dependency structure of the project. The Utility functions, extracted from the index file, have their own folder. This decoupling means that, if the internal behavior of the utility functions changes, the index file does not need to be touched at all, lending structural stability to the project.

The pre-release folder structure was:

- ./ Root folder with `index.ts` and `config.json`
  - `index.ts` contains the main function of the program
  - `config.json `contains information covered in the [next post](initialisation.md)
    - ./Utilities folder with `argUtils.ts`, `chanUtils.ts`, and `roleUtils.ts`
      - `argUtils.ts` contains utility functions for parsing arguments with the bot
      - `chanUtils.ts` contains utility functions for the server channel context
      - `roleUtils.ts` contains utility functions for user role manipulation

The release folder structure is:

- ./ Root folder *(unchanged)*
  - ./botCommands folder with `botCommands.ts`
    - botCommands.ts contains bot command functions, extracted from index.ts
      - ./botCommands/Utilities folder *(unchanged)*
      
This is because, apart from a couple of imported utilities, all of the utilities are now exclusively dependencies of the `botCommands.ts` file. Now, the internal functioning of each command can change.

The next post will detail the initialisation of the index file, as well as the behavior every time the bot detects a message has been posted to Discord.

[>> Initialisation & Message Initialisation](initialisationAndOnMessage.md)
      
