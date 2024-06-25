# Enable Balance Transfers

Now that we have initialized and started to use our balances module, let's add probably the most important API: `transfer`.

## Learn

Before we write our function, it is important that we review some of the principles of blockchain and Rust.

### Bad Actors

In a blockchain system, security is paramount. Bad actors may attempt to exploit vulnerabilities, such as insufficient balances during fund transfers, or overflow / underflow issues. Rust's safe math and error handling mechanisms help mitigate these risks.

### Safe Math

Rust's safe math operations prevent overflow and underflow. The `checked_add` and `checked_sub` methods return an `Option` that allows handling potential arithmetic errors safely.

In Rust, the Option type is a fundamental part of the standard library, designed to handle scenarios where a value may or may not be present. It's commonly used in situations where the result of an operation might be undefined or absent.

Methods like `checked_add` and `checked_sub` return `Option` to indicate success or failure due to overflow or underflow.

```rust
let result = a.checked_add(b);
match result {
    Some(sum) => println!("Sum: {}", sum),
    None => println!("Overflow occurred."),
}
```

### Error Handling

In Rust, error handling is an integral part of writing robust and safe code. The Result type is commonly used for functions that may encounter errors during their execution.

The Result type is an enum defined in the standard library. It has two variants: `Ok(value)` for a successful result and `Err(error)` for an error:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` and `E` are generic parameters that allow you to customize the result type for your needs. For the purposes of this tutorial, we will always return `Ok(())` when everything completes okay, and a `Err(&'static str)` to describe any errors with a basic string.

You can then define the `Result` type like:

```rust
Result<(), &'static str>
```

### Options and Results

You can use the `Option` type to trigger an `Err`, which is helpful when you only want your function to execute when everything goes as expected.

In this context, we want a function that will return an error whenever some safe math operation returns `None`.

For this, we can chain `ok_or` along with `?` directly after the safe math operation like so:

```rust
let new_from_balance = from_balance
    .checked_sub(amount)
    .ok_or("Not enough funds.")?;
```

If `checked_sub` returns `None`, we will then return an `Err` with the message `"Not enough funds."` that can be displayed to the user. Otherwise, if `checked_sub` returns `Some(value)`, we will assign `new_from_balance` directly to that value.

In this case, we are writing code which completely handles the `Option` type in a safe and ergonomic way.

## Create Transfer

Follow the instructions in the following template to create a safe and simple transfer function in your Balances Pallet.

Create a test showing that everything is working as expected, including error handling. There should be no compiler warnings!

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

	/// Transfer `amount` from one account to another.
	/// This function verifies that `from` has at least `amount` balance to transfer,
	/// and that no mathematical overflows occur.
	pub fn transfer(
		&mut self,
		caller: String,
		to: String,
		amount: u128,
	) -> Result<(), &'static str> {
		/* TODO:
			- Get the balance of account `caller`.
			- Get the balance of account `to`.

			- Use safe math to calculate a `new_caller_balance`.
			- Use safe math to calculate a `new_to_balance`.

			- Insert the new balance of `caller`.
			- Insert the new balance of `to`.
		*/

		Ok(())
	}
}

#[cfg(test)]
mod tests {
	#[test]
	fn init_balances() {
		let mut balances = super::Pallet::new();

		assert_eq!(balances.balance(&"alice".to_string()), 0);
		balances.set_balance(&"alice".to_string(), 100);
		assert_eq!(balances.balance(&"alice".to_string()), 100);
		assert_eq!(balances.balance(&"bob".to_string()), 0);
	}

	#[test]
	fn transfer_balance() {
		/* TODO: Create a test that checks the following:
			- That `alice` cannot transfer funds she does not have.
			- That `alice` can successfully transfer funds to `bob`.
			- That the balance of `alice` and `bob` is correctly updated.
		*/
	}
}
```
