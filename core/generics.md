# Rust Generics

Generics is one of the most common first challenges for new Substrate developers.

The whole point of generics is to keep your code agnostic of specific types or implementation details, and thus allows you to take advantage of the full flexibility, modularity, and extensibility of Substrate.

What does it mean practically?

Rather than defining a hardcoded type and using it like:

```rust
type AccountId = [u8; 32];
```

You can keep your types and code generic, and only define them at a later point.

```rust
// `frame_system` has something which looks similar to this...
trait Config {
	type AccountId;
	type BlockNumber;
	// etc...
}
```

Then when writing code, you can use these generic types:

```rust
#[pallet::pallet]
pub struct Pallet<T>(_);

// Note that we say `T` has trait `Config`, and thus has access to `AccountId`.
// `T` is a common choice for generics in Rust, but the letter is arbitrary.
impl<T: Config> Pallet<T> {
	fn my_function(user: T::AccountId) {
		// do something...
	}
}
```

<!-- slide:break -->

# Why?

Why introduce this complexity to Substrate development?

![Rust Crab Strong](../collectables-workshop/assets/rust-crab-strong.png)

Imagine the flexibility of writing code that makes no assumptions about the core blockchain architecture decisions you have made.

For example, in Substrate, most addresses are 32-byte public keys (`[u8; 32]`), but what if you wanted to make an Ethereum compatible chain? If you hardcoded types into your code, Rust (being type safe) would only ever let you use 32-byte addresses.

However, with this generic framework, you can actually write Pallets which support both 32-byte Substrate addresses or 20-byte Ethereum addresses!

You could have a blockchain where the block number is a `u32`, or where the block number could be a `u64`!

As you become more advance in your Substrate development, this "annoying" feature will actually allow you to write extremely modular and upgradable code. But for now, just note that Rust will ask you to specify generics all over the place, and that is what the `T::` or `<T>` thing does.
