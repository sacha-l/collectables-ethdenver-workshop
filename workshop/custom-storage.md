# Custom Storage

Our Pallet will use 3 storage items to track all of the state changes.

1. A `StorageValue` named `CountForPoapNfts` which will keep track of the total number of NFTs claimed in the Pallet.
	* This `StorageValue` simply keeps track of a `u64` value that we increment whenever an NFT is claimed. This means we could have up to `18_446_744_073_709_551_615` claimed NFTs in existence.... probably more than we will ever need to worry about. 💪
	* It is worth noting the `ValueQuery` configuration in the `StorageValue`. This basically assumes if there is no value in storage, for example at the start of the network, that we should return the value `0` rather than an option `None`. 📝
2. A `StorageMap` named `PoapCollections` which will map each collection to it's unique information.
	* To keep collections completely unique and easy to look up, the key of our map is the `collection_id` of the claimed NFT. As such, we cannot have two collections with the same `collection_id` since the map will have already been populated by one of them. ✅
3. A `StorageMap` named `PoapNftsOwned` which will map each user to a list of POAP NFTs they own.
	* The _key_ for this storage map will be a user account: `T::AccountID`.
	* The _value_ for this storage map will be a `BoundedVec` with the `collection_id` of the NFT from the collection they own. This will make it easy to then look up each individual POAP NFT for its information since the `collection_id` is used as the key for the `PoapCollections` map.
	* By using a `BoundedVec`, we ensure that each storage item has a maximum length, which is important for managing limits within the runtime.

<!-- slide:break-40 -->

<!-- tabs:start -->

#### ** ACTION ITEMS **

Add the following custom storage items to your Pallet.

```rust
/// Keeps track of the number of claimed POAP NFTs in existence.
#[pallet::storage]
pub(super) type CountForPoapNfts<T: Config> = StorageValue<_, u64, ValueQuery>;

/// Maps the POAP Collection to its unique ID.
#[pallet::storage]
pub(super) type PoapCollections<T: Config> =
	StorageMap<_, Twox64Concat, [u8; 16], PoapEvent<T>>;

/// Track the POAP NFTs owned by each account.
#[pallet::storage]
pub(super) type PoapNftsOwned<T: Config> = StorageMap<
	_,
	Twox64Concat,
	T::AccountId,
	BoundedVec<[u8; 16], T::MaxPoapOwned>,
	OptionQuery,
>;
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

	use frame_support::traits::{Currency, Randomness};

	// The basis required for building this pallet
	#[pallet::pallet]
	pub struct Pallet<T>(_);

	// Struct for holding POAP Collection details (by admin)
	#[derive(Clone, Encode, Decode, PartialEq, RuntimeDebug, TypeInfo, MaxEncodedLen)]
	#[scale_info(skip_type_params(T))]
	#[codec(mel_bound())]
	pub struct PoapEvent<T: Config> {
		// Using 16 bytes to represent a unique ID
		pub collection_id: [u8; 16],
		// Vec<u8> to hold a short description
		pub description: BoundedVec<u8, T::StringLimit>,
		// Amount originally created
		pub total_created: u32,
		// Amount of NFTs left in this collection
		pub amount_claimable: u32,
		// Creator of the collection
		pub creator: T::AccountId,
	}

	// Struct for holding POAP NFT details
	#[derive(Clone, Encode, Decode, PartialEq, RuntimeDebug, TypeInfo, MaxEncodedLen, Copy)]
	#[scale_info(skip_type_params(T))]
	pub struct PoapNft<T: Config> {
		// Using 16 bytes to represent the unique collection ID
		pub collection_id: [u8; 16],
		// Owner of POAP NFT
		pub owner: T::AccountId,
		// Item number of the NFT in the collection
		pub item_number: u32,
	}

	/// Keeps track of the number of claimed POAP NFTs in existence.
	#[pallet::storage]
	pub(super) type CountForPoapNfts<T: Config> = StorageValue<_, u64, ValueQuery>;

	/// Maps the POAP Collection to its unique ID.
	#[pallet::storage]
	pub(super) type PoapCollections<T: Config> =
		StorageMap<_, Twox64Concat, [u8; 16], PoapEvent<T>>;

	/// Track the POAP NFTs owned by each account.
	#[pallet::storage]
	pub(super) type PoapNftsOwned<T: Config> = StorageMap<
		_,
		Twox64Concat,
		T::AccountId,
		BoundedVec<[u8; 16], T::MaxPoapOwned>,
		OptionQuery,
	>;

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
