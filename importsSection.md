Root folder with index.ts
->botCommands folder, with botCommands.ts
 ->Utilities folder, with argUtils.ts, chanUtils.ts, and roleUtils.ts

The old folder structure is

Root folder with index.ts
->Utilities folder, with argUtils.ts, chanUtils.ts, and roleUtils.ts

While a small difference, this important change of structure decoupled the nuts & bolts of how the bot functions worked, as well as teh dependencies for those functions, with the main body of the program.

```
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

```const config = require("./config.json");
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
} from "./botCommands/botCommands";```

So what this shows is
1. Extraneous imports are cleaned up
2. The dependencies for the imports have been 
1. The fs dependency is isolated to the chanUtils.ts file, rather than being required for the index.ts file.
2. Every command (except for Help) is abstracted into the 
