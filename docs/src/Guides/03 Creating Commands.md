---
search:
  boost: 3
---

# Slash Commands

So you want to make a slash command (or interaction, as they are officially called), but don't know how to get started?
Then this is the right place for you.

## Your First Command

To create an interaction, simply define an asynchronous function and use the `@slash_command()` decorator above it.

Interactions need to be responded to within 3 seconds. To do this, use `await ctx.send()`.
If your code needs more time, don't worry. You can use `await ctx.defer()` to increase the time until you need to respond to the command to 15 minutes.
```python
from interactions import slash_command, SlashContext

@slash_command(name="my_command", description="My first command :)")
async def my_command_function(ctx: SlashContext):
    await ctx.send("Hello World")

@slash_command(name="my_long_command", description="My second command :)")
async def my_long_command_function(ctx: SlashContext):
    # need to defer it, otherwise, it fails
    await ctx.defer()

    # do stuff for a bit
    await asyncio.sleep(600)

    await ctx.send("Hello World")
```
???+ note
    Command names must be lowercase, can only contain `-` and `_` as special symbols, and must not contain spaces.

By default, commands are assumed to be global, meaning they can be used by every server the bot is in. When testing, it is recommended to use non-global commands.
For that, you can either define `scopes` in every command or set `debug_scope` in the bot instantiation which sets the scope automatically for all commands.

You can define non-global commands by passing a list of guild ids to `scopes` in the interaction creation.
```python
@slash_command(name="my_command", description="My first command :)", scopes=[870046872864165888])
async def my_command_function(ctx: SlashContext):
    await ctx.send("Hello World")
```

For more information, please visit the API reference [here](/interactions.py/API Reference/API Reference/models/Internal/application_commands/#interactions.models.internal.application_commands.slash_command).

## Subcommands

If you have multiple commands that fit under the same category, subcommands are perfect for you.

Let's define a basic subcommand:
```python
@slash_command(
    name="base",
    description="My command base",
    group_name="group",
    group_description="My command group",
    sub_cmd_name="command",
    sub_cmd_description="My command",
)
async def my_command_function(ctx: SlashContext):
    await ctx.send("Hello World")
```

This will show up in discord as `/base group command`. You may also wish to drop the `group` part to have a `/base command` subcommand, which you can do by removing the `group_name` and `group_description` parameters (they are optional).

There are more ways to add additional subcommands to the base/group from there:

=== ":one: Decorator"
    ```python
    @my_command_function.subcommand(
        group_name="group",
        group_description="My command group",
        sub_cmd_name="second_command",
        sub_cmd_description="My second command",
    )
    async def my_second_command_function(ctx: SlashContext):
        await ctx.send("Hello World")
    ```

=== ":two: Repeat Definition"
    ```python
    @slash_command(
        name="base",
        description="My command base",
        group_name="group",
        group_description="My command group",
        sub_cmd_name="second_command",
        sub_cmd_description="My second command",
    )
    async def my_second_command_function(ctx: SlashContext):
        await ctx.send("Hello World")
    ```

    **Note:** This is particularly useful if you want to split subcommands into different files.

=== ":three: Class Definition"
    ```python
    from interactions import SlashCommand

    base = SlashCommand(name="base", description="My command base")
    group = base.group(name="group", description="My command group")

    @group.subcommand(sub_cmd_name="second_command", sub_cmd_description="My second command")
    async def my_second_command_function(ctx: SlashContext):
        await ctx.send("Hello World")
    ```

All of these would create a subcommand called `/base group second_command`.

???+ note
    Using subcommands makes using the base command unusable - IE, you cannot use `/base` as a normal command.

    Group subcommands follow a similar logic, as by defining a group, you can no longer create normal subcommands - IE, you cannot use either `/base` or `/base command`.

## Options

Interactions can also have options. There are a bunch of different [types of options](/interactions.py/API Reference/API Reference/models/Internal/application_commands/#interactions.models.internal.application_commands.OptionType):

| Option Type               | Return Type                                | Description                                                                                 |
|---------------------------|--------------------------------------------|---------------------------------------------------------------------------------------------|
| `OptionType.STRING`      | `str`                                      | Limit the input to a string.                                                                |
| `OptionType.INTEGER`     | `int`                                      | Limit the input to an integer between -2^53 and 2^53.                                                               |
| `OptionType.NUMBER`      | `float`                                    | Limit the input to a float between -2^53 and 2^53.                                                                 |
| `OptionType.BOOLEAN`     | `bool`                                     | Let the user choose either `True` or `False`.                                               |
| `OptionType.USER`        | `Member` in guilds, else `User`            | Let the user choose a discord user from an automatically-generated list of options.         |
| `OptionType.CHANNEL`     | `GuildChannel` in guilds, else `DMChannel` | Let the user choose a discord channel from an automatically-generated list of options.      |
| `OptionType.ROLE`        | `Role`                                     | Let the user choose a discord role from an automatically-generated list of options.         |
| `OptionType.MENTIONABLE` | `Union[Member, User, Role]`                            | Let the user chose any discord mentionable from an automatically generated list of options. |
| `OptionType.ATTACHMENT`  | `Attachment`                               | Let the user upload an attachment.                                                          |

Now that you know all the options you have for options, you can opt into adding options to your interaction.

You do that by using the `@slash_option()` decorator and passing the option name as a function parameter:
```python
from interactions import OptionType, slash_option

@slash_command(name="my_command", ...)
@slash_option(
    name="integer_option",
    description="Integer Option",
    required=True,
    opt_type=OptionType.INTEGER
)
async def my_command_function(ctx: SlashContext, integer_option: int):
    await ctx.send(f"You input {integer_option}")
```

!!! danger "Option Names"
    Be aware that, by default, the option `name` and the function parameter need to be the same (in this example both are `integer_option`).

    If you want to use a different name for the function parameter, you can use the `argument_name` parameter in `@slash_option()`.

Options can either be required or not. If an option is not required, make sure to set a default value for them.

Always make sure to define all required options first, this is a Discord requirement!
```python
@slash_command(name="my_command", ...)
@slash_option(
    name="integer_option_1",
    description="Integer Option 1",
    required=True,
    opt_type=OptionType.INTEGER
)
@slash_option(
    name="integer_option_2",
    description="Integer Option 2",
    required=False,
    opt_type=OptionType.INTEGER
)
async def my_command_function(ctx: SlashContext, integer_option_1: int, integer_option_2: int = 5):
    await ctx.send(f"Sum: {integer_option_1 + integer_option_2}")
```

For more information, please visit the API reference [here](/interactions.py/API Reference/API Reference/models/Internal/application_commands/#interactions.models.internal.application_commands.slash_option).

### Restricting Options

If you are using an `OptionType.CHANNEL` option, you can restrict the channel a user can choose by setting `channel_types`:
```python
from interactions import ChannelType, GuildText, OptionType, SlashContext, slash_command, slash_option

@slash_command(name="my_command")
@slash_option(
    name="channel_option",
    description="Channel Option",
    required=True,
    opt_type=OptionType.CHANNEL,
    channel_types=[ChannelType.GUILD_TEXT],
)
async def my_command_function(ctx: SlashContext, channel_option: GuildText):
    await channel_option.send("This is a text channel in a guild")

    await ctx.send("...")
```

You can also set an upper and lower limit for both `OptionType.INTEGER` and `OptionType.NUMBER` by setting `min_value` and `max_value`:
```python
@slash_command(name="my_command", ...)
@slash_option(
    name="integer_option",
    description="Integer Option",
    required=True,
    opt_type=OptionType.INTEGER,
    min_value=10,
    max_value=15
)
async def my_command_function(ctx: SlashContext, integer_option: int):
    await ctx.send(f"You input {integer_option} which is always between 10 and 15")
```

The same can be done with the length of an option when using `OptionType.STRING` by setting `min_length` and `max_length`:
```python
@slash_command(name="my_command", ...)
@slash_option(
    name="string_option",
    description="String Option",
    required=True,
    opt_type=OptionType.STRING,
    min_length=5,
    max_length=10
)
async def my_command_function(ctx: SlashContext, string_option: str):
    await ctx.send(f"You input `{string_option}` which is between 5 and 10 characters long")
```


## Option Choices

If your users ~~are dumb~~ constantly misspell specific strings, it might be wise to set up choices.
With choices, the user can no longer freely input whatever they want, instead, they must choose from a pre-defined list.

To create a choice, simply fill `choices` in `@slash_option()`. An option can have up to 25 choices. The name of a choice is what will be shown in the Discord client of the user, while the value is what the bot will receive in its callback. Both can be the same.
```python
from interactions import SlashCommandChoice

@slash_command(name="my_command", ...)
@slash_option(
    name="integer_option",
    description="Integer Option",
    required=True,
    opt_type=OptionType.INTEGER,
    choices=[
        SlashCommandChoice(name="One", value=1),
        SlashCommandChoice(name="Two", value=2)
    ]
)
async def my_command_function(ctx: SlashContext, integer_option: int):
    await ctx.send(f"You input {integer_option} which is either 1 or 2")
```

For more information, please visit the API reference [here](/interactions.py/API Reference/API Reference/models/Internal/application_commands/#interactions.models.internal.application_commands.SlashCommandChoice).

## Autocomplete / More Than 25 Choices Needed

If you have more than 25 choices the user can choose from, or you want to give a dynamic list of choices depending on what the user is currently typing, then you will need autocomplete options.
The downside is that you need to supply the choices on request, making this a bit more tricky to set up.

To use autocomplete options, set `autocomplete=True` in `@slash_option()`:
```python
@slash_command(name="my_command", ...)
@slash_option(
    name="string_option",
    description="String Option",
    required=True,
    opt_type=OptionType.STRING,
    autocomplete=True
)
async def my_command_function(ctx: SlashContext, string_option: str):
    await ctx.send(f"You input {string_option}")
```

Then you need to register the autocomplete callback, aka the function Discord calls when users fill in the option.

In there, you have three seconds to return whatever choices you want to the user. In this example we will simply return their input with "a", "b" or "c" appended:
```python
from interactions import AutocompleteContext

@my_command_function.autocomplete("string_option")
async def autocomplete(ctx: AutocompleteContext):
    string_option_input = ctx.input_text  # can be empty/None
    # you can use ctx.kwargs.get("name") to get the current state of other options - note they can be empty too

    # make sure you respond within three seconds
    await ctx.send(
        choices=[
            {
                "name": f"{string_option_input}a",
                "value": f"{string_option_input}a",
            },
            {
                "name": f"{string_option_input}b",
                "value": f"{string_option_input}b",
            },
            {
                "name": f"{string_option_input}c",
                "value": f"{string_option_input}c",
            },
        ]
    )
```

???+ note
    Discord does not handle search narrowing for you when using autocomplete. You need to handle that yourself.

## Other Methods of Defining Slash Commands

There are currently four different ways to define slash commands - one does not need any decorators at all.

=== ":one: Multiple Decorators"

    ```python
    @slash_command(name="my_command", description="My first command :)")
    @slash_option(
        name="integer_option",
        description="Integer Option",
        required=True,
        opt_type=OptionType.INTEGER
    )
    async def my_command_function(ctx: SlashContext, integer_option: int):
        await ctx.send(f"You input {integer_option}")
    ```

=== ":two: Single Decorator"

    ```python
    from interactions import SlashCommandOption

    @slash_command(
        name="my_command",
        description="My first command :)",
        options=[
            SlashCommandOption(
                name="integer_option",
                description="Integer Option",
                required=True,
                type=OptionType.INTEGER
            )
        ]
    )
    async def my_command_function(ctx: SlashContext, integer_option: int):
        await ctx.send(f"You input {integer_option}")
    ```

=== ":three: Function Annotations"

    ```python
    from typing import Annotation  # using Annotation is optional, but recommended
    from interactions import slash_int_option

    @slash_command(name="my_command", description="My first command :)")
    async def my_command_function(
        ctx: SlashContext,
        integer_option: Annotation[int, slash_int_option("Integer Option")]
    ):
        await ctx.send(f"You input {integer_option}")
    ```

=== ":four: Manual Registration"

    ```python
    from interactions import SlashCommandOption

    async def my_command_function(ctx: SlashContext, integer_option: int):
        await ctx.send(f"You input {integer_option}")

    bot.add_interaction(
        SlashCommand(
            name="my_command",
            description="My first command :)",
            options=[
                SlashCommandOption(
                    name="integer_option",
                    description="Integer Option",
                    required=True,
                    type=OptionType.INTEGER
                )
            ],
            callback=my_command_function
        )
    )
    ```

## Restrict Commands Using Permissions

It is possible to disable application commands (which include slash commands) for users that do not have a set of permissions.

This functionality works for **permissions**, not to confuse with roles. If you want to restrict some command if the user does not have a certain role, this cannot be done on the bot side. However, it can be done on the Discord server side (in the Server Settings > Integrations page) or through [Checks][checks] as discussed below.

!!!warning Administrators
    Remember that administrators of a Discord server have all permissions and therefore will always see the commands.

    If you do not want admins to be able to overwrite your permissions, or the permissions are not flexible enough for you, you should use [checks][checks].

In this example, we will limit access to the command to members with the `MANAGE_EVENTS` *and* `MANAGE_THREADS` permissions.
There are two ways to define permissions.

=== ":one: Decorators"

    ```python
    from interactions import Permissions, slash_default_member_permission

    @slash_command(name="my_command")
    @slash_default_member_permission(Permissions.MANAGE_EVENTS | Permissions.MANAGE_THREADS)
    async def my_command_function(ctx: SlashContext):
        ...
    ```

=== ":two: Function Definition"

    ```python
    from interactions import Permissions

    @slash_command(
        name="my_command",
        default_member_permissions=Permissions.MANAGE_EVENTS | Permissions.MANAGE_THREADS,
    )
    async def my_command_function(ctx: SlashContext):
        ...
    ```

Multiple permissions are defined with the bitwise OR operator `|`.

### Blocking Commands in DMs

You can also block commands in DMs. To do that, just set `dm_permission` to false.

```py
@slash_command(
    name="my_guild_only_command",
    dm_permission=False,
)
async def my_command_function(ctx: SlashContext):
    ...
```

## Checks

Checks allow you to define who can use your commands however you want.

There are a few pre-made checks for you to use, and you can simply create your own custom checks.

=== ":one: Built-In Check"
    Check that the author is the owner of the bot:

    ```python
    from interactions import check, is_owner

    @slash_command(name="my_command")
    @check(is_owner())
    async def command(ctx: SlashContext):
        await ctx.send("You are the owner of the bot!", ephemeral=True)
    ```

=== ":two: Custom Check"
    Check that the author's username starts with `a`:

    ```python
    from interactions import check

    async def my_check(ctx: BaseContext):
        return ctx.author.username.startswith("a")

    @slash_command(name="my_command")
    @check(my_check)
    async def command(ctx: SlashContext):
        await ctx.send("Your username starts with an 'a'!", ephemeral=True)
    ```

=== ":three: Reusing Checks"
    While you can simply reuse checks by doing `@check(my_check)` for every command you wish to use the check with, you can also reuse them by making your own decorator wrapping the `@check()` decorator:

    ```python
    from interactions import check

    def my_check():
        async def predicate(ctx: ipy.BaseContext):
            return ctx.author.username.startswith("a")

        return check(predicate)

    @slash_command(name="my_command")
    @my_check()
    async def command(ctx: SlashContext):
        await ctx.send("Your username starts with an 'a'!", ephemeral=True)
    ```

When multiple checks are used, *all* checks must pass for the command to be executed.

## Avoid Redefining Options

If you have multiple commands that all use the same option, it might be both annoying and bad programming to redefine it multiple times.

Luckily, if you use the option decorator method, you can simply make your own decorators that themselves call `@slash_option()`:
```python
def my_own_int_option():
    """Call with `@my_own_int_option()`"""

    def wrapper(func):
        return slash_option(
            name="integer_option",
            description="Integer Option",
            opt_type=OptionType.INTEGER,
            required=True
        )(func)

    return wrapper


@slash_command(name="my_command", ...)
@my_own_int_option()
async def my_command_function(ctx: SlashContext, integer_option: int):
    await ctx.send(f"You input {integer_option}")
```

If you use `SlashCommandOption` objects directly, you can also use the same principle:
```python
my_own_int_option = SlashCommandOption(
    name="integer_option",
    description="Integer Option",
    required=True,
    type=OptionType.INTEGER
)

@slash_command(name="my_command", options=[my_own_int_option])
async def my_command_function(ctx: SlashContext, integer_option: int):
    await ctx.send(f"You input {integer_option}")
```

The same principle can be used to reuse autocomplete options.

## Error Handling

By default, if an error occurs in a command, interactions.py will send the error to its default error listener, which will either:

- Send an appropriate error message to the user, if the error is meant to be shown to the user (IE cooldown or check errors).
- Log the error and, if `send_command_tracebacks` is enabled in your Client (which it is by default), send the error as a response to the command.

To override this for a single command, you can use the `@error` decorator every command has:

```python
@slash_command(name="my_command", ...)
async def my_command_function(ctx: SlashContext):
    raise ValueError("Something went wrong!")

@my_command_function.error
async def on_command_error(error: Exception, ctx: SlashContext):
    await ctx.send(f"Something went wrong: {error}")
```

???+ note
    If you wish for an error handler for a group of commands, you may wish to check out [Extensions](20 Extensions.md), which allows you both to group commands together and add error handlers to the group.

If you want error handling for all commands, you can override the default error listener and define your own.
Any error from any command will trigger `CommandError` - note that this includes errors from context menus and (if enabled) prefixed commands.

In this example, we are logging the error and responding to the interaction if not done so yet:
```python
import traceback
from interactions.api.events import CommandError

@listen(CommandError, disable_default_listeners=True)
async def on_command_error(event: CommandError):
    traceback.print_exception(event.error)
    if not event.ctx.responded:
        await event.ctx.send("Something went wrong.")
```

!!! warning
    If you override the default error listener, you will need to handle all errors yourself. This *includes* errors typically shown to the user, such as cooldowns and check errors.

There also is `CommandCompletion` which you can listen into too. That fires on every interactions usage.

## Custom Parameter Type

If your bot is complex enough, you might find yourself wanting to use custom models in your commands.

To do this, you'll want to use a string option, and define a converter. Information on how to use converters can be found [on the converter page](../08 Converters).

## Hybrid Commands

!!! note
    Prefixed commands, called by Discord as "text commands" and sometimes called "message commands" (not to be confused with Context Menu Message Commands), are commands that are triggered when a user sends a normal message with a designated "prefix" in front of them (ie `!my_command`).

    interactions.py contains an extension for making these commands, which you can [read about here](/interactions.py/Guides/26 Prefixed Commands).

Hybrid commands are are slash commands that also get converted to an equivalent prefixed command under the hood. They are their own extension, and require [prefixed commands to be set up beforehand](/interactions.py/Guides/26 Prefixed Commands). After that, use the `setup` function in the `hybrid_commands` extension in your main bot file.

Your setup should look similar to this:

```python
import interactions
from interactions.ext import prefixed_commands as prefixed
from interactions.ext import hybrid_commands as hybrid

bot = interactions.Client(...)  # may want to enable the message content intent
prefixed.setup(bot)  # normal step for prefixed commands
hybrid.setup(bot)  # note its usage AFTER prefixed commands have been set up
```

To actually make slash commands, simply replace `@slash_command` with `@hybrid_slash_command`, and `SlashContext` with `HybridContext`, like so:

```python
from interactions.ext.hybrid_commands import hybrid_slash_command, HybridContext

@hybrid_slash_command(name="my_command", description="My hybrid command!")
async def my_command_function(ctx: HybridContext):
    await ctx.send("Hello World")
```

Suggesting you are using the default mention settings for your bot, you should be able to run this command by typing out `@BotPing my_command` or using the slash command `/my_command`. Both will work largely equivalently.

As you can see, the only difference between hybrid commands and slash commands, from a developer perspective, is that they use `HybridContext`, which attempts
to seamlessly allow using the same context for slash and prefixed commands. You can always get the underlying context via `inner_context`, though.

Of course, keep in mind that supporting two different types of commands is hard - some features may not get represented well in prefixed commands, and autocomplete is not possible at all.
