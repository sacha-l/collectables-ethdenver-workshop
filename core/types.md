# Rust Types

A common part of any development in Rust is defining and using custom types.

## Enums

An `enum` allows you to create a type where can enumerate the possible variants.

```rust
pub enum Direction {
	Up,
	Down,
	Left,
	Right,
}
```

## Structs

You can also define a `struct`, which allows you to package together various different types together into a single object.
You can even use custom types like the `enum` defined above.

```rust
pub struct MyObject {
	pub field1: [u8; 16],
	pub field2: Option<u32>,
	pub field3: Direction,
}
```

<!-- slide:break -->

#

## Derive Attributes

Normally, you would need to manually implement various traits for your custom types, but sometimes the rust compiler can do this for you!

When working with types inside of Substrate, we expect that they are able to satisfy a number of traits:

* `Encode`: How to serialize a custom type into bytes.
* `Decode`: How to deserialize bytes into a custom type.
* `TypeInfo`: Provides useful information to the Runtime Metadata, improving UX.
* `MaxEncodedLen`: Ensures that types in the runtime are bounded in size.
* `RuntimeDebug`: Allow a type to be printed to the console; useful for tests.

Additionally, some other traits can be helpful for basic logic:

* `Eq` & `PartialEq`: Check that two objects are equal.
* `Clone` & `Copy`: Allow an object to be duplicated.

You can add all of these traits super easy using the `#[derive]` macro!

```rust
#[derive(Encode, Decode, TypeInfo, MaxEncodedLen, RuntimeDebug, Eq, PartialEq, Clone, Copy)]
pub struct MyObject {
	pub field1: [u8; 16],
	pub field2: Option<u32>,
	pub field3: Direction,
}
```
