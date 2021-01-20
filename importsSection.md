## Folder Structure, Imports and Dependencies

The old folder structure is

- ./ Root folder with index.ts and config.json
  - index.ts contains the main function of the program
  - config.json contains information covered in the [initialisation section](initialisation.md)
    - ./Utilities folder with argUtils.ts, chanUtils.ts, and roleUtils.ts
      - argUtils.ts contains utility functions for parsing arguments with the bot
      - chanUtils.ts contains utility functions for the server channel context
      - roleUtils.ts contains utility functions for user role manipulation

The new folder structure is

- Root folder (same as before)
  - botCommands folder with botCommands.ts
    - botCommands.ts contains bot command functions, extracted from index.ts
      - Utilities folder (same as before)

Which is identical to the previous structure, except for the botCommands folder: Now, most of the utility functions are dependencies in the botCommands folder, not the index file. While a small difference, this decoupling means less fiddling with the index file in order to update the inner behavior of one of its sub-features.

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

So what this shows is
1. Extraneous imports are cleaned up
2. The dependencies for the imports have been 
1. The fs dependency is isolated to the chanUtils.ts file, rather than being required for the index.ts file.
2. Every command (except for Help) is abstracted into the 
