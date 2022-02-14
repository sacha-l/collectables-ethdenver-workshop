# Structs and types

Similar to a typical ERC721 NFT, our pallet will have:

- An object with unique features and a unique identifier (`[u8; 16]`) that corresponds to a Kitty's unique DNA
- 3 storage items to handle our pallet's NFT features

In this part we will:

1. Write the kitty struct, which will also be what we'll write out first storage item for.
1. Add derive attributes for our struct to be encodable and decodable.
1. Add accounts and balances.
1. Create the `Gender` type.

Check your code against the solution and let's move on to adding storage items for our kitties!

<!-- slide:break-40 -->

<!-- tabs:start -->


#### ** 1. Write a struct **

Your Kitty struct will contain the following fields:

* `dna`: the unique identifier of this kitty.
* `price`: some price for this kitty, or `None`.
* `gender`: some unique gender.
* `owner`: the account that owns it.

```rust
pub struct Kitty<T: Config> {
	pub dna: [u8; 16],
	pub price: Option<BalanceOf<T>>,
	pub gender: Gender,
	pub owner: T::AccountId,
}
```

#### ** 2. Enable encode/decode **

As FRAME runtime engineers, we're able to use special macros to ensure that the object we created can be encodable/decodable and will expose the correct metadata including information about the custom type. 

Add these before the struct declaration:

```rust
#[derive(Clone, Encode, Decode, PartialEq, RuntimeDebug, TypeInfo, MaxEncodedLen)]
#[scale_info(skip_type_params(T))]
```

#### ** 3. Add accounts and balances  **

Our kitty contains an owner and price field with a special types that we need to define. 
To help us, the `frame_support` and `frame_system` crates provides us with:
- a `Currency` trait that gives us a way create a type to handle the balances of an account
- an `AccountId` type 

1. Add the `Currency` dependency.
```rust
use frame_support::traits::Currency;
```
1. In order to implement some way for our pallet to understand accounts and balances, we also need to add a currency type in our configuration trait:
```rust
#[pallet::config]
pub trait Config: frame_system::Config {
	//--snip--
	type Currency: Currency<Self::AccountId>;
}
```
1. Write your pallet's balances handler:
```rust
type BalanceOf<T> =
	<<T as Config>::Currency as Currency<<T as frame_system::Config>::AccountId>>::Balance;
```

#### ** 4. Create the Gender type **

Let's create an enum to handle the `Gender` type for our kitty, super simple:

```rust 
	pub enum Gender {
		Male,
		Female,
	}
```

Here too we'll need to add some derive macros to make this type encodable/decodable and provide type information in the metadata of this pallet.
And one more thing too: we need to make this struct serializable to be able to pass it into the genesis configuration (which we'll get to later on)💡.

Add these lines above the enum declaration:

```rust
#[derive(Clone, Encode, Decode, PartialEq, RuntimeDebug, TypeInfo, MaxEncodedLen)]
#[cfg_attr(feature = "std", derive(Serialize, Deserialize))]
```

And finally, import the `Serialize` and `Deserialize` traits:

```rust
#[cfg(feature = "std")]
use frame_support::serde::{Deserialize, Serialize};
```

And add the required crate (in your pallet's Cargo.toml file):

```toml
[dependencies]
serde = { version = "1.0.119" }
```

#### ** Solution **

```rust
#![cfg_attr(not(feature = "std"), no_std)]
#[frame_support::pallet]
pub mod pallet {
	use frame_support::pallet_prelude::*;
	use frame_system::pallet_prelude::*;
	use frame_support::traits::Currency;
	use scale_info::TypeInfo;

	#[cfg(feature = "std")]
	use frame_support::serde::{Deserialize, Serialize};

	// Handles our pallet's currency abstraction
	type BalanceOf<T> =
		<<T as Config>::Currency as Currency<<T as frame_system::Config>::AccountId>>::Balance;

	// Struct for holding kitty information
	#[derive(Clone, Encode, Decode, PartialEq, RuntimeDebug, TypeInfo, MaxEncodedLen)]
	#[scale_info(skip_type_params(T))]
	pub struct Kitty<T: Config> {
		pub dna: [u8; 16],
		pub price: Option<BalanceOf<T>>,
		pub gender: Gender,
		pub owner: T::AccountId,
	}

	// Set Gender type in kitty struct
	#[derive(Clone, Encode, Decode, PartialEq, RuntimeDebug, TypeInfo, MaxEncodedLen)]
	#[cfg_attr(feature = "std", derive(Serialize, Deserialize))]
	pub enum Gender {
		Male,
		Female,
	}

	/// Configure the pallet by specifying the parameters and types on which it depends.
	#[pallet::config]
	pub trait Config: frame_system::Config {
		type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;
	}

	#[pallet::pallet]
	#[pallet::generate_store(pub(super) trait Store)]
	pub struct Pallet<T>(_);

	// Pallets use events to inform users when important changes are made.
	#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)]
	pub enum Event<T: Config> {

	}

	// Errors inform users that something went wrong.
	#[pallet::error]
	pub enum Error<T> {
	}

	// Dispatchable functions allows users to interact with the pallet and invoke state changes.
	#[pallet::call]
	impl<T: Config> Pallet<T> {
	}
}

```

<!-- tabs:end -->
