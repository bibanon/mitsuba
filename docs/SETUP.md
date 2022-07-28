## Setup Guide
Mitsuba is designed to be easy and quick to set up.
Download a binary build for your system, or clone the repository and build your executable.
Currently all static files mitsuba uses are embedded in the executable. You should just be able to run Mitsuba in an empty folder.

Some options need to be passed as environment variables. Mitsuba uses `dotenv`, which means that instead of setting the environment variables yourself, you can specify their values in a file called `.env` which must be in the directory you are running the mitsuba executable in. Mitsuba will read this file and apply the environment variables specified with their values.

You will find an `example.env` file in this repository. Copy it and rename it to just `.env`, then edit its configuration as needed.
There are a couple of settings that you need to be aware of right now:

- DATABASE_URL: you need to specify the connection URI for your Postgresql instance here. In the example file it's `postgres://user:password@127.0.0.1/mitsuba` .
   Replace with the correct username and password, as well as address, port and database name. The user needs to either be the owner of the specified database (in this case called `mitsuba`) if it already exists, or you need to create it yourself.

- DATA_ROOT: the directory in which to save the image files. This is an optional setting. If you leave it out entirely, Mitsuba will just create a "data" folder in the current working directory, and use that.

- RUST_LOG: the recommended setting for this is "mitsuba=info". This controls the mitsuba's log output. For now mitsuba uses `env_logger` which just prints the log information to standard output (the console). If this is not set, you will only see errors and warnings and no other feedback.

The only required setting is DATABASE_URL but RUST_LOG="info" is also recommended. Just use the `example.env` file contents and change the database URI.

We will refer to the executable as `mitsuba` in this guide from now on, but on Windows® it is of course called `mitsuba.exe` .

Run `mitsuba help` to get a quick usage guide with a list of possible commands:

```
$ mitsuba help
mitsuba 1.0
High performance board archiver software. Add boards with `add`, then start with `start` command.
See `help add` and `help start`

USAGE:
    mitsuba <SUBCOMMAND>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

SUBCOMMANDS:
    add                Add a board to the archiver, or replace its settings.
    help               Prints this message or the help of the given subcommand(s)
    list               List all boards in the database and their current settings. Includes
                       disabled ('removed') boards
    remove             Stop and disable archiver for a particular board. Does not delete any
                       data. Archiver will only stop after completing the current cycle.
    start              Start the archiver, API, and webserver
    start-read-only    Start in read only mode. Archivers will not run, only API and webserver
```
You can use `mitsuba help COMMAND` in order to get more detailed information on each command and the possible options:
```
$ mitsuba help add
Add a board to the archiver, or replace its settings.

USAGE:
    mitsuba add [OPTIONS] <name>

ARGS:
    <name>    Board name (eg. 'po')

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
        --full-images <full-images>    (Optional) If false, will only download thumbnails for this
                                       board. If true, thumbnails and full images. Default is false.
```
This command will add a database entry for the specified board and its settings. Next time Mitsuba is run, an archiver will start for this board and any other boards you added with `add`.

As you can see there is only one option in terms of board specific settings.
- `full-images=true` will make the archiver download full images (and files) for that board. The default is `false`, meaning only thumbnails will be downloaded.

Note that any time you use `add` on a board that was already added before, it enables that board if it was disabled with `remove`, and *replaces* the configuration for that board with the values you specify, or the defaults. The previous settings are **ignored**. So if you had full image download enabled on /po/ previously with a wait time of 100, and then do `mitsuba add po`, the settings will be reset to the default of no full image download, and wait time of 10.

So let's add our first board, /po/ is a good example because it's the slowest board on 4chan most of the time:
```
mitsuba add po --full-images=true
Added /po/ Enabled: true, Full Images: true
If mitsuba is running, you need to restart it to apply these changes.
```
We will get all images, and we'll look for new thread updates 10 minutes after the previous update completes.
Let's confirm our changes with the `list` command which lists all boards added to mitsuba.
```
$ mitsuba list
/po/ Enabled: true, Full Images: true
1 boards found in database
```
- `Enabled` means the board is currently being actively archived.

Finally, we take a look at the `start` command:
```
$ mitsuba help start
Start the archiver, API, and webserver

USAGE:
    mitsuba start [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
        --archiver-only <archiver-only>        (Optional) If true, will only run the archiver and
                                               not the web ui or the web API. If false, run
                                               everything. Default is false.
        --burst <burst>                        (Optional) Max burst of requests started at once.
                                               Default is 10.
        --jitter-min <jitter-min>              (Optional) Minimum amount of jitter to add to rate-
                                               limiting to ensure requests don't start at the same
                                               time, in milliseconds. Default is 200ms
        --jitter-variance <jitter-variance>    (Optional) Variance for jitter, in milliseconds.
                                               Default is 800ms. Jitter will be a random value
                                               between min and min+variance.
        --max-time <max-time>                  (Optional) Maximum amount of time to spend retrying a
                                               failed image download, in seconds. Default is 600s
                                               (10m).
        --rpm <rpm>                            (Optional) Max requests per minute. Default is 60.
```
This command starts the archiver(s) for the boards we added, as well as the web UI to browse the archives and the JSON API.
There are various options, all are related to rate-limiting for the archiver, except one: `archiver-only`.

`--archiver-only=true` will start mitsuba as just an archiver. Posts will be fetched, images downloaded, but there is no web UI or API to actually see the archived content. This option is not particularly useful, but it will use **considerably** less RAM, going down to ~9-17MB.
You can run the web UI and API separately with `mitsuba start-read-only`. You can even run multiple instances of the web UI at the same time if you want, as it's read-only like the name implies.

`--rpm` is the main rate-limiter option. It decides how many requests per minute are performed by mitsuba against 4chan's API and image servers, globally.
All of the rate limiter settings are global for all boards and images, but note that images and API-calls (which are used to fetch threads) are counted **separately** in the rate limiting.

This means that if you set say, RPM to 60, mitsuba will perform 60 requests per minute (at most, on average it will be much less depending on how many threads get updated) against 4chan's API to fetch new threads it finds, **and** it will do 60 requests per minute to fetch images, for a global total of 120 requests per minute done at most across all boards and all images. This separation ensures that even if there's a large backlog of images to download, mitsuba can continue to fetch new posts at the same time, without the two interfering with each other.

We can now start our archiver with `mitsuba start`.
It will start its work of archiving the board, /po/.
After a while you can browse your archive by visiting http://127.0.0.1:8080/po/1

You can start a read-only instance of mitsuba, which will only serve the web UI and API but not archive anything, with `mitsuba start-read-only`.

You can separate the archiver and the public web UI/API by running the archiver by itself using `mitsuba start --archiver-only=true`, and then launch `mitsuba start-read-only` separately. This means you can stop or restart the archiver, add new boards to be archived, without any visible disruption to users who are just browsing your archive.
You can also run multiple instances of `mitsuba start-read-only` if you want, or run the full archiver and web UI with `mitsuba start` along with additional read-only instances.
