# Conditional language syntax

Some parts of Ziz's `/config` command use a custom syntax I've called, uncreatively, "conditional language" (or `condlang`). Conditional language is used to determine *if* Ziz should do something: for example, if it should react to a message. Because `/config` doesn't change very often, and these conditions can be complex, Ziz using conditional language is significiantly simpler than trying to make something more interactive and user friendly.

Conditional language is currently used in two places:

- Autoreacts (should I react to a message with this emoji?)
- Link checks (should I report that this link is a violation?)

You can see it for yourself if you open up `/config` with `tab:autoreacts` or `tab:link checks`.

# How it works

> [!NOTE]
> If you have experience with C-descended programming languages, you might skip to "List of predicates". Just know that `condlang`'s syntax is based on the same logical operators, e.g. `(true && false) || !thing && is-foo("bar")` and such. The differences from C-like are that identifiers use `dash-case`, and grouping is right associative with no preference between `||` and `&&`.

## Logical operators

The conditional syntax is based, on, well, conditions. And ways of combining them. Here is the simplest valid condition.

```perl6
true
```

`true` is a condition that always returns yes! And I imagine you can guess what `false` does. These two unit values are called "booleans", and what we're doing here is called "boolean algebra" - combining some arbitrarily complex conditions to return a true or false value.

The first way to combine conditions is with an "or" expression, written as `||`.

```perl6
false || true
```

An or expression results in `true` if *either* the sub-expression on the left or the one on the right also result in `true`. The answer in this case, of course, is `true`.

Next we have an "and" expression, written as `&&`.

```perl6
true && false
```

This one requires that the sub-expressions its left and the right are both `true` in order to be `true` itself. In this case, it's false.

The last operator is "not", written as `!`.

```perl6
!true
```

This one goes *before* an expression instead of between two expressions, and it simply negates it. `true` becomes `false` and vice versa.

## Grouping

Like in high school math, you may group conditions with parentheses.

```perl6
(true || false) && true
```

Why would you do this? Well, you might need to if the default way expressions are grouped isn't what you want. For example, this...

```perl6
true || true && false
```

...is ambiguous to the eye. Is it read as `(true || true) && false`, which would be `false`, or as `true || (true && false)`, which would be `true`?

The answer, if you don't use parentheses, is the latter. `condlang` is *right associative*, which means that if not explicitly stated, groups are inferred from right to left.

If you're writing anything where the grouping might be non-obvious, I recommend parentheses.

## Predicates

So far, we've only used *constants* in our examples; `true` and `false` never change their meaning in any context. This is for simplicity of explanation but it's also not useful, since it can't depend on the input at all. And making decisions based on input is the whole point of `condlang`!

For this, the language has "predicates". A predicate is a function that allows us to check something about the input. Predicates may take operands, and give back booleans that we can use in our boolean algebra.

**There is no universal list of predicates**, which makes explaining this a little tricky. This is because `condlang` is generic and can be used for any conditions, which may provide different predicates. For example, when used for link checks, a `query` predicate is provided that checks if something (like a banned tag) exists on the page. When used for autoreacts, there is no `query` predicate, because that wouldn't make sense, no web page is being loaded.

For now, let's imagine we're using the language in such a way that we have these predicates:

- `am-i-a-duck` tells you if you're a duck.
- `is-foo(string)` takes an operand, and tells you if that if that operand is `"foo"`.

Predicates can be used in all the same boolean algebra ways we already discussed:

```perl6
true || (am-i-a-duck && false)
```

If a predicate needs operands, like `is-foo` does, write them after the name, delimited by parentheses and separated by commas. Right now, the only supported operand type is the *string literal*, i.e. some text enclosed by double quotes.

```perl6
(is-foo("bar") || is-foo("foo")) && !am-i-a-duck
```

And that's pretty much it!

## List of predicates

### When used for autoreacts

- `is-channel(string)` checks if the channel *name* of the message is the same as the operand, e.g. `is-channel("memes")`.
- `is-channel-id(string)` is likewise, but uses the channel ID, e.g. `is-channel-id("1115463698425847808")`.
- `is-user(string)` checks if the username of the message author is the same as the operand.
- `is-user-id(string)` again, same but with user IDs.
- `contains(string)` checks if the message content contains the provided string, e.g. `contains("quack")`.

### When used for link checks

- `query(string)` searches for a certain node on the page using [CSS selector syntax](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Selectors). Returns whether the node exists.
- `is-moderator` returns whether the message author is a mod.
- `is-nsfw-channel` returns if the channel is NSFW.

### Examples

At the time of this writing, the autoreact and link check conditions are:

**React with the :ziz: emote in #welcome:**

```perl6
is-channel("welcome")
```

**React with the :ziz_read: emote in #updates and #nsfw-updates, but ignore that Greg poster:**

```perl6
(is-channel("updates") || is-channel("nsfw-updates")) && !is-user("marethyuthekursed")
```

**Link check violation on AO3 links with a certain banned tag, unless it's linked by a mod:**

```perl6
query("a[href='/tags/Underage%20Sex/works']") && !is-moderator
```

**Also link check violation on NSFW fics linked in SFW channels, again unless a mod:**

```perl6
!is-nsfw-channel && query("a[href='/tags/Explicit/works']") && !is-moderator
```