# Create a Kitty

Every dispatchable declared in a pallet must contain an associated weight for the runtime to know what the worst case execution time is.
Usually, when writing a pallet you would perform benchmarks to correctly determine what the worst case values are. 
But for this workshop, we'll set all weights to 0 because our goal isn't to write production ready code. 

We'll write our disptachable function under the `#[pallet::call] impl<T: Config> Pallet<T> {}` declaration and all helper functions under `impl<T: Config> Pallet<T> {}`.

`create_kitty` is a dispatchable that checks the caller of the function is a signed origin, generates unique DNA and calls a minting function.

`gen_dna` is a private helper function that returns some DNA and a Gender type.

`mint` is a private helper function that checks that the kitty doesn't already exist updates storage items and deposits an event.

# Steps
1. Write `create_kitty`.
Use `ensure_signed` to verify that the caller is froma  signed origin.
It returns [`DispatchResult`](https://docs.substrate.io/rustdocs/latest/frame_support/dispatch/type.DispatchResult.html).
1. Write `gen_dna`.
This just needs randomly generate a `[u8; 16]` and `Gender` and return a tuple of (`[u8; 16]`, `Gender`).
1. For randomness, we'll need to add a special type to our pallet's configuration trait. 
Add this line in `pub trait Config: frame_system::Config {}`
```rust
type KittyRandomness: Randomness<Self::Hash, Self::BlockNumber>;
```
1. Write the minting function
1. Handle events and errors.

<!-- slide:break-40 -->

<!-- tabs:start -->
#### ** 1. Create kitty **

The publicly callable `create_kitty` function will only check the origin of the caller and call the `mint` function:

```rust
#[pallet::weight(0)]
pub fn create_kitty(origin: OriginFor<T>) -> DispatchResult {
    let sender = ensure_signed(origin)?; 
    let kitty_id = Self::mint(&sender, None, None)?; 
    Ok(())
}
```
#### ** 2. Generate DNA helper **

```rust
// Generates and returns DNA and Gender
fn gen_dna() -> ([u8; 16], Gender) {
    // Create randomness
    let random = T::KittyRandomness::random(&b"dna"[..]).0;

    // Multiple unique kitties can be generated in the same block
    let unique_payload = (
        random,
        frame_system::Pallet::<T>::extrinsic_index().unwrap_or_default(),
        frame_system::Pallet::<T>::block_number(),
    );

    // Turns into a byte array
    let encoded_payload = unique_payload.encode();
    let hash = blake2_128(&encoded_payload);

    // Generate Gender
    if hash[0] % 2 == 0 {
        (hash, Gender::Male)
    } else {
        (hash, Gender::Female)
    }
}
```

📝 Don't forget to add the `KittyRandomness` type to your configuration trait.

#### ** 3. Minting helper **

Using the following methods from our storage API: 
* [`contains_key`](https://docs.substrate.io/rustdocs/latest/frame_support/storage/trait.StorageMap.html#tymethod.contains_key)
* [`try_append`](https://docs.substrate.io/rustdocs/latest/frame_support/storage/types/struct.StorageMap.html#method.try_append)
* [`get`](https://docs.substrate.io/rustdocs/latest/frame_support/storage/trait.StorageMap.html#tymethod.get)
* [`insert`](https://docs.substrate.io/rustdocs/latest/frame_support/storage/trait.StorageMap.html#tymethod.insert)
* [`put`](https://docs.substrate.io/rustdocs/latest/frame_support/storage/trait.StorageValue.html#tymethod.put)


```rust
// Helper to mint a kitty
pub fn mint(
    owner: &T::AccountId,
    dna: [u8; 16],
    gender: Gender,
) -> Result<[u8; 16], DispatchError> {
    // Create a new object
    let kitty = Kitty::<T> { dna, price: None, gender, owner: owner.clone() };

    // Check if the kitty does not already exist in our storage map
    ensure!(!Kitties::<T>::contains_key(&kitty.dna), Error::<T>::NoKitty);

    // Performs this operation first as it may fail
    let count = CountForKitties::<T>::get();
    let new_count = count.checked_add(1).ok_or(ArithmeticError::Overflow)?;

    // Append kitty to KittiesOwned
    KittiesOwned::<T>::try_append(&owner, kitty.dna)
        .map_err(|_| Error::<T>::TooManyOwned)?;

    // Write new kitty to storage
    Kitties::<T>::insert(kitty.dna, kitty);
    CountForKitties::<T>::put(new_count);

    // Deposit our "Created" event.
    Self::deposit_event(Event::Created { kitty: dna, owner: owner.clone() });

    // Returns the DNA of the new kitty if this succeeds
    Ok(dna)
}
```

#### ** 4. Events and errors **

In the minting function, we used different `Errors` and `Events`. 
These will be emitted once they are reached, to indicate whether an update to storage suceeded or if some checks failed.
We have:
* `Error::<T>::NoKitty`: handles the case when the kitty doesn't exist
* `Error::<T>::TooManyOwned`: handles the case when the owner has exceed the maximum amount of kitties allowed
* `ArithmeticError::Overflow`: handles overflow when incrementing the amount of kitties by 1

```rust
// Errors
#[pallet::error]
pub enum Error<T> {
    /// An account may only own `MaxKittiesOwned` kitties.
    TooManyOwned,
    /// This kitty does not exist!
    NoKitty,
}
```

* `Event::Created`: indicates a kitty has been created successfully

```rust
#[pallet::event]
#[pallet::generate_deposit(pub(super) fn deposit_event)]
pub enum Event<T: Config> {
    /// A new kitty was successfully created.
    Created { kitty: [u8; 16], owner: T::AccountId },
}
```

<!-- tabs:end -->