# Imports, Dependencies and Folder Structure

[<< Back to Project Overview](defenderProject.md)

[< Back to Introduction](introduction.md)

Every programming project needs an main index file, to define the primary behavior of the program. This post details the imports & dependencies of the this main index file, as well as the folder structure of the project that reflects these imports & dependencies. Since this post is for the imports, dependencies, and folder structure, the details of the functions and imports in question will be skimmed over for now.

For the initial commit, these are the imports:
- A config file that contains bot keys & server IDs
- The fs library for reading & writing files with node
- The 'Discord.js' library, since this of course is a discord bot
- A slew of utility functions abstracted from the production of the initial commit, to declutter the main index file

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

Some problems that needed to be cleared up included:
- Importing the node File System library (`fs`) library for a single function in the main index file
- Redundant imports of the same `applyRoleByIdToUser` and `RemoveRoleByIdFromUser` functions
- Not enough functions are extracted from the body of the index file into seperate libraries

The 1.0 Release imports section looks like this:

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

1. The `fs` dependency is isolated to the chanUtils.ts file, rather than being required for the index.ts file.
2. Every command (except for Help) is abstracted into botCommands.ts, which now depends on the lower level components in Utilities.

This change in dependency structure naturally requires an update in folder structure to accomodate. The Utility functions need to be decoupled from the index file, and which should be reflected in an updated folder structure. This decoupling means the index file remains untouched if the implementation of any imported function is updated, so the high level structure of the program is stable.

The pre-release folder structure was:

- ./ Root folder with index.ts and config.json
  - index.ts contains the main function of the program
  - config.json contains information covered in the [initialisation section](initialisation.md)
    - ./Utilities folder with argUtils.ts, chanUtils.ts, and roleUtils.ts
      - argUtils.ts contains utility functions for parsing arguments with the bot
      - chanUtils.ts contains utility functions for the server channel context
      - roleUtils.ts contains utility functions for user role manipulation

The release folder structure is:

- ./ Root folder *(unchanged)*
  - ./botCommands folder with botCommands.ts
    - botCommands.ts contains bot command functions, extracted from index.ts
      - ./botCommands/Utilities folder *(unchanged)*
      
The next post will detail the initialisation of the index file, as well as the behavior every time the bot detects a message has been posted to Discord.

[>> Initialisation & behavior on Message](initialisationAndOnMessage.md)
      
