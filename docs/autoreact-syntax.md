# Syntax for `/config` autoreacts.

Autoreacts, as specified in the `/config` command, use a special syntax of `{{helpers}}` based on the templating language [Handlebars](https://handlebarsjs.com/) to allow advanced specification of which emojis to react to, and with that. This page exists to explain that syntax.

This syntax is used because it's easy to implement and we will rarely need to change our autoreact settings, compared to the difficulty of making an interactive interface for it under Discord's limitations.

The basic gist of this is, you use the `{{builtin-emoji "name"}}`, `{{guild-emoji "name"}}`, and `{{bot-emoji "name"}}` helpers to specify emojis to react with. Here's the simplest possible example:

```hbs
{{builtin-emoji "heart"}}
```

This will cause the bot to react to every new message, everywhere with a heart. You can add a more reactions as well. This will react to every message with a heart and star:

```hbs
{{builtin-emoji "heart"}}
{{builtin-emoji "star"}}
```

But reacting unconditionally isn't very useful, so you may filter the reactions based on Handlebars' `{{#if (cond)}}{/if}` syntax:

```hbs
{{#if (is-channel "chat")}}
  {{builtin-emoji "heart"}}
{{/if}}
```

`{{#if (cond)}}` is the start of an *if block*, where the content inside the block will only be evaluated if the condition passes. In this case, it checks that the channel name is `"chat"`, and only reacts with the emojis specified inside the block if that's the case. The `{{/if}}` indicates the end of the block.

Note that the indentation of the content inside the if block is optional, but aids readability.

You can have multiple independent if blocks:

```hbs
{{#if (is-channel "chat")}}
  {{builtin-emoji "heart"}}
{{/if}}

{{#if (is-channel "worm")}}
  {{builtin-emoji "star"}}
{{/if}}
```

You can create more complex conditions with the `any` and `all` helpers.

```hbs
{{#if (any (is-channel "chat") (is-channel "nsfw-chat"))}}
  {{builtin-emoji "heart"}}
{{/if}}

{{#if (all (is-channel "tipsy-turvy-treatise") (is-user "tipsypastels"))}}
  {{builtin-emoji "duck"}}
{{/if}}
```

In the former case, the bot will react with heart if `any` of these conditions are true: the channel name is `"chat"`, or the channel name is `"nsfw-chat"`. 

The latter will react with duck if `all` of the conditions are true: the channel name is `"tipsy-turvy-treatise"` and the message author's username is `"tipsypastels"`.

# List of helpers

## Emoji helpers

Helpers for specifying which emojis to react with.

### `{{builtin-emoji "name"}}`

Specifies a Discord built-in emoji by name. All applicable messages (all of them, if this is not inside an if block, otherwise all messages that match the if block's condition) will be reacted to with that emoji.

**Example:** `{{builtin-emoji "heart"}}`

### `{{guild-emoji "name"}}`

Specifies an Earth Vav server emoji by name. Otherwise behaves as above.

**Example:** `{{guild-emoji "tattletrue"}}`

### `{{bot-emoji "name"}}`

Specifies a bot emoji by name. This is only for emojis that have been uploaded to the bot directly.

**Example:** `{{bot-emoji "ziz_wing_left"}}`

## If-block helpers

Helpers that allow specifying the condition of an if block.

### `(is-channel "name")`

When used in an if block, ensures the emojis are only reacted with if the channel name matches the name provided.

**Example:** 
```hbs
{{#if (is-channel "chat")}} 
  {{bot-emoji "ziz"}}
{{/if}}
```

### `(is-channel-id "id")`

As above, but compares channel IDs instead.

**Example:** 
```hbs
{{#if (is-channel-id "1115463698425847808")}} 
  {{bot-emoji "ziz"}} 
{{/if}}
```

### `(is-user "username")`

When used in an if block, ensures the emojis are only reacted with if the author's username matches the username provided.

**Example:** 
```hbs
{{#if (is-user "tipsypastels")}} 
  {{bot-emoji "ziz"}} 
{{/if}}
```

### `(is-user-id "username")`

As above, but compares user IDs instead.

**Example:** 
```hbs
{{#if (is-user-id "147564239941402625")}} 
  {{bot-emoji "ziz"}} 
{{/if}}
```

### `(contains "text")`

When used in an if block, ensures the emojis are only reacted with if the message content contains the provided text.

**Example:** 
```hbs
{{#if (contains "angler")}} 
  {{builtin-emoji "fish"}} 
{{/if}}
```

### `(all (cond1) (cond2) (...))`

When used in an if block, requires that all of its child conditions pass in order to pass itself.

**Example:**
```hbs
{{#if (all (is-channel "tipsy-turvy-treatise") (is-user "tipsypastels"))}}
  {{builtin-emoji "duck"}}
{{/if}}
```

### `(any (cond1) (cond2) (...))`

As above, but only requires at least one of its child conditions to pass in order to pass itself.

**Example:**
```hbs
{{#if (any (is-channel "chat") (is-channel "nsfw-chat"))}}
  {{builtin-emoji "heart"}}
{{/if}}
```

### `(not (cond))`

Inverts its child condition.

**Example:**
```hbs
{{#if (not (is-channel "chat"))}}
  {{builtin-emoji "heart"}}
{{/if}}
```