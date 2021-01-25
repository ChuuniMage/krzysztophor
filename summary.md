# Summary

[<< Back to Project Overview](defenderProject.md)

Throughout this series of blog posts I have demonstrated the following design improvements to the bot's code, from the initial commit of the github project to the production release:
- Implement better higher-level architecture, fixing multiple lower-level issues by moving the issue's upstream logic
- Less reliance on higher context-level variables, strengthening the program structure
- Extracting more functions in order to:
  - Declutter the main index file
  - Keep function behavior as pure as possible, 
  - Facilitate update of lower-level details without any danger of changing the higher-level architecture
- Remove repeated boilerplate by implementing better architecture (ie, constant repetition of ‘if (!firstArgId) { return }’ 
- Remove extraneous setup in the Message Initialisation portion of the program, increasing performance
- Converting maps into arrays to work on them, rather than using the appropriate methods to directly work with maps
- Add helpful abstraction when it leads to increased code brevity and intellisense help
- Remove counterproductive over-abstraction

Thank you for reading these posts about my bot project. I look forward to working on more projects like this to increase my programming skill, and acheive gainful employment in web development. :)
