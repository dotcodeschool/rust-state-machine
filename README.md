# Nested Dispatch

Now that we have defined Pallet level dispatch logic in the Pallet, we should update our Runtime to take advantage of that logic.

After this, whenever the Pallet logic is updated, the Runtime dispatch logic will also automatically get updated and route calls directly. This makes our code easier to manage, and prevent potential errors or maintenance in the future.

## Nested Calls

The Balances Pallet now exposes its own list of calls in `balances::Call`. Rather than list them all again in the Runtime, we can use a nested enum to route our calls correctly.

Imagine the following construction:

```rust
pub enum RuntimeCall {
	Balances(balances::Call<Runtime>),
}
```

In this case, we have a variant `RuntimeCall::Balances`, which itself contains a type `balances::Call`. This means we can access all the calls exposed by `balances:Call` under this variant. As we create more pallets or extend our calls, this nested structure will scale very well.

We call the `RuntimeCall` an "outer enum", and the `balances::Call` an "inter enum". This construction of using outer and inter enums is very common in the Polkadot SDK.

## Re-Dispatching to Pallet

Our current `dispatch` logic directly calls the functions in the Pallet. As we mentioned, having this logic live outside of the Pallet can increase the burden of maintenance or errors.

But now that we have defined Pallet level dispatch logic in the Pallet itself, we can use this to make the Runtime dispatch more extensible.

To do this, rather than calling the Pallet function directly, we can extract the inner call from the `RuntimeCall`, and then use the `balances::Pallet` to dispatch that call to the appropriate logic.

That would look something like:

```rust
match runtime_call {
	RuntimeCall::Balances(call) => {
		self.balances.dispatch(caller, call)?;
	},
}
```

Here you can see that the first thing we do is check that the call is a `Balances` variant, then we extract from it the `call` which is a `balances::Call` type, and then we use `self.balances` which is a `balances::Pallet` to dispatch the `balances::Call`.

## Updating Your Block

Since we have updated the construction of the `RuntimeCall` enum, we will also need to update our `Block` construction in `fn main`. Nothing magical here, just needing to construct a nested enum using both `RuntimeCall::Balances` and `balances::Call::Transfer`.

## Enable Nested Dispatch

Now is the time to complete this step and glue together Pallet level dispatch with the Runtime level dispatch logic.

Follow the `TODO`s provided in the template below to get your full end to end dispatch logic running. By the end of this step there should be no compiler warnings.

```rust filename="src/main.rs"
mod balances;
mod support;
mod system;

use crate::support::Dispatch;

// These are the concrete types we will use in our simple state machine.
// Modules are configured for these types directly, and they satisfy all of our
// trait requirements.
mod types {
	pub type AccountId = String;
	pub type Balance = u128;
	pub type BlockNumber = u32;
	pub type Nonce = u32;
	pub type Extrinsic = crate::support::Extrinsic<AccountId, crate::RuntimeCall>;
	pub type Header = crate::support::Header<BlockNumber>;
	pub type Block = crate::support::Block<Header, Extrinsic>;
}

// These are all the calls which are exposed to the world.
// Note that it is just an accumulation of the calls exposed by each module.
pub enum RuntimeCall {
	/* TODO: Turn this into a nested enum where variant `Balances` contains a `balances::Call`. */
	BalancesTransfer { to: types::AccountId, amount: types::Balance },
}

// This is our main Runtime.
// It accumulates all of the different pallets we want to use.
#[derive(Debug)]
pub struct Runtime {
	system: system::Pallet<Self>,
	balances: balances::Pallet<Self>,
}

impl system::Config for Runtime {
	type AccountId = types::AccountId;
	type BlockNumber = types::BlockNumber;
	type Nonce = types::Nonce;
}

impl balances::Config for Runtime {
	type Balance = types::Balance;
}

impl Runtime {
	// Create a new instance of the main Runtime, by creating a new instance of each pallet.
	fn new() -> Self {
		Self { system: system::Pallet::new(), balances: balances::Pallet::new() }
	}

	// Execute a block of extrinsics. Increments the block number.
	fn execute_block(&mut self, block: types::Block) -> support::DispatchResult {
		self.system.inc_block_number();
		if block.header.block_number != self.system.block_number() {
			return Err(&"block number does not match what is expected");
		}
		// An extrinsic error is not enough to trigger the block to be invalid. We capture the
		// result, and emit an error message if one is emitted.
		for (i, support::Extrinsic { caller, call }) in block.extrinsics.into_iter().enumerate() {
			self.system.inc_nonce(&caller);
			let _res = self.dispatch(caller, call).map_err(|e| {
				eprintln!(
					"Extrinsic Error\n\tBlock Number: {}\n\tExtrinsic Number: {}\n\tError: {}",
					block.header.block_number, i, e
				)
			});
		}
		Ok(())
	}
}

impl crate::support::Dispatch for Runtime {
	type Caller = <Runtime as system::Config>::AccountId;
	type Call = RuntimeCall;
	// Dispatch a call on behalf of a caller. Increments the caller's nonce.
	//
	// Dispatch allows us to identify which underlying module call we want to execute.
	// Note that we extract the `caller` from the extrinsic, and use that information
	// to determine who we are executing the call on behalf of.
	fn dispatch(
		&mut self,
		caller: Self::Caller,
		runtime_call: Self::Call,
	) -> support::DispatchResult {
		// This match statement will allow us to correctly route `RuntimeCall`s
		// to the appropriate pallet level function.
		match runtime_call {
			/*
				TODO:
				Adjust this logic to handle the nested enums, and simply call the `dispatch` logic
				on the balances call, rather than the function directly.
			*/
			RuntimeCall::BalancesTransfer { to, amount } => {
				self.balances.transfer(caller, to, amount)?;
			},
		}
		Ok(())
	}
}

fn main() {
	// Create a new instance of the Runtime.
	// It will instantiate with it all the modules it uses.
	let mut runtime = Runtime::new();
	let alice = "alice".to_string();
	let bob = "bob".to_string();
	let charlie = "charlie".to_string();

	// Initialize the system with some initial balance.
	runtime.balances.set_balance(&alice, 100);

	// Here are the extrinsics in our block.
	// You can add or remove these based on the modules and calls you have set up.
	let block_1 = types::Block {
		header: support::Header { block_number: 1 },
		extrinsics: vec![
			/* TODO: Update your extrinsics to use the nested enum. */
			support::Extrinsic {
				caller: alice.clone(),
				call: RuntimeCall::BalancesTransfer { to: bob, amount: 30 },
			},
			support::Extrinsic {
				caller: alice,
				call: RuntimeCall::BalancesTransfer { to: charlie, amount: 20 },
			},
		],
	};

	// Execute the extrinsics which make up our block.
	// If there are any errors, our system panics, since we should not execute invalid blocks.
	runtime.execute_block(block_1).expect("invalid block");

	// Simply print the debug format of our runtime state.
	println!("{:#?}", runtime);
}
```