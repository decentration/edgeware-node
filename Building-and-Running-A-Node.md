
## Fresh start
If your device is clean (such as a fresh cloud VM) you can use this script, otherwise, proceed with the *Initial Setup*.
```
./setup.sh
```
Then proceed to the *Running* instructions or follow the instructions below for the manual setup.

### Initial Setup

```
curl https://sh.rustup.rs -sSf | sh
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
rustup update stable
cargo install --git https://github.com/alexcrichton/wasm-gc
```

You will also need to install the following packages:

Linux:
```
sudo apt install cmake pkg-config libssl-dev git clang libclang-dev
```

Mac:
```
brew install cmake pkg-config openssl git llvm
```

### Building

```
./build.sh
cargo build --release
```

### Running

Ensure you have a fresh start if updating from another version:
```
./purge-chain.sh
```
To start up the Edgeware node and connect to the latest testnet, run:
```
./target/release/edgeware --chain=edgeware --name <INSERT_NAME>
```

If you use the `--key` flag, ensure that either it is a 32-byte hex string or prefixed with `//` like so:
```
./target/release/edgeware --chain=edgeware --name <INSERT_NAME> --key //testkey
```

### Visualization

To ensure you followed the steps correctly, check https://telemetry.polkadot.io/#/Edgeware%20Testnet%20V0.2.0. If done correctly, you should see your node with the inserted name.