# Initialize your Rust Project

In this step, we will initialize a basic rust project, where we can start building our simple Rust state machine.

## cargo init

1. Typically, you would create a directory where you want your project to live, and navigate to that folder like so:

	```
	mkdir dotcodeschool-rust-state-machine
	cd dotcodeschool-rust-state-machine
	```
	
	But since we already cloned the repository in a folder named `dotcodeschool-rust-state-machine` during the setup process, we will use that.

2. In that folder, initialize your rust project using `cargo init`:

	```
	cargo init
	```

	This will scaffold a basic Rust executable which we can use to start building.

3. You can verify that your new project is working as expected by running:

	```
	cargo run
	```

	You should see "Hello, World!" appear at the end of the compilation:

	```
	➜  rust-state-machine git:(master) ✗ cargo run
	Compiling rust-state-machine v0.1.0 (/Users/shawntabrizi/Documents/GitHub/rust-state-machine)
		Finished dev [unoptimized + debuginfo] target(s) in 2.19s
		Running `target/debug/rust-state-machine`
	Hello, world!
	```

If we look at what has been generated, in that folder, you will see the following:

- [src/main.rs](#) - This is the entry point to your program. We will be building everything for this project in the `src` folder.

```rust filename="src/main.rs"
fn main() {
    println!("Hello, world!");
}
```

- [Cargo.toml](#) - This is a configuration file for your Rust project. Quite similar to a `package.json` that you would see in a Node.JS project. We will modify this in the future when we import crates to use in our project, but We can leave this alone for now.

```toml filename="Cargo.toml"
[package]
name = "rust-state-machine"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

```

- [Cargo.lock](#) - This is an autogenerated lock file based on your `cargo.toml` and the compilation. This usually defines the very specific versions of each crate being imported, and should not be manually edited.

```toml filename="Cargo.lock"
# This file is automatically @generated by Cargo.
# It is not intended for manual editing.
version = 3

[[package]]
name = "rust-state-machine"
version = "0.1.0"

```

- `target/*` - You might also see a target folder if you did `cargo run`. This is a folder where all the build artifacts are placed during compilation. We do not commit this folder into our git history.

All of this should be pretty familiar to you if you have already had some minimal experience with Rust. If any of this is new, I would suggest you first walk through the [Rust Book](https://doc.rust-lang.org/book/) and [Rust by Example](https://doc.rust-lang.org/rust-by-example/), as this is already an indication that this guide might not be targeted at your level of knowledge.
