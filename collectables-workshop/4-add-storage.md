# Add storage

Similar to the ERC721 standard, our pallet will have the following storage items:

* `CountForKitty`: Keeps track of the number of kitties in existence.
* `Kitty`: Maps the kitty struct to the kitty DNA.
* `KittiesOwned`: Track kitties owned by account.

<!-- slide:break-40 -->

<!-- tabs:start -->

#### ** CountForKitty **

```rust
	#[pallet::storage]
	#[pallet::getter(fn kitty_count)]
	pub(super) type CountForKitty<T: Config> = StorageValue<_, u64, ValueQuery>;
```

#### ** Kitties **

```rust
	#[pallet::storage]
	#[pallet::getter(fn kitty_count)]
	pub(super) type CountForKitty<T: Config> = StorageValue<_, u64, ValueQuery>;
```

#### ** KittiesOwned **

```rust
	#[pallet::storage]
	#[pallet::getter(fn kitties_owned)]
	pub(super) type KittiesOwned<T: Config> = StorageMap<
		_,
		Twox64Concat,
		T::AccountId,
		BoundedVec<[u8; 16], T::MaxKittyOwned>,
		ValueQuery,
	>;
```

<!-- tabs:end -->