# Transfer

The structure of our transfer function will be very similar to how we wrote `create_kitty`.
We'll have a dispatchable, i.e. the function that any account can call and a helper function that's private to our pallet.

Remember to write your dispatchables under the `#[pallet::call] impl<T: Config> Pallet<T> {}` declaration and your helper functions under `impl<T: Config> Pallet<T> {}`.

`transfer` is a dispatchable that checks the caller of the function is a signed origin, checks the kitty to be transferred exists and is owned by the caller and calls `do_transfer`.

`do_transfer` is a private helper function that returns some DNA and a Gender type.

# Steps
1. Write `transfer`.
It takes an `origin`, `T::AccountId` and `[u8; 16]` and returns [`DispatchResult`](https://docs.substrate.io/rustdocs/latest/frame_support/dispatch/type.DispatchResult.html).
Use `ensure_signed` to verify that the caller is from a signed origin and `ensure!` to check that the caller owns this kitty.
1. Write `do_transfer`.
1. Write relevant events and errors.

<!-- slide:break-40 -->

<!-- tabs:start -->
#### ** 1. Transfer kitty **

```rust
#[pallet::weight(0)]
pub fn transfer(
    origin: OriginFor<T>,
    to: T::AccountId,
    kitty_id: [u8; 16],
) -> DispatchResult {
    // Make sure the caller is from a signed origin
    let from = ensure_signed(origin)?;
    let kitty = Kitties::<T>::get(&kitty_id).ok_or(Error::<T>::NoKitty)?;
    ensure!(kitty.owner == from, Error::<T>::NotOwner);
    Self::do_transfer(kitty_id, to, None)?;
    Ok(())
}
```
#### ** 2. do_transfer **

We'll break up the `do_transfer` function into several parts to make it easier to follow.
To get started, we need to perform the following checks:
* Does this kitty really exist?
* Is destination the owner?

```rust
// Update storage to transfer kitty
pub fn do_transfer(
    kitty_id: [u8; 16],
    to: T::AccountId,
    maybe_bid_price: Option<BalanceOf<T>>,
) -> DispatchResult {
    // Does this kitty really exist?
    // Is destination the owner?
}
```
#### ** 3. Continue do_transfer **

Continuing, we need to:
* Get the kitty of the function caller
* Remove the kitty from the list of owned kitties
* Add the kitty to the list of owned kitties

```rust
let mut from_owned = KittiesOwned::<T>::get(&from);

// Remove kitty from list of owned kitties.
if let Some(ind) = from_owned.iter().position(|&id| id == kitty_id) {
    from_owned.swap_remove(ind);
} else {
    return Err(Error::<T>::NoKitty.into())
}

// Add kitty to the list of owned kitties.
let mut to_owned = KittiesOwned::<T>::get(&to);
to_owned.try_push(kitty_id).map_err(|()| Error::<T>::TooManyOwned)?;
```

#### ** 4. More do_transfer **

At this point, we can initiate the transfer by updating the balances of each account:
* We need to make sure the buying price is at least the selling price
* We need to handle the balance transfer for that price
* We'll use the `transfer` method from the `Currency` trait from `frame_system` to reflect a change in account balances

```rust
// Mutating state here via a balance transfer, so nothing is allowed to fail after this.
if let Some(bid_price) = maybe_bid_price {
    if let Some(price) = kitty.price {
        ensure!(bid_price >= price, Error::<T>::BidPriceTooLow);
        // Transfer the amount from buyer to seller
        T::Currency::transfer(&to, &from, price, ExistenceRequirement::KeepAlive)?;
        // Deposit sold event
        Self::deposit_event(Event::Sold {
            seller: from.clone(),
            buyer: to.clone(),
            kitty: kitty_id,
            price,
        });
    } else {
        return Err(Error::<T>::NotForSale.into())
    }
}
```
#### ** 4. Write to storage **

Finallly, we can update all storage items, reset the price to `None` and deposit a `Transferred` event.

```rust
// Transfer succeeded, update the kitty owner and reset the price to `None`.
kitty.owner = to.clone();
kitty.price = None;

// Write updates to storage
Kitties::<T>::insert(&kitty_id, kitty);
KittiesOwned::<T>::insert(&to, to_owned);
KittiesOwned::<T>::insert(&from, from_owned);

Self::deposit_event(Event::Transferred { from, to, kitty: kitty_id });

Ok(())
```
#### ** 5. Events and errors **

There are some events and errors specific to our `do_transfer` function we still need to declare: 

* `Error::<T>::TransferToSelf`: an owner can't transfer the kitty to themself
* `Error::<T>::BidPriceTooLow`: the buyer's price is too low
* `Error::<T>::NotForSale`: the kitty is not for sale

```rust
// Errors
#[pallet::error]
pub enum Error<T> {
// -- snip --
    /// Trying to transfer or buy a kitty from oneself.
    TransferToSelf,
    /// This kitty is not for sale.
    NotForSale,
    /// Ensures that the buying price is greater than the asking price.
    BidPriceTooLow,
}
```

* `Event::Transferred`: indicates that a kitty has successfully been transferred

```rust
#[pallet::event]
#[pallet::generate_deposit(pub(super) fn deposit_event)]
pub enum Event<T: Config> {
// -- snip --
    /// A kitty was successfully transferred.
	Transferred { from: T::AccountId, to: T::AccountId, kitty: [u8; 16] },
}
```

#### ** Solution **

Here's the full `do_transfer` function:

```rust
// Update storage to transfer kitty
pub fn do_transfer(
    kitty_id: [u8; 16],
    to: T::AccountId,
    maybe_bid_price: Option<BalanceOf<T>>,
) -> DispatchResult {
    // Get the kitty
    let mut kitty = Kitties::<T>::get(&kitty_id).ok_or(Error::<T>::NoKitty)?;
    let from = kitty.owner;

    ensure!(from != to, Error::<T>::TransferToSelf);
    let mut from_owned = KittiesOwned::<T>::get(&from);

    // Remove kitty from list of owned kitties.
    if let Some(ind) = from_owned.iter().position(|&id| id == kitty_id) {
        from_owned.swap_remove(ind);
    } else {
        return Err(Error::<T>::NoKitty.into())
    }

    // Add kitty to the list of owned kitties.
    let mut to_owned = KittiesOwned::<T>::get(&to);
    to_owned.try_push(kitty_id).map_err(|()| Error::<T>::TooManyOwned)?;

    // Mutating state here via a balance transfer, so nothing is allowed to fail after this.
    if let Some(bid_price) = maybe_bid_price {
        if let Some(price) = kitty.price {
            ensure!(bid_price >= price, Error::<T>::BidPriceTooLow);
            // Transfer the amount from buyer to seller
            T::Currency::transfer(&to, &from, price, ExistenceRequirement::KeepAlive)?;
            // Deposit sold event
            Self::deposit_event(Event::Sold {
                seller: from.clone(),
                buyer: to.clone(),
                kitty: kitty_id,
                price,
            });
        } else {
            return Err(Error::<T>::NotForSale.into())
        }
    }

    // Transfer succeeded, update the kitty owner and reset the price to `None`.
    kitty.owner = to.clone();
    kitty.price = None;

    // Write updates to storage
    Kitties::<T>::insert(&kitty_id, kitty);
    KittiesOwned::<T>::insert(&to, to_owned);
    KittiesOwned::<T>::insert(&from, from_owned);

    Self::deposit_event(Event::Transferred { from, to, kitty: kitty_id });

    Ok(())
}
```
<!-- tabs:end -->