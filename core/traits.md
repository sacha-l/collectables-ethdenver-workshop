# Rust Traits

Traits tell the Rust compiler what kinds of functionality a type has.
Traits are similar to a feature often called interfaces in other languages, although with some differences.

You can imagine manually implementing a simple trait like `PartialEq` for the types we described before:

```rust
enum Direction {
	Up,
	Down,
	Left,
	Right,
}

impl PartialEq for Direction {
    fn eq(&self, other: &Self) -> bool {
        self == other
    }
}
```

With this trait implemented, we can do things like:

```rust
let direction_1 = Direction::Left;
let direction_2 = Direction::Right;
let direction_3 = Direction::Left;

assert!(direction_1 != direction_2);
assert!(direction_1 == direction_3);
```

<!-- slide:break -->

# Advance Traits

You can also define custom traits, and associate things like type definitions, which themselves have traits.

Every Pallet in Substrate has a `trait Config`, which allows you to define the types and interfaces that your Pallet depends on.

This trait always inherits from the `frame_system::Config` which defines all the core types in your runtime, like:

* `AccountId`: The type representing a user address.
* `BlockNumber`: The type representing a block number.
* etc...

These traits allows your runtime to be **generic**, which means that you can easily change the fundamental types in your blockchain, and things will still work.


Here is a hint as to what you might see later in the tutorial, but practically speaking, this is a pretty advance concept, so mostly take it for granted.

```rust
#[pallet::config]
pub trait Config: frame_system::Config {
	// The type used across your runtime for emitting events.
	type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;

	// The interface used to manage user balances from this pallet.
	type Currency: Currency<Self::AccountId>;

	// A configurable max value,
	#[pallet::constant]
	type MaxValue: Get<u32>;

	// The interface used to get random values to use in this pallet.
	type RandomnessProvider: Randomness<Self::Hash, Self::BlockNumber>;
}
```
