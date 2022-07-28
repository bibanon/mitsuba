## API
Mitsuba features a read-only JSON API that is designed to be compatible with 4chan's [official API](https://github.com/4chan/4chan-API).
Because of that, we will not fully document it here, since that would be redundant. You can read their documentation, because the URIs and data returned are mostly the same. There are a few (non-breaking) differences that are explained below (of course, besides the difference that this API is not served from 4chan's domains, but instead from your own server).

First, currently, only the [Threads](https://github.com/4chan/4chan-API/blob/master/pages/Threads.md) and [Indexes](https://github.com/4chan/4chan-API/blob/master/pages/Indexes.md) endpoints have been implemented.
In addition to that, we added an endpoint to serve individual posts.

- Threads: `/[board]/thread/[op ID].json` Serves a full 4chan thread, in the same format the official API uses.
- Indexes: `/[board]/[1-...].json` Serves the content of a board's index page. This is the default page you see when you visit a board, for example https://boards.4channel.org/po/ . On 4chan there are normally only 15 index pages, going for example from  `/po/1.json` to `/po/15.json`. On Mitsuba, since old threads are never deleted, there are as many pages as are needed to list all of the threads currently on the archive. Once there are no more threads, higher index numbers will return a 404 status code. This means you can easily scrape a mitsuba archive getting progressively higher indexes until it 404s. However note that the order is the same as on 4chan, so it's not guaranteed to remain consistent. The order is based on which thread has had the most recent new post, not when the thread was first archived. Also pages don't contain full threads, but only the OP and a few posts.

In addition to these endpoints, we have implemented a `/[board]/post/[ID].json` endpoint that just serves a single post by itself. Using this, you can get a post even if you only know its own ID and not the OP's.

There's also one extra endpoint that's entirely specific to Mitsuba: `/boards-status.json`, this returns the same data as the CLI's `list` command, but in JSON format.

### Differences with 4chan API
The main difference with 4chan's API is that every post also contains a `board` field with the name of the board it is in.

In addition to that, in every situation where 4chan's API would not return a field, we instead return that field with a default value.

The only types present in 4chan's API are integers and strings, so for integers the value would  `0` and for strings it's an empty string.
This means that all JSON posts returned by our API are guaranteed to contain *all 37 fields* that a post could contain, no matter the board or type of post.
For example, on 4chan's side, `sticky` field is only ever set on OP posts, and its value is `1` if it is set. On our side, if it wasn't set by 4chan, it would be present but set to `0`.

In practice this should not cause any issues with existing code targeting 4chan's API as long as you are checking the truthiness of each field rather than just its presence. Arguably, this should be easier to handle in most languages, since all fields are always guaranteed to be present.

