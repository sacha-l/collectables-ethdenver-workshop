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

Storage API:
- [StorageMap](https://docs.substrate.io/rustdocs/latest/frame_support/storage/types/struct.StorageMap.html): `contains_key`
- ..

## Steps

1. Write `create_kitty` 
1. Write `mint` 
```rust
// Check if the kitty does not already exist in our storage map
ensure!(Kitties::<T>::contains_key(&kitty.dna), Error::<T>::NonExistantKitty);
```

<!-- slide:break-40 -->

<!-- tabs:start -->

#### ** Hide Hints **

Click the other tabs to view hints.

#### ** Hint: Input **

```rust
#[pallet::weight(0)]
pub fn create_kitty(origin: OriginFor<T>) -> DispatchResult {
    let sender = ensure_signed(origin)?; // <- add this line
    let kitty_id = Self::mint(&sender, None, None)?; // <- add this line
    // Logging to the console
    log::info!("A kitty is born with ID: {:?}.", kitty_id); // <- add this line

    // ACTION #4: Deposit `Created` event

    Ok(())
}
```

#### ** Hint: Check + Error **

> **Note:** You need to import:
> * `system::ensure_signed`
> * `frame_support::ensure`

```rust
```

#### ** Hint: Logic **

```rust
// Call the `system` runtime module to get the current block number
let current_block = <system::Module<T>>::block_number();

// Store the proof with the sender and the current block number
Proofs::<T>::insert(&proof, (sender.clone(), current_block));
```

#### ** Hint: Emit Event **

```rust

```

#### ** Solution **

```rust

```

<!-- tabs:end -->
