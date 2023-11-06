# Verusfmt

An opinionated formatter for [Verus] code.

## WARNING

`verusfmt` is highly experimental code. Make backups of your files before trying
`verusfmt` on them.

## Goals

1. Make it easier to read and contribute to [Verus] code by automatically
   formatting it in a consistent style (added bonus: eliminating soul-crushing
   arguments about style).
2. Produce acceptably "pretty" output.
3. Run fast!  `verusfmt` may be run in pre-submit scripts, CI, etc., so it can't
   be slow.
4. Keep the code reasonably simple. Pretty printers are [notoriously
   hard](https://journal.stuffwithstuff.com/2015/09/08/the-hardest-program-ive-ever-written/),
   so we try to take steps to reduce that difficulty, so that `verusfmt` can be
   updated and adapted with a reasonable amount of effort. 

### FAQ

1. Why not adapt [`rustfmt`] for [Verus] idioms?

 **TODO**: jayb

1. Why not build this as a feature of [Verus]?

By the time Verus receives an AST from `rustc`, we've already lost information
about whitespace and comments, meaning that we couldn't preserve the comments
in the reformatted code.  Plus, firing up all of `rustc` just to format some
code seems heavyweight.

## Future Work
- Special handling for commonly used macros, like `println!`, `state_machine!`
- In `record_expr_field_list` and `record_pat_field_list`, handle vanishing comma's interaction with `rest_pat`
- Enforce the [Rust naming policy](https://doc.rust-lang.org/beta/style-guide/advice.html#names)? 

## Non-Future Work
- We currently have no plans to sort `use` declarations the way [`rustfmt`] does
- We do not intend to parse code that [Verus] itself cannot parse.  Sometimes `verusfmt` 
  happens to parse such code, but that is unintentional and cannot be counted upon.
- Perfectly match the formatting produced by [`rustfmt`]
- Handle comments perfectly -- they're surprisingly hard!

## Design Overview

Our design is heavily influenced by the [Goals](#Goals) above.  Rather than
write everything from scratch ([a notoriously hard
undertaking](https://journal.stuffwithstuff.com/2015/09/08/the-hardest-program-ive-ever-written/)),
we use a parser-generator to read in Verus source code, and a pretty-printing
library to format it on the way out.  We try to keep each phase as performant
as possible, and we largely try to keep the formatter stateless, for
performance reasons but more importantly to try to keep the code reasonably
simple and easy to reason about.  Hence we sometimes deviate from Rust's style
guidelines for the sake of simplicity.

### Parsing

We define the syntax of Verus source code using [this
grammar](src/verus.pest), which is processed by the [Pest](https://pest.rs/)
parser generator, which relies on Parsing Expression Grammars
([PEG](https://en.wikipedia.org/wiki/Parsing_expression_grammar)s).  It
conveniently allows us to define our notion of whitespace and comments, which
the remaining rules can then ignore; Pest will handle them implicitly.  We
explicitly ignore the code outside the `verus!` macro, leaving it to
[`rustfmt`].  We prefer using explicit rules for string constants, since it
allows a more uniform style when formatting the code.  In some places, we have
multiple definitions for the same Verus construct, so that we can format it
differently depending on the context (see, e.g., `attr_core`).  Many of the
rules are designed to follow the corresponding description in [The Rust
Reference](https://doc.rust-lang.org/beta/reference/introduction.html).

### Formatting

[pretty](https://crates.io/crates/pretty) crate, based on [Philip Wadler's](https://homepages.inf.ed.ac.uk/wadler/papers/prettier/prettier.pdf) design.

**TODO**

## Contributing

We welcome contributions! Please read on for details.

We consider it a bug in `verusfmt` if you provide `verusfmt` with code
that [Verus] accepts and `verusfmt` produces code that Verus does not accept
or code that has different semantics from the original.  When this happens,
please open a GitHub issue with a minimal example of the offending code
before and after formatting.

If `verusfmt` produces valid code but you dislike the formatting, please open
a GitHub pull request with your proposed changes and rationale for those changes.
Please ensure that existing test cases still pass (see below for more details),
unless your goal is to change how some of those test cases are handled.  Please
also include new/updated tests that exercise your proposed changes.


## Testing

### Rust-like formatting

In general, we try to adhere to Rust's style guide.  Tests for such adherence live in
[tests/rustfmt-tests.rs](tests/rustfmt-tests.rs).  These tests will compare the output
of [`rustfmt`] to that of `verusfmt`.  You can run
them via `cargo test`.

### Verus-like formatting

In various places, we deviate from Rust's style, either to simplify the formatter
or to handle [Verus]-specific syntax.  Tests for formatting such code live in
[tests/snap-tests.rs](tests/snap-tests.rs).  You can add a new test or modify an
existing one by writing/changing the input code.  The test's correct answer is
maintained via the [Insta testing framework](https://insta.rs).

Insta recommends installing the `cargo-insta` tool for an improved review experience:
```
cargo install cargo-insta
```

You can run the tests normally with `cargo test`, but it's often more convenient
to run the tests and review the results via:
```
cargo insta test
cargo insta review
```
or more succinctly:
```
cargo insta test --review
```


[Verus]: https://github.com/verus-lang/verus
[`rustfmt`]: https://github.com/rust-lang/rustfmt
