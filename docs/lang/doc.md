# Document language syntax

Some Ziz commands that allow the user to enter custom longform text via a modal (popup) use a custom syntax I've called, uncreatively, "document language" (or `doclang`). This language exists because Discord modals only support **plain text** inputs. Although typing `**bold**` in the text will produce bold in the output, that will not be previewed or autocompleted in any way. This is not a big problem for bold, but it is a problem for things like mentions, which require you to type in their internal syntax, e.g. `<#1115463698425847808>` instead of `#chat`. To do that you will have to close and reopen the modal multiple times to copy IDs, potentially losing progress. Hence, document language began as a way to preprocess modal text inputs to make them more convenient to use.

In addition, some places where the language is used (like the `/document` commands) support additional syntax used to customize the appearance and behaviour of the output, like changing the accent colour of the container or attaching autorole or modmail-triggering buttons. These features are called "blocks mode" internally, a superset of the simpler default "text mode".

Document language is currently used in three places:

- The `/document` family of commands (in blocks mode).
- The `/config tab:help` fields for "other features" of Ziz (in text mode).
- The `/infraction create` fields for "reason" and "moderator note" (in text mode).

# How it works

## Text mode syntax

To restate the opening, Discord markdown features **still work** in the document language. Bold and italics and lists work, even regular mentions and emojis work if you use their internal syntax. The document language supplements these with additional syntax that make the lack of preview or autocomplete in modals less painful.

### Named Mentions

Normally, when you mention something in a Discord text input, the client transparently swaps out a mention like `#chat` into `<#1115463698425847808>` behind the scenes. As mentioned in the opening, modals as plain text inputs do not do this, so `doclang` does it instead.

- The syntax `#channel-name` automatically becomes a mention of that channel.
- If the channel name has a space in it, such as a thread name, it must be enclosed in angle brackets, e.g. `<#STAR WARS>`.

Likewise, the same is done for roles. The syntax for roles is `@&Name`, similar to Discord's own internal syntax for it.

- `@&Mods` becomes a mention of the mods role.
- Once again, angle brackets are required if there are spaces, e.g. `<@&Fact Checkers>`.

Lastly, there is a shorthand for mentioning Ziz:

- `@ziz` becomes a mention of The Simurgh.

There is no syntax for mentioning a specific (non-Ziz) user by name because looking up an arbitrary user by name is expensive. I may add this if needed.

### Sourced Emojis

Much like mentions, Discord hides the actual syntax of emojis for you, except in modals where it provides no help at all. `doclang` provides its own syntax for emojis, with the requirement that you must explicitly specify the *emoji source*.

An emoji source can be one of
  - `builtin`, e.g. Discord provides this emoji.
  - `guild`, e.g. this emoji is from the current server.
  - `bot`, e.g. this emoji comes with Ziz.

The syntax for a "sourced emoji", then, is `:source:name:`.

  - `:builtin:heart:` will become the Unicode heart emoji.
  - `:guild:ratty:` will become Vav's ratty emoji.
  - `:bot:info:` will become Ziz's commonly used info icon.

### Guild Emoji

A special case.

`:guild:` becomes a generated emoji of the server icon.

### Stylized List Items

As seen in the rules, Ziz has (or rather, can generate) stylized emojis for the 0 to 9 and a-z characters. The syntax for this is `(_.)`, where `_` is the number or letter.

- `(1.)` becomes a stylized number.
- `(a.)` becomes a stylized letter.

## Blocks mode synax

Blocks mode is a superset of text mode, intended for cases where the output is an entire message (like for the `/document` commands), rather than a piece of an existing message (like `/help`). In blocks mode, additional features exist to customize the message.

An up-front error will be issued if blocks mode features are used in text mode.

### Dividers

Blocks mode is called blocks mode because the text is separated into "blocks" by dividers.

A divider is written as `---`, and must be entirely on a line of its own. It creates a divider in the output, and causes the text after it to be part of the next block.

There are two other subtypes that have minor differences.

- `---?` creates a divider but without a visible dividing line.
- `---!` does not create a divider in the output, but still causes everything after it to be part of the next "block".

### Command lines

Command lines are blocks mode features that allow tweaking the generated message, but do not themselves output text.

A command line is written as `$ command-name`. Some may require parameters, written after the name and separated by commas. It must be entirely on a line of its own.

#### `$ accent-colour` (or `$ accent-color`)

Sets the accent colour of the output. The accent colour of a container is the bar of colour running along the left side.

This command takes a parameter that must be one of `info`, `ok`, `warning`, or `error`. These map to Ziz's builtin accent colours. Custom values are not supported.

For example, `$ accent-colour info` sets the accent colour to the one Ziz uses for most messages, which is the same as Vavvers grey.

#### `$ thumbnail`

Sets the thumbnail of the current block. Requires one parameter which may be exactly `guild` to use the server icon, or a URL delimited by double quotes.

For example, `$ thumbnail guild` or `$ thumbnail "https://i.imgur.com/G1lSVZ9.jpeg"`.

#### `$ no-container`

Causes the output to appear without a container (the visual block enclosing the text). This also causes the `$ accent-colour` and `$ thumbnail` commands to do nothing.

#### `$ autorole-button`

Creates a button that assigns a role. It takes one parameter, which uses the same "named role mention" syntax described in text mode, and several more optional keyword parameters.

The simplest use is as follows:

`$ autorole-button @&Vavvers`

This creates a button that assigns the Vavvers role. The button will always be placed at the end of the current block. Multiple buttons can be added to the same block, and will become one or more rows.

There are additional options for customizing the button. These are **keyword parameters**, meaning you need to write them as `keyword=value` rather than just `value`, for clarity of use. They can be passed in any order.

`label=` sets the text of the button. If not provided it will be the role name. The parameter value must be text enclosed in double quotes. For example, `label="click me!"`

`emoji=` sets the emoji of the button. The parameter value can be a "sourced emoji" using the same syntax described in text mode, e.g. `:builtin:heart:`, or it can be `:guild:` for the server icon, `:icon:` for the role icon, or `:colour:`/`:color:` for the role colour. For examples, `emoji=:builtin:heart:`, `emoji=:guild:`.

`reversible=` sets whether the role is reversible (interacting again to remove it). The parameter value must be `true` or `false`, for example, `reversible=true`. Autoroles default to not reversible.

`external=` changes the button placement if set to `true`. The parameter value must be `true` or `false`, for example, `external=true`. When enabled, external buttons will not appear at the end of the current block, but will instead be outside and beneath the entire container.

Here is an example of all options used at once. Note the mandatory commas between parameters.

`$ autorole-button <@&Fact Checkers>, emoji=:builtin:nerd:, label="I know things!", reversible=true, external=true`

#### `$ modmail-button`

Creates a button that opens a modmail when clicked. Requires one keyword parameter `label=` and supports other optional ones.

The simplest use is as follows:

`$ modmail-button label="Need help? Click!"`

There are additional options for customizing the button.

`emoji=` sets the emoji of the button. This works identically to the equivalent option for autorole buttons, except `:icon:` and `:colour`/`:color:` aren't supported as there is no role.

`external=` sets whether the button should appear outside and below the container. This works identically to the equivalent option for autorole buttons.

Here is an example of all options used at once. Note the mandatory commas between parameters.

`$ modmail-button label="Create Modmail", emoji=:bot:config_add:, external=true`