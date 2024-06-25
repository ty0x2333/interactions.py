---
search:
  boost: 3
---

# Error Tracking

So, you've finally got your bot running on a server somewhere.  Chances are, you're not checking the console output 24/7, looking for exceptions.

You're going to want to have some way of tracking if errors occur.

# Sending inline tracebacks

By default, if a command throws an uncaught exception, it'll send the traceback to the user.  This is very useful when in development, but doesn't help you once you've gone public, and might not be in the same servers as your errors.  Non-technical users may also find it confusing to see trackbacks instead of user-friendly error messages.

If you wish to turn this off, create your client with `Client(..., send_command_tracebacks=False)`


# The simple and dirty method

!!! Please don't actually do this.

The most obvious solution is to think "Well, I'm writing a Discord Bot.  Why not send my errors to a discord channel?"

```python
from interactions.api.events import Error

@listen()
async def on_error(error: Error):
    await bot.get_channel(LOGGING_CHANNEL_ID).send(f"```\n{error.source}\n{error.error}\n```")
```

And this is great when debugging.  But it consumes your rate limit, can run into the 2000 character message limit, and won't work on shards that don't contain your personal server.  It's also very hard to notice patterns and can be noisy.

# So what should I do instead?

interactions.py contains built-in support for Sentry.io, a cloud error tracking platform.

To enable it, call `bot.load_extension('interactions.ext.sentry', dsn=SENTRY_DSN)` as early as possible in your startup. Load this extension before your own extensions, so it can catch intitialization errors in those extensions. `SENTRY_DSN` is provided by your Sentry.io project and should look something like `https://...@o9253.sentry.io/1048576`.

# What does this do that vanilla Sentry doesn't?

We add some [tags](https://docs.sentry.io/platforms/python/enriching-events/tags/) and [contexts](https://docs.sentry.io/platforms/python/enriching-events/context/) that might be useful, and filter out some internal-errors that you probably don't want to see.
