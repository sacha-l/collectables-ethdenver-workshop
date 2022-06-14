# Runtime Storage

> 💪 Blockchains are all about storing data that you want to come to consensus about.

If you are familiar with contract development, the storage types below should be very familiar to you:

* `StorageValue`: Storing a single type in storage.
* `StorageMap`: Storing a map from key to value in storage.

Defining new storage items is easy, and looks like this:

```rust
/// Store a single `u64` in the blockchain storage.
#[pallet::storage]
pub(super) type CountForItems<T: Config> = StorageValue<_, u64, ValueQuery>;

/// Store a map from user (`T::AccountId`) to a custom type `Item`.
#[pallet::storage]
pub(super) type Items<T: Config> = StorageMap<_, Twox64Concat, T::AccountId, Item>;
```

Substrate has even more complex storage items like `StorageDoubleMap`, `StorageCountedMap`, and more!
But we won't use those in this workshop.

<!-- slide:break -->

# Storage APIs

Once you have defined these storage items, it is pretty easy to add, read, or remove values from storage.

**Manipulating a `StorageValue`**

```rust
// Put a value in storage.
CountForItems::<T>::put(10);
// Get a value from storage.
let ten = CountForItems::<T>::get();
// Kill a value in storage.
CountForItems::<T>::kill();
```

**Manipulating a `StorageMap`**

```rust
// Check if a value exists in storage. This will currently result in `false`.
let is_false = Items::<T>::contains_key(user);
// Put a value in storage.
Items::<T>::insert(user, new_item);
// Get a value from storage.
let my_item = Items::<T>::get(user);
// Kill a value in storage.
Items::<T>::remove(user);
```
