# Pallet Level Dispatch

We want to make our code more modular and extensible.

Currently, all dispatch happens through the `RuntimeCall`, which is hardcoding dispatch logic for each of the Pallets in our system.

What we would prefer is for Pallet level dispatch logic to live in the Pallet itself, and our Runtime taking advantage of that. We have already seen end to end what it takes to set up call dispatch, so let's do it again at the Pallet level.

## Pallet Call

To make our system more extensible, we want to keep all the calls for a pallet defined at the pallet level.

For this, we define an `enum Call` in our Balances pallet, and just like before, we introduce a new enum variant representing the function that we want to call.

Note that this enum needs to be generic over `T: Config` because we need access to the types defined by our configuration trait!

## Pallet Dispatch

You will also notice in the template, we have included the shell for you to implement Pallet level dispatch.

Everything should look the same as the Runtime level dispatch, except the `type Call` is the Pallet level call we just created.

Just like before, you simply need to match the `Call` variant with the appropriate function, and pass the parameters needed by the function.

## Create Your Pallet Level Dispatch

Follow the `TODO`s in the template below to complete the logic for Pallet level dispatch.

The "never constructed" warning for the `Transfer` variant is okay.

In the next step, we will use this logic to improve our dispatch logic in our Runtime.

```rust filename="src/balances.rs"
use num::traits::{CheckedAdd, CheckedSub, Zero};
use std::collections::BTreeMap;

/// The configuration trait for the Balances Module.
/// Contains the basic types needed for handling balances.
pub trait Config: crate::system::Config {
	/// A type which can represent the balance of an account.
	/// Usually this is a large unsigned integer.
	type Balance: Zero + CheckedSub + CheckedAdd + Copy;
}

/// This is the Balances Module.
/// It is a simple module which keeps track of how much balance each account has in this state
/// machine.
#[derive(Debug)]
pub struct Pallet<T: Config> {
	// A simple storage mapping from accounts to their balances.
	balances: BTreeMap<T::AccountId, T::Balance>,
}

impl<T: Config> Pallet<T> {
	// Create a new instance of the balances module.
	pub fn new() -> Self {
		Self { balances: BTreeMap::new() }
	}

	/// Set the balance of an account `who` to some `amount`.
	pub fn set_balance(&mut self, who: &T::AccountId, amount: T::Balance) {
		self.balances.insert(who.clone(), amount);
	}

	/// Get the balance of an account `who`.
	/// If the account has no stored balance, we return zero.
	pub fn balance(&self, who: &T::AccountId) -> T::Balance {
		*self.balances.get(who).unwrap_or(&T::Balance::zero())
	}

	/// Transfer `amount` from one account to another.
	/// This function verifies that `from` has at least `amount` balance to transfer,
	/// and that no mathematical overflows occur.
	pub fn transfer(
		&mut self,
		caller: T::AccountId,
		to: T::AccountId,
		amount: T::Balance,
	) -> crate::support::DispatchResult {
		let caller_balance = self.balance(&caller);
		let to_balance = self.balance(&to);

		let new_caller_balance = caller_balance.checked_sub(&amount).ok_or("Not enough funds.")?;
		let new_to_balance = to_balance.checked_add(&amount).ok_or("Overflow")?;

		self.balances.insert(caller, new_caller_balance);
		self.balances.insert(to, new_to_balance);

		Ok(())
	}
}

// A public enum which describes the calls we want to expose to the dispatcher.
// We should expect that the caller of each call will be provided by the dispatcher,
// and not included as a parameter of the call.
pub enum Call<T: Config> {
	/* TODO: Create an enum variant `Transfer` which contains named fields:
		- `to`: a `T::AccountId`
		- `amount`: a `T::Balance`
	*/
	/* TODO: Remove the `RemoveMe` placeholder. */
	RemoveMe(core::marker::PhantomData<T>),
}

/// Implementation of the dispatch logic, mapping from `BalancesCall` to the appropriate underlying
/// function we want to execute.
impl<T: Config> crate::support::Dispatch for Pallet<T> {
	type Caller = T::AccountId;
	type Call = Call<T>;

	fn dispatch(
		&mut self,
		caller: Self::Caller,
		call: Self::Call,
	) -> crate::support::DispatchResult {
		/* TODO: use a `match` statement to route the `Call` to the appropriate pallet function. */
		Ok(())
	}
}

#[cfg(test)]
mod tests {
	struct TestConfig;

	impl crate::system::Config for TestConfig {
		type AccountId = String;
		type BlockNumber = u32;
		type Nonce = u32;
	}

	impl super::Config for TestConfig {
		type Balance = u128;
	}

	#[test]
	fn init_balances() {
		let mut balances = super::Pallet::<TestConfig>::new();

		assert_eq!(balances.balance(&"alice".to_string()), 0);
		balances.set_balance(&"alice".to_string(), 100);
		assert_eq!(balances.balance(&"alice".to_string()), 100);
		assert_eq!(balances.balance(&"bob".to_string()), 0);
	}

	#[test]
	fn transfer_balance() {
		let mut balances = super::Pallet::<TestConfig>::new();

		assert_eq!(
			balances.transfer("alice".to_string(), "bob".to_string(), 51),
			Err("Not enough funds.")
		);

		balances.set_balance(&"alice".to_string(), 100);
		assert_eq!(balances.transfer("alice".to_string(), "bob".to_string(), 51), Ok(()));
		assert_eq!(balances.balance(&"alice".to_string()), 49);
		assert_eq!(balances.balance(&"bob".to_string()), 51);

		assert_eq!(
			balances.transfer("alice".to_string(), "bob".to_string(), 51),
			Err("Not enough funds.")
		);
	}
}
```
