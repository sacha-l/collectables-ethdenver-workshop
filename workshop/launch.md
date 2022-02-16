# Launch

Now its time to launch your chain and interact with it. 

1. Build your chain:
```bash
cargo build --release
```
1. Launch it:
```bash
./target/release/node-template --dev
```
1. Open up [Polkadot JS Apps](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/explorer), connect to your local node under "Development".
Then head to "Developer -> Extrinsics" and submit a transaction using the `SubstrateKitties` pallet.

Congratulations for completing this workshop with us! 🥳
