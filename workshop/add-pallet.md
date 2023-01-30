# Add Pallet to Runtime

Now that we have completed our pallet development, we need to enable this pallet in our runtime.

We will need to navigate to and modify a different file than the one we have been working with till now:

```
./substrate-node-template/runtime/src/lib.rs
```

The `pallet-template` was already added to this runtime, but we have made some significant updates throughout this workshop, so we need to update the configuration.

Follow the 2 update steps outlined on this page.

If everything was successful, you should be able to successfully compile your blockchain node:

```bash
cargo build --release
```

# 🎉 CONGRATS 🎉

You have successfully made your first custom Substrate blockchain!

Let's try it out next!

<!-- slide:break -->

1. Update the `pallet_template::Config` to match the following:

```rust
impl pallet_template::Config for Runtime {
	type RuntimeEvent = RuntimeEvent;
	type Currency = Balances;
	type KittyRandomness = RandomnessCollectiveFlip;
	type MaxKittiesOwned = frame_support::pallet_prelude::ConstU32<100>;
}
```

2. Finally, update the naming of the pallet in the `construct_runtime!` macro to `SubstrateKitties`:

```rust
// Create the runtime by composing the FRAME pallets that were previously configured.
construct_runtime!(
	pub enum Runtime where
		Block = Block,
		NodeBlock = opaque::Block,
		UncheckedExtrinsic = UncheckedExtrinsic
	{
		System: frame_system,
		RandomnessCollectiveFlip: pallet_randomness_collective_flip,
		Timestamp: pallet_timestamp,
		Aura: pallet_aura,
		Grandpa: pallet_grandpa,
		Balances: pallet_balances,
		TransactionPayment: pallet_transaction_payment,
		Sudo: pallet_sudo,
		SubstrateKitties: pallet_template::{Pallet, Call, Storage, Event<T>}, // <-- Update name here
	}
);
```
