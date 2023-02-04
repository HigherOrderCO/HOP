Higher Order Parser
===================

HOP is a lightweight higher-order parser in Rust.

Hi-Parser provides a more Haskell-like parser style, and explores the `?` syntax
of `Result` to have a do-block-like monadic notation. This isn't as good as a
proper do notation, but avoids identation hell. For example, a lambda, in a
functional language, can be parsed as:

```rust
// Parses a λ-term in the syntax: `λx => body`
pub fn parse_lam(state: parser::State) -> parser::Answer<Option<Box<Term>>> {
  let (state, _)    = parser::there_take_exact("λ")?;
  let (state, name) = parser::there_nonempty_name(state)?;
  let (state, _)    = parser::there_take_exact("=>")?;
  let (state, body) = parse_term(state)?;
  Ok((state, Box::new(Term::Lam { name, body })))
}
```

A Parser is defined simply as:

```rust
Answer<A> = Result<(State, A), String>
Parser<A> = Fn(State) -> Answer<A>>
```

That is, a Parser is a function that receives a `State` and returns an `Answer`.
That answer is either an updated state, and the parse result (if it succeeds),
or an error message, as a simple string, if it fails. Note that there are two
ways to fail:

### 1. Recoverable. Return an `Ok` with a `Bool`, or an `Option`:

```rust
- Ok((new_state, Some(result))) if it succeeds
- Ok((old_state, None))         if it fails
```

This backtracks, and can be used to implement alternatives. For example, if
you're parsing an AST, "Animal", with 2 constructors, dog and cat, then you
could implement:

```rust
parse_dog    : Parser<Option<Animal>>
parse_cat    : Parser<Option<Animal>>
parse_animal : Parser<Animal>
```

### 2. Irrecoverable. Return an `Err` with a message:

```rust
- Err(error_message)
```

This will abort the entire parser, like a "throw", and return the error message.
Use this when you know that only one parsing branch can reach this location, yet
the source is wrong.

---

This parser is used by [HVM](https://github.com/HigherOrderCO/hvm).

Note: this parser is in very early stage and provides very few features.

TODO: add examples, improve documentation.
