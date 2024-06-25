# Basic Balance Test

Now that we have the basics of our `Pallet` set up, let's actually interact with it.

For that, we will create our first `#[test]` which will play with the code we have written so far.

1. Copy the following code into your `src/balances.rs` file, specifically the new `#[cfg(test)]` module (the rest of the code should be the same as before):

```rust filename="src/balances.rs"
use std::collections::BTreeMap;

/// This is the Balances Module.
/// It is a simple module which keeps track of how much balance each account has in this state
/// machine.
pub struct Pallet {
	// A simple storage mapping from accounts (`String`) to their balances (`u128`).
	balances: BTreeMap<String, u128>,
}

impl Pallet {
	/// Create a new instance of the balances module.
	pub fn new() -> Self {
		Self { balances: BTreeMap::new() }
	}

	/// Set the balance of an account `who` to some `amount`.
	pub fn set_balance(&mut self, who: &String, amount: u128) {
		self.balances.insert(who.clone(), amount);
	}

	/// Get the balance of an account `who`.
	/// If the account has no stored balance, we return zero.
	pub fn balance(&self, who: &String) -> u128 {
		*self.balances.get(who).unwrap_or(&0)
	}
}

#[cfg(test)]
mod tests {
	#[test]
	fn init_balances() {
		/* TODO: Create a mutable variable `balances`, which is a new instance of `Pallet`. */

		/* TODO: Assert that the balance of `alice` starts at zero. */
		/* TODO: Set the balance of `alice` to 100. */
		/* TODO: Assert the balance of `alice` is now 100. */
		/* TODO: Assert the balance of `bob` has not changed and is 0. */
	}
}
```

2. In your `src/balances.rs` file, add a new `#[test]` named `fn init_balances()`:

	```rust
	#[test]
	fn init_balances() { }
	```

3. To begin our test, we need to initialize a new instance of our `Pallet`:

	```rust
	#[test]
	fn init_balances() {
		let mut balances = super::Pallet::new();
	}
	```

	Note that we make this variable `mut` since we plan to mutate our state using our newly created API.

4. Finally, let's check that our read and write APIs are working as expected:

	```rust
	#[test]
	fn init_balances() {
		let mut balances = super::Pallet::new();

		assert_eq!(balances.balance(&"alice".to_string()), 0);
		balances.set_balance(&"alice".to_string(), 100);
		assert_eq!(balances.balance(&"alice".to_string()), 100);
		assert_eq!(balances.balance(&"bob".to_string()), 0);
	}
	```

5. We can run our tests using `cargo test`, where hopefully you should see that it passes. **There should be no compiler warnings now!**

I hope at this point you can start to see the beginnings of your simple blockchain state machine.
