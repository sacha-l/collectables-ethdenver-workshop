# Runtime and genesis config

We've written a pallet that compiles without errors alone, but now its time to link it up to our runtime.
To do this we need to:
1. Implement our pallet for our runtime and make our configuration types concrete.
1. Set up the genesis configuration for our pallet.

#### **Update your runtime configuration**

Implement your pallet for the runtime:

```rust
impl pallet_template::Config for Runtime {
	type Event = Event;
	type Currency = Balances;
	type KittyRandomness = RandomnessCollectiveFlip;
	type MaxKittiesOwned = ConstU32<100>;
}
```

Name your pallet for your runtime:

```rust
construct_runtime!(
    // -- snip --
    SubstrateKitties: pallet_template
    // -- snip --
```

Given that we're writing a pallet that will be part of our blockchain's runtime, before we compile and launch it, we need to configure what will be in our storage items at genesis.
Or at least, we need to tell our runtime that there isn't anything in our pallet's storage item at genesis.

To configure the genesis of our pallet:

1. Write a `GenesisConfig` struct with the `#[pallet::genesis_config]` attribute
1. Implement `Default` for the struct
1. Build the genesis by implementing `GenesisBuild` for the `GenesisConfig` struct
1. Update the `node/src/chain_spec.rs` file


<!-- slide:break-40 -->

#### **Configure genesis for your pallet**

Add the genesis configuration in `pallets/template/src/lib.rs`:

```rust
// Our pallet's genesis configuration
#[pallet::genesis_config]
pub struct GenesisConfig<T: Config> {
    pub kitties: Vec<(T::AccountId, [u8; 16], Gender)>,
}

// Required to implement default for GenesisConfig
#[cfg(feature = "std")]
impl<T: Config> Default for GenesisConfig<T> {
    fn default() -> GenesisConfig<T> {
        GenesisConfig { kitties: vec![] }
    }
}

#[pallet::genesis_build]
impl<T: Config> GenesisBuild<T> for GenesisConfig<T> {
    fn build(&self) {
        // When building a kitty from genesis config, we require the DNA and Gender to be
        // supplied
        for (account, dna, gender) in &self.kitties {
            assert!(Pallet::<T>::mint(account, *dna, *gender).is_ok());
        }
    }
}
```

#### **Update your chain spec**

Navigate to the `node/src/chan_spec.rs` file.
This holds all of the information about the chain including genesis configuration.

Add this line:

```rust
substrate_kitties: SubstrateKittiesConfig { kitties: vec![] },
```

<!-- tabs:end -->
