# Syntax for `/config` autoreacts.

Autoreacts, as specified in the `/config` command, use a special syntax of `{{helpers}}` based on the templating language [Handlebars](https://handlebarsjs.com/) to allow advanced specification of which emojis to react to, and with that. This page exists to explain that syntax.

This syntax is used because it's easy to implement and we will rarely need to change our autoreact settings, compared to the difficulty of making an interactive interface for it under Discord's limitations.

The basic gist of this is, you use the `{{builtinemoji "name"}}`, `{{guildemoji "name"}}`, and `{{botemoji "name"}}` helpers to specify emojis to react with. Here's the simplest possible example:

```hbs
{{builtinemoji "heart"}}
```

This will cause the bot to react to every new message, everywhere with a heart. You can add a more reactions as well. This will react to every message with a heart and star:

```hbs
{{builtinemoji "heart"}}
{{builtinemoji "star"}}
```

But reacting unconditionally isn't very useful, so you may filter the reactions based on Handlebars' `{{#if (cond)}}{/if}` syntax:

```hbs
{{#if (eq channel.name "chat")}}
  {{builtinemoji "heart"}}
{{/if}}
```

`{{#if (cond)}}` is the start of an *if block*, where the content inside the block will only be evaluated if the condition passes. In this case, it checks that the channel name is `"chat"`, and only reacts with the emojis specified inside the block if that's the case. The `{{/if}}` indicates the end of the block.

Note that the indentation of the content inside the if block is optional, but aids readability.

You can have multiple independent if blocks:

```hbs
{{#if (eq channel.name "chat")}}
  {{builtinemoji "heart"}}
{{/if}}

{{#if (eq channel.name "worm")}}
  {{builtinemoji "star"}}
{{/if}}
```

You can create more complex conditions with the `any` and `all` helpers.

```hbs
{{#if (any (eq channel.name "chat") (eq channel.name "nsfw-chat"))}}
  {{builtinemoji "heart"}}
{{/if}}

{{#if (all (eq channel.name "tipsy-turvy-treatise") (eq author.username "tipsypastels"))}}
  {{builtinemoji "duck"}}
{{/if}}
```

In the former case, the bot will react with heart if `any` of these conditions are true: the channel name is `"chat"`, or the channel name is `"nsfw-chat"`. 

The latter will react with duck if `all` of the conditions are true: the channel name is `"tipsy-turvy-treatise"` and the message author's username is `"tipsypastels"`.

## List of helpers

TODO