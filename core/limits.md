# Blockchain Limits

All blockchains, even those built with Substrate, fundamentally have limits:

* Each block can only take so long to produce.
* Each block can only be so big.
* Each block can only grow your state by so much.

When doing contract development on a platform like Ethereum, the EVM acts as a limiter for what kind of logic you can run.

Within the EVM, you are bounded by the **Gas Limit**, which means that any individual transaction can only execute so much logic and read or write so much storage.


<!-- slide:break -->

# Runtime Limits

Substrate gives you full access to write code as you like. However, with great power comes great responsibility.

Rather than have something like the EVM meter your every line of code, Substrate allows you to benchmark your own code ahead of time, which is much more efficient.

Benchmarking is beyond the scope of this workshop, but you should note that Substrate provides and expects you to use certain tools to ensure your runtime is always **bounded**.

One example is the `BoundedVec` type, which is like a standard `Vec`, but has a limit to the number of elements which can be in the vector.

This ensures that there is an upper limit to the amount of data that can exist inside of that vector, which will ensure that your runtime code always has some upper limit.

```rust
type MaxLimit = ConstU32<100>;

struct Item {
	field1: bool,
	field2: [u8; 16],
}

// This is vector which can hold at most 100 items.
let mut my_items = BoundedVec::<Item, MaxLimit>::default();
```

Since this vector is bounded, we cannot always guarantee that we can add a new item, but we can easily handle those errors if they were to occur:

```rust
// This is how we can try to add a new item to the vector
my_item.try_push(new_item).map_err(|()| "this vector is full!")?;
```
