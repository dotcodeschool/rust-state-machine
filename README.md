# Rust Tooling

In this step, we will initialize a basic rust project, where we can start building our simple Rust state machine.

## rustfmt

To keep your code clean and easy to read, we use a tool called [`rustfmt`](https://github.com/rust-lang/rustfmt). To access all the latest features of `rustfmt` we specifically use the `nightly` toolchain.

To install `rustfmt` for `nightly`:

```bash
rustup component add rustfmt --toolchain nightly
```

To configure the behavior of `rustfmt`, we will create a `rustfmt.toml` file:

1. Create a new file in your project's root directory called `rustfmt.toml`.

	```bash
	touch rustfmt.toml
	```
2. Use the following `rustfmt.toml` file to configure your formatting preferences.

```toml filename="rustfmt.toml"
# Basic
edition = "2021"
hard_tabs = true
max_width = 100
use_small_heuristics = "Max"
# Imports
imports_granularity = "Crate"
reorder_imports = true
# Consistency
newline_style = "Unix"
# Misc
chain_width = 80
spaces_around_ranges = false
binop_separator = "Back"
reorder_impl_items = false
match_arm_leading_pipes = "Preserve"
match_arm_blocks = false
match_block_trailing_comma = true
trailing_comma = "Vertical"
trailing_semicolon = false
use_field_init_shorthand = true
# Format comments
comment_width = 100
wrap_comments = true
```

3. Run the code formatter using the following command:

	```bash
	cargo +nightly fmt
	```
	
	But since we already cloned the repository in a folder named `dotcodeschool-rust-state-machine` during the setup process, we will use that.

You shouldn't see any changes this time around, but as you write more code, you will be able to see `cargo +nightly fmt` make everything look pretty, consistent, and easy to read.

> We recommend you run `cargo +nightly fmt` after every step!

## Rust Analyzer

Another popular tool in the Rust community is [Rust Analyzer](https://rust-analyzer.github.io/).

It provides many features like code completion and goto definition for code editors like VS Code.

However, to provide the full functionality that it does, Rust Analyzer needs to compile your code. For a small project like this one, this is not a problem, however working with a large project like Substrate / Polkadot-SDK, it is.

It is my personal recommendation that Rust Analyzer is not needed in this workshop, and generally you should not use it for Substrate development. However, this section might be updated in the future to include special configurations of Rust Analyzer which will work well with Polkadot SDK in the future.

However, if you would like to use it anyway, now is the right time to set it up.
