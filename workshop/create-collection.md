# Create POAP collection 🎉

> Now that we have set up our custom types and storage items, we can start to write functions that the user can interact with.
We will start with writing a `create_collection` function so we can allow a user to create a collection of POAP NFTs for their event. 

At this stage we will:

1. 🧑‍💻 Setup our first publicly callable function (a "dispatchable call" in Substrate lingo).
2.  🤗 Specify the _[events and errors](https://docs.substrate.io/v3/runtime/events-and-errors/)_ our function should handle.
3. ⚪️ Create an _internal function_ to generate some randomness to create a unique collection ID.
4. ✏️ Create an _internal function_ to write newly created collection to storage.

Our Pallet has some limitations when it comes to creating a new collection...

> 💡 To provide a better user experience for users calling our functions, we can both create custom _error messages_ in case of a failure, and custom _events_ in case of a success.

1. The collection we create _must be unique_, and thus cannot have a duplicate `collection_id` as another collection.
2. The user who owns this collection cannot _own too many collection_, ensuring our storage for any account is bounded.
3. The description _cannot exceed our maximum value_ previously configured, ensuring that we our runtime does not accept too big of a description.
4. A collection must have _at lest one claimable NFT_.

If our `create_collection` call is successful, we can emit an event with information about the collection that was created, and who the owner is. These events can be used by front-end applications to trigger updates to the UX or notify users that things went successfully. 
Also, block explorers normally index all of the events by default for Substrate chains, so it will be easy to look up when these events happen.

You will also notice that we separate out most of our logic into an **internal** function. 
This allows us to expose apis in the future, which gives other parts of our runtime low level access to do things like `mint` new collections. 
You can see that this internal function does not do any kind of authorization or authentication, but that is fine, because it is an internal function, and only accessible by the runtime developer.

<!-- slide:break-40 -->

<!-- tabs:start -->

#### ** 1. SETUP **

Let's start by creating the basics of our dispatchable function.
To write it completely, we'll need to write our additional internal helper functions but, for now:

```rust
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

	// Write new collection to storage by calling helper function

	Ok(())
}
```

#### ** 2. ADD ERRORS **

Add the errors that can come up when trying to create a new collection: `DuplicateCollection`, `TooManyOwned`, `CollectionDescriptionTooBig` and `AmountMustBeGreaterThanZero`.

```rust
// Your Pallet's error messages.
#[pallet::error]
pub enum Error<T> {
	/// The collection being minted already exists.
	DuplicateCollection,
	/// Account has exceeded MaxPaopOwned.
	TooManyOwned,
	/// The description for this collection is too big.
	CollectionDescriptionTooBig,
	/// Collections can only be created with an amount greater than zero.
	AmountMustBeGreaterThanZero,
}
```

#### ** 3. ADD EVENT **

Add the `Created` event to your Pallet.

```rust
// Your Pallet's events.
#[pallet::event]
#[pallet::generate_deposit(pub(super) fn deposit_event)]
pub enum Event<T: Config> {
	/// A new collection was successfully created.
	Created { collection_id: [u8; 16], owner: T::AccountId },
}
```

#### ** 4. GEN ID **

Create an **internal** helper function which generates a new unique collection ID.

```rust
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
```

#### ** 5. MINT COLLECTION **

Create an **internal** helper function which enables "minting" the collection (_a.k.a. writing the a valid dispatchable call to storage_)🏋️.

```rust
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
```

#### ** 6. COMPLETE `create_collection` **

Finally, we can complete the actual **callable** function.

At this point, really minimal logic since we just call into our helper.

```rust
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
```

#### ** SOLUTION **

This should compile successfully by running:

```bash
cargo build -p pallet-template
```

This should compile without warnings.

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
	}

	// Your Pallet's events.
	#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)]
	pub enum Event<T: Config> {
		/// A new kitty was successfully created.
		Created { kitty: [u8; 16], owner: T::AccountId },
	}

	// Your Pallet's error messages.
	#[pallet::error]
	pub enum Error<T> {
		/// The collection being minted already exists.
		DuplicateCollection,
		/// Account has exceeded MaxPaopOwned.
		TooManyOwned,
		/// The description for this collection is too big.
		CollectionDescriptionTooBig,
		/// Collections can only be created with an amount greater than zero.
		AmountMustBeGreaterThanZero,
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
	}

	// Your Pallet's internal functions.
	impl<T: Config> Pallet<T> {
		// Generates and returns random collection ID
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
	}
}
```

<!-- tabs:end -->
