# Syntax for `/document`s.

Posts created with the `/document`-family of commands have a special syntax of `{{helpers}}` based on the templating language [Handlebars](https://handlebarsjs.com/). This page exists to explain that syntax.

Some of the `{{helpers}}` exist because of the problem that Discord's popup modal text inputs are always **plain text**. Although typing `**bold**` into the text input will produce bold in the output, that will not be previewed or autocompleted in any way. This is not a big problem for bold, but it is a problem for things like mentions, which require you to type in their internal syntax, e.g. `<@147564239941402625>` instead of `@tipsypastels`. To do that you will have to close and reopen the modal multiple times to copy IDs, potentially losing progress. Hence, some helpers exist to add a different syntax that bypasses the need to copy IDs.

Other helpers, referred to as flags, do not translate into rendered text in the output but instead set various options about the output. For example, the `{{accent}}` helper sets the accent colour of the resulting container.

# List of helpers

## Rendering helpers

Helpers that directly transform into text in the rendered output.

### `{{channel "name"}}`

Becomes a "mention" of a channel, without needing to copy its ID.

**Example:** `{{channel "meta"}}`.

### `{{role "name"}}`

Becomes a mention of a role, without needing to copy its ID.

**Example:** `{{role "Mods"}}`.

### `{{ziz}}`

Becomes a mention of **The Simurgh**.

**Example:** `{{ziz}}`.

### `{{botemoji "name"}}`

Becomes a bot-specific emoji, without needing to copy its ID. This is only for emojis that have been uploaded to Ziz directly, you may want `{{guildemoji}}` instead.

**Example:** `{{botemoji "zizwing"}}`.

### `{{guildemoji "name"}}`

Becomes an emoji from Vav, without needing to copy its ID.

**Example:** `{{guildemoji "tattletrue"}}`.

### `{{guildicon}}`

Becomes an emoji of the server icon. This is the same in practice as `{{guildemoji "earthvav"}}` since we already have a server icon emoji on Vav, but `{{guildicon}}` would work even if that emoji didn't exist.

**Example:** `{{guildicon}}`.

### `{{li digit|"letter"}}`

Becomes stylized virtual emoji of a digit or letter, to be used in lists. The input must be that same digit or letter. If the input is a number outside of `0-9` or a string outside of `a-z`, an error is thrown.

**Examples:** `{{li 1}}`, `{{li "a"}}`

## Section Flags

Helpers that affect the behaviour of the current **section**. A section is a portion of the input text separated by `---`, which translates into a visible divider.

### `{{thumbnail "url"}}`.

Causes the section to have the provided image url as its thumbnail. The thumbnail is a large image that appears in the top right of the section. Mutually incompatible with `{{guildthumbnail}}`.

**Example:** `{{thumbnail "https://raw.githubusercontent.com/tipsypastels/vav-assets/refs/heads/main/ziz.png"}}`.

### `{{guildthumbnail}}`

Causes the section to have the server icon as its thumbnail. Mutually incompatible with `{{thumbnail}}`.

**Example:** `{{guildthumbnail}}`.

### `{{nextseparator "hidden"|"skip"}}`

Changes the behaviour of the dividing line separating this section and the next section. By default, a visible line will be shown.

- If the provided parameter is `"hidden"`, the line will be invisible, but the content will still be spaced as though there was a dividing line.
- If the provided parameter is `"skip"`, the line and spacing between sections will both be skipped, and the spacing will instead be half that of a standard paragraph gap.
- If the provided parameter is any other value, an error will be thrown.

This command has no effect if there is no next section.

**Examples:** `{{nextseparator "hidden"}}`, `{{nextseparator "skip"}}`.

## Global Flags

Helpers that affect the behaviour of the final output in some other way.

### `{{accent "colour"}}`

Sets the accent colour of the output. The accent colour of a container is the bar of colour running along the left side. The value for colour may be a hex code, e.g. `#11806a`, or one of Ziz's built-in colour names: `info`, `ok`, `warning`, `error`, `mod`. For the default Vavvers colour used for most Ziz messages, use `info`.

**Examples:** `{{accent "info"}}`, `{{accent "#11806a"}}`.