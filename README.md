# Mitsuba

![screenshot](https://i.ibb.co/B2K8rSy/mitsuba-rust.png)

Mitsuba is a lightweight 4chan board archiver written in Rust. It continuously monitors a set of 4chan boards, fetches new posts, thumbnails, and optionally full images, and makes them available through an imageboard web UI as well as a read-only JSON API that is compatible with 4chan's [official API](https://github.com/4chan/4chan-API).

Mitsuba's main goal is to be very lightweight in terms of CPU and memory usage, and Rust helps accomplish this goal. Mitsuba is designed to be easy to deploy and doesn't currently have any runtime dependencies besides needing a PostgreSQL database.

The intended usage is self-hosting an archive on a low budget, however the Actix based web UI and API are quite performant, and should be capable of scaling to any amount of readers, with much lower resource consumption compared to competing frameworks in other languages, and without the possible latency spikes caused by garbage collection.

Mitsuba does not support "ghost posting" as it's not an imageboard engine. This could be supported in the future with some work (mostly on the front-end) but it requires actual administration tools and an accounts system, neither of which are actually present. What few options Mitsuba has are set through the CLI.

| Archive | Boards | Country | Cloudflare |
|-|-|-|-|
| [mitsuba.eientei.xyz](https://mitsuba.eientei.xyz) | biz,lit,sci,g | ?? | ?? |
| [mitsuba.world](https://mitsuba.world) (running by original dev) | only /a/ fully archived | ?? | ?? |
## Features
- Very quick and easy to set up
- No runtime dependencies except a running Postgresql database.
- Single static executable, all assets and dependencies embedded.
- Extremely lightweight, can run on a budget VPS.
- Fully integrated: Mitsuba archives boards, threads, and images, serves them through a JSON API and Web UI all in one.
- Easy administration with a few CLI commands
- Configurable rate limiter
- Optional full image download setting per-board
- Web UI has a field that lets you jump to any post by typing its ID and selecting the board
- Sha256 image deduplication, doesn't rely on 4chan's MD5 hash
- Support for S3-compatible image storage backend
- Reduced database writes: the hash of every post is kept in memory, if a post hasn't changed, no DB operation is performed
- Can find an image from its original 4chan URL. `https://i.4cdn.org/po/1546293948883.png` can be found on mitsuba at `/po/1546293948883.png`
- Can be configured to load balance requests to 4chan between multiple proxies with different weights, to bypass rate limits

There are some important features missing:
- No "ghost posting" or posting of any kind. Read only archive.
- No full text search, or any search really. Can only get to a post or thread from the ID. We want to have search eventually.
- No admin UI, administration CLI only (but there are only a few things you'd want to change anyways)
- No administration tools to delete individual posts or images (but you can safely delete an image file from the folder if necessary, it won't be downloaded again)
- No account system whatsoever (but this makes it inherently secure)
- No tools for deleting all posts or images from a particular board (This is a planned feature, for now you could just run one instance per board)

## Dependencies
You need to have a PostgreSQL instance available somewhere Mitsuba can reach it with the DATABASE_URL env variable provided.
If you get an error about the server not accepting any more connections on startup, you might need to increase your database's `max_connections` configuration.

If you want to build with custom memory alloc like `mimalloc` then in features type `cargo build --release --feature mimalloc`
This command will produce executable with mimalloc as memory allocator, with disabled secure mode.

## Quick Setup
```
export DATABASE_URL="postgres://user:password@127.0.0.1/mitsuba"
export RUST_LOG=mitsuba=info # Optional, to get feedback
mitsuba add po
mitsuba start
```
After some threads have been archived, you can visit http://127.0.0.1:8080/po/1 to see your new archive for the /po/ board.

This will only get posts and thumbnails but not full images.

Use `mitsuba add po --full-images=true` to change that.

Mitsuba will not fetch full images for a post it has already archived previously, unless it has to visit that post again. Currently this means if you enable full images on a board you were already archiving with only thumbnails, Mitsuba won't fetch full images for posts on threads until that particular thread gets a new post.

`mitsuba add BOARD` and `mitsuba remove BOARD` are safe to use while mitsuba is running. But in that scenario, they will not take effect until the current archive cycle is completed.

If you would like, to take a look at full setup guide.

## Web UI
The web UI is currently made to look and work exactly like 4chan's Yotsuba Blue theme, except it's the same on every board (even non-worksafe boards).
We also include 4chan's official default "inline extension" with all of its features and options (the inline extension is licensed as MIT).

The JS code has been slightly modified through trial and error to work correctly with Mitsuba. If you find an issue, file a bug report.

In other works, Mitsuba's UI looks and works (almost) exactly like 4chan's UI because it is exactly the same. This was simply the easiest short term solution, and it allows us to leverage the fact that our API is the same as 4chan's, which makes the inline extension work.
In the future, we want to customize the theme with different colors at least, to distinguish it, and add features that are suited to an archive.

The Web UI **also works with Javascript disabled**, just like 4chan's UI does.

Most of 4chan's features are correctly displayed in the Web UI. One limitation is that currently only real country flags are supported, and "troll flags" (eg. "Anarcho Capitalist" flag) are simply not displayed in the UI. However, these are only enabled on one board (/pol/).

Also all capcode posts (Mod, Admin, Manager, etc) will be displayed the same as "Moderator" with no distinction between roles.
These posts are extremely rare anyways (besides, Moderator, which is displayed correctly), and I don't think there are even any up currently, and the distinction doesn't seem super important.
Might be fixed eventually.

The Flash board also has some features we haven't implemented but Flash is dead anyway.
### Note:
Currently the web UI is very inflexible when it comes to URLs. For example for a thread, you have to visit `/[board]/thread/[id]` , something like `/po/thread/570368/welcome-to-po` will get you a 404 because of the trailing `welcome-to-po`.

Index pages need to have the index number in the URL explicitly, so `/po/1` for example works, but `/po/` by itself returns 404.
This is WIP.
Also, there is no homepage `/` whatsoever.

### Images
In addition to the API being compatible, we also support getting the images from the same paths 4chan uses.

For example, if an image on 4chan is served from https://i.4cdn.org/po/1546293948883.png, Mitsuba will serve it under `/po/1546293948883.png` and its corresponding thumbnail will be `/po/1546293948883s.jpg` just like on the original site.

This allows you to get an image from the archive *even if you only have the original link* (which might be dead now) and don't know the post or thread ID.

Note that this is intended to help you find lost images, but it's not the correct way to serve them to many users. **Visiting these links involves a database lookup every time** because we store images differently compared to how 4chan does it.

You can see the correct link for static images being used on the web UI. The URL is based on the image's MD5 hash as supplied by 4chan, but encoded in Base32.

For example, for the previous image, the link to the full image would be `/img/full/XG/XGKR4ZPAOXQFKUPYY43ETQPPKQ.png` , the thumbnail is at `/img/thumb/XG/XGKR4ZPAOXQFKUPYY43ETQPPKQ.jpg` .

The `/img/` path serves all images directly from disk. Mitsuba looks in your `DATA_ROOT` folder, which is `data` by default, and serves the `images` folder within from this path (`/img/`). So you can find all the images in there.

## Administration
Mitsuba does not come with any admin UI besides the CLI commands above.

There are no commands to purge a board from the database or delete its images.

Images are stored in the same folders for all boards so you can't easily remove all the images just for one board, and some images might have been posted on multiple boards.

Thumbnails and full images **are** stored in different folders (`full` and `thumb`) so you can delete all full images (for all boards) by just deleting that entire folder if you want to.

However, if you really need to delete a particular image, you can just delete the file. The images on disk are never read, and whether an image has been downloaded or not is tracked in the database. So Mitsuba doesn't know that the image was removed from disk, and will never download it again, because it would be marked as already present.

It's safe to delete any of the images on disk, but of course they will return 404s if someone tries to access them.

We want to eventually have some convenient CLI tools for administration. Maybe a web UI in the future, but that's a bigger endeavour.

## Future

Some features that might be added:
- Search: this is by far the biggest missing feature. I wanted to have this in 1.0 but I didn't have the time. Ideally, we should have full text search for post content and titles, names and such, plus all the advanced search options foolfuuka has. There are multiple ways to go about this, right now I am considering two. The first option is to use Postgresql full text search. This is limited, but it actually does have all the features realistically needed for our use case, and it doesn't add any external dependencies. This would ensure search is always available. The second option is using `meilisearch` which is a rust full text search engine that is both lightweight and easy to set up, it works well out of the box. This would give us more capabilities, it is very flexible. But it's an external dependency that users would have to download and run separately like Postgresql itself. So I am hesitant to add it, and it would have to be optional, making search not always available. On the other hand, using something like elasticsearch would be a huge dependency that I'm not willing to take, and it requires a lot more configuration. Also it's the opposite of lightweight among web servers.
- `purge BOARD` command, to delete all archived data relative to a particular board. Would delete all posts, and all images that were only posted on that board, while preserving images also present on others. Would have option to only delete full images and preserve thumbnails and post data.
- `hide ID` command, to hide a particular post from being served by the API, or an entire thread if the ID is of a thread OP. Would simply mark the post as "hidden" on the database, without deleting, it, but it would no longer be publically visible. Mainly meant in cases where someone was doxxed and asked to have the info taken down. This would also have an option to delete the images associated with the post or thread that aren't present on other posts, while still keeping them marked as downloaded, so they would not be downloaded again.
- `check` command, to check the image store for corruption or missing images. This would first scan the database to see all images that are marked as downloaded, and then ensure that they exist on disk, hash the files to compare MD5s to make sure they are not corrupted. If an image is missing or corrupted, it would correct the database, and mark it as absent. Might also delete images that are on disk but aren't tracked in the database.
- Object storage (S3 compatible) option to store images. This is not very difficult to implement by itself, but it requires handling more error cases, for example retrying when an upload to the object storage provider fails. Right now if an image fails to be saved to disk, it is dropped immediately and no retries are made. But with a remote storage provider more things can fail, and they might only fail temporarily.

At the moment a full imageboard engine with posting and administration is considered out of scope, however if you are interested in working on that, you should make an issue to discuss it.

## Thanks
Thanks reasv, for originally creating Mitsuba.
