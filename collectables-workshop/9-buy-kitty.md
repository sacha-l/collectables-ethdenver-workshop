# Buy

Now that we already have a `do_transfer` helper function, adding the `buy` disptachable requires little code.

```rust
#[pallet::weight(0)]
pub fn buy_kitty(
    origin: OriginFor<T>,
    kitty_id: [u8; 16],
    bid_price: BalanceOf<T>,
) -> DispatchResult {
    // Make sure the caller is from a signed origin
    let buyer = ensure_signed(origin)?;
    // Transfer the kitty from seller to buyer as a sale.
    Self::do_transfer(kitty_id, buyer, Some(bid_price))?;

    Ok(())
}
```
