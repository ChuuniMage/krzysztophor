## Introduction

[<< Back to Project Overview](../defenderIndex.md)

A problem prompted this update: Sometimes, the bot would not perform its command, and if it would perform a command which posted a message alongside the command, it would sometimes fail to post the accompanying message.

The bot would parse messages, like this with no issue:
`=kick @user for being a blockhead`

However, if it were to be passed a message like this:

`=kick  @user for being a blockhead`

It would not parse. Why? There are two spaces, not one, between the `=kick` command and the `@user` being kicked.

So then, an overhaul of the input parsing system was needed. While I was at it, I refactored the architecture, because why not leave a place better than you found it.

[>> Changes to the Index File](indexFile.md)
