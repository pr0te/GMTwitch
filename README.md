# GMTwitch
Lightweight, open source Twitch interface for Game Maker: Studio

This interface uses only ten scripts, all vanilla code withouth any extensions or included files.

Using these scripts is simple, as soon as you understand the workflow. It's simply:

**INIT -> REQUEST INFO -> RECEIVE INFO -> UTILIZE INFO**

___

**INIT**

The first step is really only one line of code, usually in the create event of a controller object.
No arguments, parameters or requirements. No extensions to setup, no libraries or DLL's. Just one script.

```
twitch_init();
```

All this does is initiallize some variables, and more importantly create the necessary data structures
to store information we will request later.

___

**REQUEST INFO**

Next we move on to the part where we ask Twitch for stream details. This only involves two scripts.
You can drop these scripts anywhere in your code, just note that putting them in the step event is not only
wasteful, but it will probably crash the game/cause unexpected errors. You only need to call each script once
for it to request the info. Of course you can put them on a timer or use the auto update script I'll cover later.

```
twitch_stream_get_info( channel_id );
twitch_stream_get_thumbnail( channel_id, size );
```

> You'll notice both of these scripts require a single, shared parameter: *channel_id*
> This is simply the unique channel identification handle for the live stream to be hosted on. So, for example, say
> you are trying to find the details for a livestream being broadcasted at the URL: http://twitch.tv/xarrotstudios/
> To find the channel id, just take the string after the "twitch.tv/" part, not including the backslash following it
> For our example, the channel id would be *xarrotstudios*. Make sure it's represented as a string, and you're all set.

The first script makes an HTTP GET request to the exposed Twitch API, asking for the full details (JSON formatted)
for the channel id provided. This also sets up the data structure to store the payload we are waiting for.

The second script is a little trickier, but not much at all. The only requirement to use the second script is that
the first script *twitch_stream_get_info()* must have been called *and* the corresponding data received. You will
not get an error calling it before, it just doesn't do anything useful if the preceding script wasn't called.
Once you have got your info for the stream, and then you request the thumbnail properly, all it does is make a
new, separate request for a thumbnail in a specified size. You can fiddle with the sizes in that script, it's all
up to the user on what size thumbnail the server will respond with. That's it for the requests

___

**RECEIVE INFO**

This one is the easiest, but the most complex step at the same time. It's easy because it's a single, static script
that just gets plopped into the HTTP Async event. Set it and forget it. It's complex because the code inside the
script does some dirty stuff with strings and data structures to put up with errors and still keep things running.
Anyways, it just looks like this:

```
twitch_async();
```

Done. Finished. Complete. Moving on!

___

**UTILIZE INFO**

Arguably the hardest step of the whole process, only because we have a bunch of keys to throw at you, and they return
all sorts of different things. You'll find that it's actually a breeze to use once you take a look at the keys.
It's all packed into one tight script, so I thought this was the best way to keep the whole
motif of 'simple scripts, simple parameters' going smoothly. Here is said script:

```
var info = twitch_stream_find_value( channel_id, key );
```

Again, you have your *channel_id* handle, but now we are using a key to indentify the data we want to retrieve. Also,
this is the first script we've covered that has a return value. Let's see if I can cover this stuff in an neat way.
The return value is the data you want that has already been picked up by the *twitch_async()* script. It was already
stored (if you did everything right, come on there's no way to screw this up) and now we are just using this
newly acquired tool to seamlessly extract the information. If the info has not made it to us yet, say we forgot to
request the info in the first place or there's heavy traffic and we experience a delay; the return value will
be a constant Game Maker recognizes as *undefined*. So, typically you would make sure to do a quick check before you
compare anything, by making sure it's not an undefined value. If we didn't have any delays and we remembered to
write our code to request the info in the first place, we did good, so the following table will kindly explain what
each key does and what you should expect to be returned.

| Key           | Returns       | Return Type  |
|:-------------:|:-------------:|:-----:|
| "status"      | online/offline | True/False |
| "name"        | stream's channel name | String |
| "game" | game the stream is broadcasting | String |
| "url"        | URL to watch stream | String |
| "viewers" | current viewer count | String |
| "views" | total viewer count | String |
| "followers" | total follower count | String |
| "thumb_url" | URL template for preview thumbnail | String |
| "thumb" | sprite handle of thumbnail | Real |

___

That's it! Those are the core functions for the API I've written for you guys. Use it, abuse it, fork it,
spoon it. I don't care. No credit required. Just don't claim this as your own! Now, before you leave, I'll
list the last of the scripts and give a brief description of each:

```
// Place in the step event to auto update all active channel info on a timer
twitch_auto_update();

// Place anywhere to manually update all active channel information
twitch_update();

// Removes an active channel; you will not be able to use the data from this channel until requesting it again
twitch_remove( channel_id );

// Free the data structures and sprites used by this API, call when you want to stop using it
twitch_free();

// Used internally. You will probably never use this script, up until the heat death of the universe.
twitch_parse();
```

___

Have fun with it guys!