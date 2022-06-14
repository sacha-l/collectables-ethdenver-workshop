# Setup Your Project

1. Make sure you already have Rust build environment setup on your computer:

	https://docs.substrate.io/v3/getting-started/installation/

2. Clone the [`substrate-node-template`](https://github.com/substrate-developer-hub/substrate-node-template).

```bash
git clone https://github.com/substrate-developer-hub/substrate-node-template
```

3. Replace the contents of `substrate-node-template/pallets/template/src/lib.rs` with the **Empty Pallet Template**.

4. Compile your project. Don't worry about warnings. 🙂

```bash
cargo build -p pallet-template
```

If you got this far, then you are ready to move forward!

<!-- slide:break -->

# Empty Pallet Template

```rust
#![cfg_attr(not(feature = "std"), no_std)]

pub use pallet::*;

#[frame_support::pallet]
pub mod pallet {
	use frame_support::pallet_prelude::*;
	use frame_system::pallet_prelude::*;

	// The struct on which we build all of our Pallet logic.
	#[pallet::pallet]
	pub struct Pallet<T>(_);

	/* Placeholder for defining custom types. */

	/* Placeholder for defining custom storage items. */

	// Your Pallet's configuration trait, representing custom external types and interfaces.
	#[pallet::config]
	pub trait Config: frame_system::Config {
		type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;
	}

	// Your Pallet's events.
	#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)]
	pub enum Event<T: Config> {}

	// Your Pallet's error messages.
	#[pallet::error]
	pub enum Error<T> {}

	// Your Pallet's callable functions.
	#[pallet::call]
	impl<T: Config> Pallet<T> {}

	// Your Pallet's internal functions.
	impl<T: Config> Pallet<T> {}
}
```
