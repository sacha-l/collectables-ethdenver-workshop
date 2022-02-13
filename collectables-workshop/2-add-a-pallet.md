# Project structure

1. Open up your code editor with your compiled node template.
(Or, if you're using the Substrate Playground, go to a Node Template instance and hit `Ctrl + c` to stop the chain).
1. Notice the `node`, `pallets` and `runtime` folders
1. Go to `runtime/src/lib.rs` and `Ctrl + f` "construct_runtime!("
This macro compiles externally published pallets to generates the state transition function of our chain. 
We currently have: 

* `System: frame_system`: core system level functionality.
* `RandomnessCollectiveFlip: pallet_randomness_collective_flip`: provides on-chain pseudo-randomness.
* `Timestamp: pallet_timestamp`: provides on-chain timestamping utilities.
* `Aura: pallet_aura`: proof of authority consensus.
* `Grandpa: pallet_grandpa`: block finalization utility.
* ⭐️ `Balances: pallet_balances`: the underlying currency system for our chain.
* `TransactionPayment: pallet_transaction_payment`: transaction fee system.
* `Sudo: pallet_sudo`: most basic on-chain goverance.
* ⭐️ `TemplateModule: pallet_template`: an dummy pallet to make it easy to start hacking with.
		
## Customize the template pallet

No need to reinvent the wheel right? 
Let's use the template pallet as a starting point!

1. Navigate to the `pallets/template` folder of your project
1. Take everything out except the FRAME macros (see the solution here on the left).
	- What's FRAME ? 💡
	- What are these macros? 🤔 
1. Build your empty pallet:
```bash
cargo build -p pallet-template
```

Yay - that's basic stuff now lets get to building some custom logic. 


<!-- slide:break-40 -->

<!-- tabs:start -->

#### ** Solution **

```rust
#![cfg_attr(not(feature = "std"), no_std)]

/// Edit this file to define custom logic or remove it if it is not needed.
/// Learn more about FRAME and the core library of Substrate FRAME pallets:
/// <https://docs.substrate.io/v3/runtime/frame>
#[frame_support::pallet]
pub mod pallet {
	use frame_support::pallet_prelude::*;
	use frame_system::pallet_prelude::*;

	/// Configure the pallet by specifying the parameters and types on which it depends.
	#[pallet::config]
	pub trait Config: frame_system::Config {
		/// Because this pallet emits events, it depends on the runtime's definition of an event.
		type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;
	}

	#[pallet::pallet]
	#[pallet::generate_store(pub(super) trait Store)]
	pub struct Pallet<T>(_);

	// Pallets use events to inform users when important changes are made.
	// https://docs.substrate.io/v3/runtime/events-and-errors
	#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)]
	pub enum Event<T: Config> {

	}

	// Errors inform users that something went wrong.
	#[pallet::error]
	pub enum Error<T> {
	}

	// Dispatchable functions allows users to interact with the pallet and invoke state changes.
	// These functions materialize as "extrinsics", which are often compared to transactions.
	// Dispatchable functions must be annotated with a weight and must return a DispatchResult.
	#[pallet::call]
	impl<T: Config> Pallet<T> {
	}
}
```

<!-- tabs:end -->