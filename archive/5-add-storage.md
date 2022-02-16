# Add storage

Our pallet will have the following storage items:

* `CountForKitty`: Keeps track of the number of kitties in existence.
* `Kitty`: Maps the kitty struct to the kitty DNA.
* `KittiesOwned`: Track kitties owned by account.

⚠️ These items will persist in your blockchain's state, meaning that querying or writing to them is expensive and it's your job as a runtime engineer to write checks to ensure the changes in storage are allowed.

`frame_support` provides us with a [`storage` utlility module](https://docs.substrate.io/rustdocs/latest/frame_support/storage/index.html) that helps use various traits to instantiate different types of storage and use their APIs.
We'll be using two types of storage structures: 
- [`StorageValue`](https://docs.substrate.io/rustdocs/latest/frame_support/storage/types/struct.StorageValue.html)
- [`StorageMap`](https://docs.substrate.io/rustdocs/latest/frame_support/storage/types/struct.StorageMap.html)

<!-- slide:break-40 -->

<!-- tabs:start -->

#### ** 1. CountForKitty **

```rust
#[pallet::storage]
pub(super) type CountForKitty<T: Config> = StorageValue<_, u64, ValueQuery>;
```

#### ** 2. Kitties **

```rust
#[pallet::storage]
pub(super) type Kitties<T: Config> = StorageMap<_, Twox64Concat, [u8; 16], Kitty<T>>;
```

#### ** 3. KittiesOwned **

```rust
	#[pallet::storage]
	pub(super) type KittiesOwned<T: Config> = StorageMap<
		_,
		Twox64Concat,
		T::AccountId,
		BoundedVec<[u8; 16], T::MaxKittyOwned>,
		ValueQuery,
	>;
```

#### ** 4. Add pallet constant **

Add these two lines to your pallet's `Config` trait:

```rust
#[pallet::constant]
type MaxKittiesOwned: Get<u32>;
```

<!-- tabs:end -->


1. Write the `CountForKitty` storage item.
It will be a `StorageValue` that keeps track of a `u64`. 
1. Write the `Kitties` storage item.
This will be a storage map that maps from a kitty's DNA `[u8; 16]` and your kitty struct.
1. Write the `KittiesOwned` struct. 
This will be a storage map from `T::AccountId` to a vector of kitty DNA `[u8; 16]`.
This vector needs to be bounded and we need to keep track of the maximum amount an account can hold in storage.
1. Update your configuration trait to include `MaxKittiesOwned`.
