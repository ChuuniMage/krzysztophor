## Imports, Dependencies and Folder Structure, 

[<< Back to Project Overview](defenderProject.md)

```typescript
const config = require("./config.json");
const fs = require("fs");
import * as Discord from "discord.js";
import { extractNumbersForId } from "./Utilities/argUtils";
import { postInNamedChannel } from "./Utilities/chanUtils";
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
- importing the 'fs' library for a single function in the main index file
- repeated imports of the same 
- partial extraction of functions from

The final imports section looks like this

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

1. Dependencies for lower level components have been abstracted, and extraneous imports have been removed
2. The fs dependency is isolated to the chanUtils.ts file, rather than being required for the index.ts file.
3. Every command (except for Help) is abstracted into the botCommands.ts file

This change in dependency structure naturally requires an update in folder structure to accomodate.

The old folder structure is

- ./ Root folder with index.ts and config.json
  - index.ts contains the main function of the program
  - config.json contains information covered in the [initialisation section](initialisation.md)
    - ./Utilities folder with argUtils.ts, chanUtils.ts, and roleUtils.ts
      - argUtils.ts contains utility functions for parsing arguments with the bot
      - chanUtils.ts contains utility functions for the server channel context
      - roleUtils.ts contains utility functions for user role manipulation

The new folder structure is

- ./ Root folder *(unchanged)*
  - ./botCommands folder with botCommands.ts
    - botCommands.ts contains bot command functions, extracted from index.ts
      - ./botCommands/Utilities folder *(unchanged)*
      
With the addition of the botCommands folder, where the Utilities folder has been relocated, a key architectural improvement has been implemented: Now, most of the utility functions are dependencies of botCommands.ts, not the index file. This decoupling means the index file can remain untouched if any of the inner workings of the bot functions needs updating, and if a new feature is implemented, the index file needs less change.
