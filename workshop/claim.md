# Claim POAP NFT 🎯

Next we'll need a way for users of this pallet to claim POAP NFTs.
We'll do this by creating a publicly callable function called `claim_poap_nft`.

This function will rely on an internal function called `do_claim` to perform checks and write changes to storage.
It will emit events and handle any errors and handle checks to enforce rules around claiming a POAP NFT from an existing collection.
These include:
* A collection must exist in order for its NFT to be claimable
* A collection cannot be claimed by an account that already has the maximum POAP NFTs allowed
* A collection cannot be claimed twice (not implemented)

To handle errors and events we'll need:
1. Errors `NoneLeftToClaim`, `CollectionDescriptionTooBig`, and `AlreadyClaimed` to emit relevant errors when checks do not go through.
1. A `CollectionItemClaimed` event, to emit an event when an NFT transfer is successful.

Then, in `do_claim` we'll:
1. Check if the collection exists.
1. Check that the collection isn't claimed twice (not implemented)
1. Update our `PoapNftsOwned` storage item to reflect the new owners and write all changes to storage.

<!-- slide:break-40 -->

<!-- tabs:start -->

#### ** 1. ADD ERRORS **

Add the `NoneLeftToClaim`, `CollectionDescriptionTooBig`, and `AlreadyClaimed` errors.

These can come up when trying to claim a POAP NFT.

```rust
// Your Pallet's error messages.
#[pallet::error]
pub enum Error<T> {
	/// The collection being minted already exists.
	DuplicateCollection,
	/// Account has exceeded MaxPaopOwned.
	TooManyOwned,
	/// This collection does not exist.
	CollectionDoesNotExist,
	/// There are no more NFTs of this collection to claim.
	NoneLeftToClaim,
	/// The description for this collection is too big.
	CollectionDescriptionTooBig,
	/// Collections can only be created with an amount greater than zero.
	AmountMustBeGreaterThanZero,
	/// The POAP NFT has already been claimed.
	AlreadyClaimed,
}
```

#### ** 2. ADD EVENT **

Add the `CollectionItemClaimed` event to your Pallet.

```rust
// Your Pallet's events.
#[pallet::event]
#[pallet::generate_deposit(pub(super) fn deposit_event)]
pub enum Event<T: Config> {
	/// A new collection was successfully created.
	Created { collection_id: [u8; 16], owner: T::AccountId },
	/// A collection's item was successfully claimed.
	CollectionItemClaimed { collection_id: [u8; 16], item_number: u32 },
}
```

#### ** 3. DO CLAIM **

Create an **internal** helper function which enables claiming unique POAP NFTs.

```rust
// Returns the POAP NFT claimed
pub fn do_claim(
	claimer: &T::AccountId,
	collection_id: [u8; 16],
) -> Result<PoapNft<T>, DispatchError> {
	// Get the collection and make sure it exists
	let from_collection = PoapCollections::<T>::get(&collection_id)
		.ok_or(Error::<T>::CollectionDoesNotExist)?;

	// Ensure there are still NFTs of this collection left to claim
	ensure!(from_collection.amount_claimable > 0, Error::<T>::NoneLeftToClaim);

	// Get item number from collection, which will be the number of NFTs left in this collection
	let number_claimed = from_collection.amount_claimable;

	// Update the new claimable value
	let new_amount_claimable =
		number_claimed.checked_sub(1).ok_or(ArithmeticError::Underflow)?;

	// Create a PoapNFT object with this collection's details and unique item id
	let poap_nft =
		PoapNft::<T> { collection_id, owner: claimer.clone(), item_number: number_claimed };

	// Gets the count to add new amount of claimed NFTs in existence
	let count = CountForPoapNfts::<T>::get();

	// Performs this operation first as it may fail
	let new_count = count.checked_add(1).ok_or(ArithmeticError::Overflow)?;

	// Write changes to storage
	CountForPoapNfts::<T>::put(new_count);

	// Append POAP NFT collection id to PoapNftsOwned
	PoapNftsOwned::<T>::try_append(&claimer, collection_id)
		.map_err(|_| Error::<T>::TooManyOwned)?;

	// Update Collection's available amount
	PoapCollections::<T>::try_mutate(collection_id, |maybe_details| {
		let details = maybe_details.as_mut().ok_or(Error::<T>::CollectionDoesNotExist)?;

		details.amount_claimable = new_amount_claimable;

		Self::deposit_event(Event::<T>::CollectionItemClaimed {
			collection_id,
			item_number: number_claimed,
		});

		Ok(poap_nft)
	})
```

#### ** 4. WRITE CLAIM **

Finally, create the actual **callable** function.

At this point, really minimal logic since we just call into our helper.

```rust
#[pallet::weight(0)]
pub fn claim_poap_nft(origin: OriginFor<T>, collection_id: [u8; 16]) -> DispatchResult {
	// Make sure the caller is from a signed origin
	let caller = ensure_signed(origin)?;

	Self::do_claim(&caller, collection_id)?;

	Ok(())
}
```

#### ** SOLUTION **

This should compile successfully by running:

```bash
cargo build -p pallet-template
```

There should be no warnings.

```rust
#![cfg_attr(not(feature = "std"), no_std)]

pub use pallet::*;

use frame_support::pallet_prelude::*;
use frame_support::traits::{Randomness};
use frame_system::pallet_prelude::*;
use scale_info::prelude::vec::Vec;
use sp_io::hashing::blake2_128;
use sp_runtime::ArithmeticError;

/* Placeholder for defining custom types. */

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

#[frame_support::pallet]
pub mod pallet {

	use super::*;

	// The struct on which we build all of our Pallet logic.
	#[pallet::pallet]
	pub struct Pallet<T>(_);

	/* Placeholder for defining custom storage items. */

	/// Keeps track of the number of claimed POAP NFTs in existence.
	/// Note that u64::MAX is 18,446,744,073,709,551,615
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

		/// The maximum length of a name or symbol stored on-chain.
		#[pallet::constant]
		type StringLimit: Get<u32>;
	}

	// Your Pallet's events.
	#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)]
	pub enum Event<T: Config> {
		/// A new collection was successfully created.
		Created { collection_id: [u8; 16], owner: T::AccountId },
		/// A collection's item was successfully claimed.
		CollectionItemClaimed { collection_id: [u8; 16], item_number: u32 },
	}

	// Your Pallet's error messages.
	#[pallet::error]
	pub enum Error<T> {
		/// The collection being minted already exists.
		DuplicateCollection,
		/// Account has exceeded MaxPaopOwned.
		TooManyOwned,
		/// This collection does not exist.
		CollectionDoesNotExist,
		/// There are no more NFTs of this collection to claim.
		NoneLeftToClaim,
		/// The description for this collection is too big.
		CollectionDescriptionTooBig,
		/// Collections can only be created with an amount greater than zero.
		AmountMustBeGreaterThanZero,
		/// The POAP NFT has already been claimed.
		AlreadyClaimed,
	}

	// Your Pallet's callable functions.
	#[pallet::call]
	impl<T: Config> Pallet<T> {
		/// Create a new unique POAP collection.
		///
		/// The actual POAP NFT creation is done in the `mint()` function.
		#[pallet::weight(0)]
		pub fn create_collection(
			origin: OriginFor<T>,
			description: Vec<u8>,
			amount: u32,
		) -> DispatchResult {
			// Make sure the caller is from a signed origin
			let caller = ensure_signed(origin)?;

			// Generate unique ID using a helper function
			let unique_id = Self::gen_id();

			// Write new collection to storage by calling helper function
			Self::mint(&caller, unique_id, description, amount)?;

			Ok(())
		}

		#[pallet::weight(0)]
		pub fn claim_poap_nft(origin: OriginFor<T>, collection_id: [u8; 16]) -> DispatchResult {
			// Make sure the caller is from a signed origin
			let caller = ensure_signed(origin)?;

			Self::do_claim(&caller, collection_id)?;

			Ok(())
		}
	}

	// Your Pallet's internal functions.
	impl<T: Config> Pallet<T> {
		// Helper to mint a collection.
		pub fn mint(
			owner: &T::AccountId,
			collection_id: [u8; 16],
			description: Vec<u8>,
			amount: u32,
		) -> Result<[u8; 16], DispatchError> {

			ensure!(amount > 0, Error::<T>::AmountMustBeGreaterThanZero);

			let bounded_description: BoundedVec<_, T::StringLimit> =
				description.try_into().map_err(|()| Error::<T>::CollectionDescriptionTooBig)?;

			// Create new collection
			let collection = PoapEvent::<T> {
				collection_id,
				description: bounded_description,
				total_created: amount,
				amount_claimable: amount,
				creator: owner.clone(),
			};

			// Check if the collection does not already exist in our storage map
			ensure!(
				!PoapCollections::<T>::contains_key(&collection.collection_id),
				Error::<T>::DuplicateCollection
			);

			// Write new POAP collection to storage
			PoapCollections::<T>::insert(collection.collection_id, collection);

			// Deposit our "Created" event.
			Self::deposit_event(Event::Created { collection_id, owner: owner.clone() });

			// Returns the unique collection id if this succeeds
			Ok(collection_id)
		}

		// Returns the POAP NFT claimed
		pub fn do_claim(
			claimer: &T::AccountId,
			collection_id: [u8; 16],
		) -> Result<PoapNft<T>, DispatchError> {
			// Get the collection and make sure it exists
			let from_collection = PoapCollections::<T>::get(&collection_id)
				.ok_or(Error::<T>::CollectionDoesNotExist)?;

			// Ensure there are still NFTs of this collection left to claim
			ensure!(from_collection.amount_claimable > 0, Error::<T>::NoneLeftToClaim);

			// Get item number from collection, which will be the number of NFTs left in this collection
			let number_claimed = from_collection.amount_claimable;

			// Update the new claimable value
			let new_amount_claimable =
				number_claimed.checked_sub(1).ok_or(ArithmeticError::Underflow)?;

			// Create a PoapNFT object with this collection's details and unique item id
			let poap_nft =
				PoapNft::<T> { collection_id, owner: claimer.clone(), item_number: number_claimed };

			// Gets the count to add new amount of claimed NFTs in existence
			let count = CountForPoapNfts::<T>::get();

			// Performs this operation first as it may fail
			let new_count = count.checked_add(1).ok_or(ArithmeticError::Overflow)?;

			// Write changes to storage
			CountForPoapNfts::<T>::put(new_count);

			// Append POAP NFT collection id to PoapNftsOwned
			PoapNftsOwned::<T>::try_append(&claimer, collection_id)
				.map_err(|_| Error::<T>::TooManyOwned)?;

			// Update Collection's available amount
			PoapCollections::<T>::try_mutate(collection_id, |maybe_details| {
				let details = maybe_details.as_mut().ok_or(Error::<T>::CollectionDoesNotExist)?;

				details.amount_claimable = new_amount_claimable;

				Self::deposit_event(Event::<T>::CollectionItemClaimed {
					collection_id,
					item_number: number_claimed,
				});

				Ok(poap_nft)
			})
		}

		pub fn gen_id() -> ([u8; 16]) {
			let subject = b"poap_collection";
			let (seed, _) = T::Randomness::random(subject);

			// Create randomness payload so that multiple collections can be generated in the same block,
			// retaining uniqueness.
			let unique_payload = (
				seed,
				frame_system::Pallet::<T>::extrinsic_index().unwrap_or_default(),
				frame_system::Pallet::<T>::block_number(),
			);

			// Turns into a byte array
			let encoded_payload = unique_payload.encode();
			let unique_id_hashed = blake2_128(&encoded_payload);

			return unique_id_hashed;
		}
	}
}
```

<!-- tabs:end -->
