# Create a Kitty

`create_kitty` is a dispatchable function or extrinsic that:

- checks the origin is signed
- generates a random hash with the signing account
- creates a new Kitty object using the random hash
- calls a private mint() function

`mint` is a private helper function that:

- checks that the Kitty doesn't already exist
- updates storage with the new Kitty ID (for all Kitties and for the owner's account)
- updates the new total Kitty count for storage and the new owner's account
- deposits an Event to signal that a Kitty has successfully been created

Check out the storage APIs we'll be using: [StorageMap](https://docs.substrate.io/rustdocs/latest/frame_support/storage/types/struct.StorageMap.html): `contains_key`


## Steps

1. Write `create_kitty` 
1. Write `mint` 
1. Write `gen_dna`

```rust
// Check if the kitty does not already exist in our storage map
ensure!(Kitties::<T>::contains_key(&kitty.dna), Error::<T>::NonExistantKitty);
```
1. Write events
1. Write errors 

<!-- slide:break-40 -->

<!-- tabs:start -->

#### ** create_kitty **

```rust
#[pallet::weight(0)]
pub fn create_kitty(origin: OriginFor<T>) -> DispatchResult {
    let sender = ensure_signed(origin)?; 
    let kitty_id = Self::mint(&sender, None, None)?; 
    Ok(())
}
```

#### ** Minting function **

```rust
// Helper to mint a kitty
pub fn mint(
    owner: &T::AccountId,
    dna: [u8; 16],
    gender: Gender,
) -> Result<[u8; 16], Error<T>> {
    
    // Create a new object
    let kitty = Kitty::<T> {
        dna: dna.clone(),
        price: None,
        gender: gender.clone(),
        owner: owner.clone(),
    };

    // Check if the kitty does not already exist in our storage map
    ensure!(Kitties::<T>::contains_key(&kitty.dna), Error::<T>::NonExistantKitty);

    // Performs this operation first as it may fail
    let new_count = Self::kitty_count().checked_add(1).ok_or(Error::<T>::CountForKittyOverflow)?;
    
    // Append kitty to KittiesOwned
    KittiesOwned::<T>::try_append(&owner, kitty.dna)
        .map_err(|_| Error::<T>::ExceedMaxKittyOwned)?;

    // Write new kitty to storage
    Kitties::<T>::insert(kitty.dna, kitty);
    CountForKitty::<T>::put(new_count);
    
    // Returns the DNA of the new kitty if this suceeds
    Ok(dna)
}
```

#### ** Events **

```rust
```

#### ** Errors **

```rust
```

<!-- tabs:end -->
