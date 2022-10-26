## Customize the template pallet

We'll rewrite what's in the template pallet, but we'll use it to preserve the way it's already wired up in our node.

1. Navigate to the `pallets/template` folder of your project and remove everything.
1. Paste in the code in the solution box on the right.
1. Build your empty pallet:
```bash
cargo build -p pallet-template
```

While our pallet builds, lets decribe what the code we just pasted in does.

1. This macro tells our pallet to use Rust's `no_std` library unless `std` is specified.
```rust
#![cfg_attr(not(feature = "std"), no_std)]
```
1. This attribute ensures our pallet implements the required pallet attributes.
```rust
#[frame_support::pallet]
```
1. These are FRAME system utilities that gives us useful types and traits.
```rust
	use frame_support::pallet_prelude::*;
	use frame_system::pallet_prelude::*;
```
1. Pallet macros:
* `#[pallet::config]`: our pallet's configuration traits.
* `#[pallet::pallet]`: generates some storage item for our pallet.
* `#[pallet::event]` and `#[pallet::generate_deposit]`: enables our pallet to emit events.
* `#[pallet::error]`: enabels our pallets to emit errors.
* `#[pallet::call]`: a way for our pallet to instantiate calls that can be made from signed and unsigned origins.

These are the macros _required_ for a pallet to build.
We'll also be using additional ones as we build out our pallet.

Yay - that's basic stuff now lets get to building some custom logic. 🥳


<!-- slide:break-40 -->

<!-- tabs:start -->

#### ** Basic pallet structure **

```rust
#![cfg_attr(not(feature = "std"), no_std)]

pub use pallet::*;

#[frame_support::pallet]
pub mod pallet {
	use frame_support::pallet_prelude::*;
	use frame_system::pallet_prelude::*;

	#[pallet::config]
	pub trait Config: frame_system::Config {
		type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;
	}

	#[pallet::pallet]
	pub struct Pallet<T>(_);

	#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)]
	pub enum Event<T: Config> {
	}

	#[pallet::error]
	pub enum Error<T> {
	}

	#[pallet::call]
	impl<T: Config> Pallet<T> {
	}
}
```

<!-- tabs:end -->
