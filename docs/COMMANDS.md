## Commands
Use `mitsuba help` to get a list of commands and their descriptions, `mitsuba help COMMAND` to see the options specific to each command.

### List
`mitsuba list`

Returns the list of boards currently known to the archiver. Includes "removed" boards that are not being archived, which are set as Enabled: False.

### Add
`mitsuba add BOARD [options]`

Adds a board to the database if not present, and sets it to enabled. Next time the archiver is started it will archive this board as well.
If the board was not present or disabled when this command was run, and Mitsuba's archiver is currently running, you need to restart it before the board actually starts being archived.

You can set some options for the board, which are stored in the database. `help add` to get more information on the options.
If you don't specify an option, it will be set to the default value. Even if the board was already in the database, and the setting was different, if you use `add`, the setting will be reset to the default value unless you specify a different value.

### Remove
`mitsuba remove BOARD`

Does *not* remove a board from the database, does *not* delete any of its data, posts or images.

What it *does* do is **disable** a board. That is, it stops being archived. The archiver for this board will complete its current cycle if it's in the middle of one, and then shut down. Other boards will not be affected.

This also does not affect the Web UI or the API. Data that was already archived for this board is still served like normal, just it won't be updated.

You can enable a board again by using `add`, however this doesn't apply until you restart mitsuba.

### Start
`mitsuba start`

Starts the archivers and the web UI and web API server. There are various settings you can check with `help start`, most of them are settings for the rate limiter.

The option `--archiver-only=true` will only start the archiver without the API or web UI. This is useful if you want to start the web UI/API separately using `start-read-only`, this setup allows you to restart or stop the archiver without causing any disruption to the public facing website.

### Start Read Only
`mitsuba start-read-only`

Starts the web UI and API, without the archivers. You can run as many instances of Mitsuba in this read-only mode as you wish.
