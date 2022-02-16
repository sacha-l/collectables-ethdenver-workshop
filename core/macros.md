# Rust Macros

Rust provides an amazing powerful tool called **macros** which allows you to automatically generate code for developers! This means, that where you would normally need to write hundreds (or thousands) of lines of code, Rust can do it for you automatically!

However, on the downside, macros can seem like magic to developers, and if you don't understand exactly what they do, they may obfuscate important parts of runtime development.

In general, macros are there for you to make building a Pallet way easier.

You should not spend your early days in Substrate trying to understand all of them, but rather, just use them to make development faster. As you learn more and more about Substrate, you can then start to investigate exactly what these macros do for you, and really come to appreciate how awesomely powerful they are.

We mainly use two kinds of macros:

* Attribute Macros: `#[thingy]`
* Function-like Macros: `thingy!(...)`

If you ever see an `#` or an `!` like above, you are probably working with a macro, and there is probably some magic happening in the background.

A function-like macro we use in Substrate a lot is `ensure!(condition, error)`, which expands to the following code:

```rust
if !condition {
	return Err(error.into())
}
```

So rather than write those 3 lines, you can just write that one line.

<!-- slide:break -->

# Common Pallet Macros

```rust
#![cfg_attr(not(feature = "std"), no_std)]
#[frame_support::pallet]
pub mod pallet {
	use frame_support::pallet_prelude::*;
	use frame_system::pallet_prelude::*;

	#[pallet::config]
	pub trait Config: frame_system::Config {}

	#[pallet::pallet]
	pub struct Pallet<T>(_);

	#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)]
	pub enum Event<T: Config> {}

	#[pallet::error]
	pub enum Error<T> {}

	#[pallet::call]
	impl<T: Config> Pallet<T> {}
}
```

You can see that each "section" in the pallet uses a macro, which does special things for that section. This literally expands to hundreds of lines of code, providing things like:

* Automatic metadata generation.
* Implementing various traits.
* Creating functions for you to access.
* and much more!
