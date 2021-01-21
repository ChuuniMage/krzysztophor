# Summary

[<< Back to Project Overview](defenderProject.md)

The purpose of further development & refactorisation was to 

The main design flaws I would nut out are:
1. Reliance on higher level context variable, rather than passing in the variable as an argument
2. Repeated boilerplate (Constant repetition of ‘if (!firstArgId) { return }’ and ‘await currentGuildObject.whatever.fetch(variable)’ which clutter up the code. Prior to the initial commit, the design pattern was to fetch the guild object in every Bot Command.
3. Too much setup in the client.on("message"…	   portion of the program
4. Not enough function extraction into other modules to declutter the main index.ts file
