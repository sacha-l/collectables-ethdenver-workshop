# What is Substrate?

Substrate is a framework and toolkit to develop application specific blockchains.
Application logic is encapsulated by writing specialized [runtimes](https://docs.substrate.io/v3/concepts/runtime/), which really is the executable that nodes use to run the blockchain logic of the network.

Substrate is built in Rust and uses WebAssembly for some key defining features.
WebAssembly compatibility enables provability, verification and upgradability of runtimes, useful for on-chain governance and multi-chain consensus, as well as ensuring that blockchains built with it can be platform agnostic.

Substrate provides a way to easily write runtime logic using pallets written with [FRAME](https://docs.substrate.io/v3/runtime/frame/) &mdash; Substrate's opiniated toolkit for writing runtime logic.
You can think of comparing the runtime of a Substrate blockchain to a crate, carrying all of its business logic in a multitude of different pallets.

In this workshop, we'll focus on writing a pallet for a blockchain specialized in managing the decentralized creation and ownership of crypto Kitties.

<!-- slide:break-40 -->

![pallets](../assets/frame-pallets.png)
