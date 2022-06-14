
# Welcome to the Substrate Workshop

> This workshop was created for the We Are Developers (Berlin, June 2022) conference and is primarily based on Substrate's [Kitties tutorial](https://docs.substrate.io/tutorials/v3/kitties/pt1/).

In this workshop, you'll learn how to build a custom Substrate blockchain that can mint **Proof of Attendance NFTs**.

#### Proof of Attendance Protocol (POAP) NFTs are like digital badges that users can claim at events they've been, and use them later (for priveldged access, or just to show them off to their peers).

## What You'll Learn

* The basic patterns for building and running a Substrate node on your local machine.
* The basics of [Rust](https://www.rust-lang.org/) as used in Substrate.
* How to write a custom FRAME Pallet which can create NFTs.
	* Create custom types.
	* Manage custom storage.
	* Write custom logic and callable functions.
* Integrate your new pallet into your Substrate runtime.

We have a lot to cover.
So let's get right to it!

<!-- slide:break -->

<img src="assets/wadwc2022-logo.png" alt="qr-code" width="500"/>

To follow along for this workshop, you'll need **follow the [setup instructions](https://docs.substrate.io/tutorials/v3/create-your-first-substrate-chain/#install-required-packages)** to:

- Set up you developer environment with Rust
- Compile a copy of [Substrate's Node Template](https://github.com/substrate-developer-hub/substrate-node-template)

**Acknowledgements:** thank you to Shawn for the original tutorial, inspired by the [crypto zombies tutorial](https://cryptozombies.io/en/lesson/1/chapter/1).