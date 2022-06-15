# Configuring Your Pallet

To build our pallet, we need to include some custom configurations which will allow our pallet to gain access to outside interfaces like:

* Generating on-chain randomness.
* Setting limits for how many NFTs a single user can own.
* Setting limits to how many bytes a description can be.

We will introduce these to the `trait Config` for our Pallet.

To do this, this we use a few different tools:

* `Get<u32>`: A trait which simply fetches a `u32` value, allowing the user to configure the `MaxPoapOwned`.
* `Randomness`: A trait which describes an interface to access an on-chain random value.

We will use these interfaces in the future, but a sneak peak to how you might actually see these used in the code:

```rust
// Get some on-chain randomness.
let (seed, _) = T::Randomness::random(subject);

// Get the `MaxPoapOwned` limit.
let poap_owned_limit: u32 = T::MaxPoapOwned::get();

```

<!-- slide:break -->

<!-- tabs:start -->

#### ** ACTION ITEMS **

Import the `Randomness` trait to your project:

```rust
use frame_support::traits::{Randomness};
```

Then, update your `trait Config` to have the following:

```rust
// Your Pallet's configuration trait, representing custom external types and interfaces.
#[pallet::config]
pub trait Config: frame_system::Config {
	/// Because this pallet emits events, it depends on the runtime's definition of an event.
	type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;

	/// Something that provides randomness in the runtime.
	type Randomness: Randomness<Self::Hash, Self::BlockNumber>;

	/// The maximum amount of POAP NFTs a single account can own.
	#[pallet::constant]
	type MaxPoapOwned: Get<u32>;

    /// The maximum length for a description stored on-chain.
    #[pallet::constant]
    type StringLimit: Get<u32>;
}
```

#### ** SOLUTION **

This should compile successfully by running:

```bash
cargo build -p pallet-template
```

Don't worry about warnings.

```rust
#![cfg_attr(not(feature = "std"), no_std)]

pub use pallet::*;

#[frame_support::pallet]
pub mod pallet {
    use frame_support::pallet_prelude::*;
    use frame_system::pallet_prelude::*;

    use frame_support::traits::{Randomness};

    // The struct on which we build all of our Pallet logic.
    #[pallet::pallet]
    pub struct Pallet<T>(_);

    /* Placeholder for defining custom types. */

    /* Placeholder for defining custom storage items. */

    // Your Pallet's configuration trait, representing custom external types and interfaces.
    #[pallet::config]
    pub trait Config: frame_system::Config {
        /// Because this pallet emits events, it depends on the runtime's definition of an event.
        type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;

		/// Something that provides randomness in the runtime.
		type Randomness: Randomness<Self::Hash, Self::BlockNumber>;

        /// The maximum amount of POAP NFTs a single account can own.
        #[pallet::constant]
        type MaxPoapOwned: Get<u32>;
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

<!-- tabs:end -->
