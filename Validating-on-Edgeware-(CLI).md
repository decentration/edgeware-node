Welcome to the official, in-depth Edgeware guide to validating. We're happy that you're interested in validating on Edgeware and we'll do our best to provide in-depth documentation on the process below. As always, reach out on [Discord](https://discord.gg/CJRfb3) or [Telegram](https://t.me/heyedgeware) if you have questions about the project.

This document contains all the information one should need to start validating on Edgeware using the **command line interface**. We will start with how to setup one's node and proceed to how to key management and monitoring. To start, we will use the following terminology of keys for the guide:

- ***stash*** - the stash keypair is where your funds should be located
- ***controller*** - the controller is the keypair that will control your validator settings
- ***session*** - the 3 session keypairs will be responsible for validating, authoring, and finalizing blocks.

## Requirements
1. You will need 5 keypairs: a `stash` (ed25519 or sr25519), `controller` (ed25519 or sr25519), and 3 `session` (only ed25519) keypairs. You can generate these using the `subkey` utility. We will be using derived keys in the examples, if you do not use derived keys, simply input the seed/mnemonic needed to sign from these accounts.
2. You will need at least the existential balance (1,000,000,000,000,000 token units i.e 0.0001 EDG) in both the `stash` and `controller` accounts plus the balances needed to send transactions from these accounts.
3. You will need a live, fully-synced Edgeware node running with the `--validator` flag that has set one's session keys, either before or after you complete the onboarding process.

## Pre-requisites
- First follow the guide in the [README.md](https://github.com/hicommonwealth/edgeware-node/blob/master/README.md) for installing and running the `edgeware-node`.
- Download from source or from the `npm` registry the `edgeware-cli` located [here](https://github.com/hicommonwealth/edgeware-cli/).
Note: `edgeware-cli` has several dependencies [viewable here.](https://www.npmjs.com/package/edgeware-cli) 
- Install `subkey` as well if you do need to generate new keypairs:

    cargo install --force --git https://github.com/paritytech/substrate subkey

From this point on, we will assume you are familiar with using `subkey`, if that is not the case, you can read about the `subkey` commands [here](https://github.com/paritytech/substrate/tree/master/subkey).

## Onboarding
1. First, create the ***stash*** and ***controller*** keypairs using `subkey`. You can also **optionally** **create your 3 session keys. Ensure all 3 session keys are ED25519 keypairs, using `-e` flag with subkey.
2. Next, you will need to bond from your ***stash*** keypair to your ***controller*** keypair. Using the CLI, you will run:
```
/bin/edge -r edgeware -s <STASH_SEED> staking bond <CONTROLLER_B58_ADDRESS> <AMOUNT> <REWARD_DESTINATION>
```
- The ***stash*** seed should be a mnemonic + derivation path for your ***stash*** keypair
- The ***controller*** address should be a Base58 encoded public key (starts with a 5)
- The bond ***amount*** should be an integer balance
- The ***reward destination*** is where rewards will go; the options are `stash`, `controller`, and `staked` (where staked adds rewards to the amount staked)
3. Next, you will need to set your validator preferences from your ***controller*** account. Using the CLI, you will run:
```
/bin/edge -r edgeware -s <CONTROLLER_SEED> staking validate <UNSTAKE_THRESHOLD> <VALIDATOR_PAYMENT>
```
- The ***controller*** seed should be a mnemonic + derivation path for your ***controller*** keypair
- The ***unstake threshold*** is the number of times your node is offline before dropping out
- The ***validator payment*** is the reward the validator takes up front as an integer balance
4. Next, you will need to set your ***session*** keys from your ***controller*** keypair. Using the CLI, you will run:
```
/bin/edge -r edgeware -s <CONTROLLER_SEED> session setKeys <SESSION_PUBLIC_KEY1>,<SESSION_PUBLIC_KEY2>,<SESSION_PUBLIC_KEY3>
```
- The ***controller*** seed should be a mnemonic + derivation path for your ***controller*** keypair
- The ***session*** public keys should be comma separated, and hex-encoded public keys


#### Examples of all the commands are below:
In the following, we have downloaded and compiled `edgeware-cli` from source to yield a `/bin/edge` binary. You can use `tsc` to do so if you compile from source.
```
/bin/edge -r edgeware -s "eager exit major position method auction duck fix journey supply mad caught"//stash staking bond 5HmGM2dKGbZNQYBLgTepEmv9Ynbc6Mep8L4B8ZiUtpcqPhn7 500 stash
/bin/edge -r edgeware -s "eager exit major position method auction duck fix journey supply mad caught"//controller staking validate 3 0
/bin/edge -r edgeware -s "eager exit major position method auction duck fix journey supply mad caught"//controller session setKeys 0xa29aed52530899b4fbc14772cfb9b06284509b8733bfa8d8bf3e09db4a108bb2,0xe8987d80ee5a8a239dd33eabfee6b5bb287264f0a32ce6ef13ccfedf598b98718d8bf3e09db4a108bb2,0x6ed5ac9bf1388dd38c9e271f8e49e4691de2bf174990c7d37b637b46d5b6763b
```


## Validating

Now you will need to install the `edgeware-node` to start validating with the keys you have generated above. If you followed the first step of the pre-requisites you should be able to build the node as is described below and in the repo.

1. Once you build with `cargo build --release`, run your node with `./target/release/edgeware --chain=edgeware --validator`
2. Now that the chain is running, you will need to pass the chain your session keys. To do so, you will need to use the following `curl` commands to send the node your keys.

## If you already have session keys:

For example, if you were selected to participate as a validator from the Edgeware Lockdrop, you will likely want to input the keys you have generated to your node's keystore. Or if you have a generated a set of keys previously that you want to use for validating, these commands will guide you through that process.

    curl -H 'Content-Type: application/json' --data '{ "jsonrpc":"2.0", "method":"author_insertKey", "params":["aura", "SESSION_KEY_SEED", "<public_key>"],"id":1 }' localhost:9933
    curl -H 'Content-Type: application/json' --data '{ "jsonrpc":"2.0", "method":"author_insertKey", "params":["gran", "SESSION_KEY_SEED", "<public_key>"],"id":1 }' localhost:9933
    curl -H 'Content-Type: application/json' --data '{ "jsonrpc":"2.0", "method":"author_insertKey", "params":["imon", "SESSION_KEY_SEED", "<public_key>"],"id":1 }' localhost:9933

You will need to pass in the respective public key for the `aura` and `imon` because it can work for both `SR25519` and `ED25519` keys and needs the public key to decide. ***Note: Edgeware uses ED25519 keys for Aura, Grandpa, and Im_Online***.

## If you don't already have session keys

If you don't have a set of session keys, the following `curl` command will create ***N*** pairs for you where ***N=3*** for the Edgeware network. This is because session participants maintain a key for the Aura, Grandpa, and Im_Online modules.

    curl -H 'Content-Type: application/json' --data '{ "jsonrpc":"2.0", "method":"author_rotateKeys", "id":1 }' localhost:9933

---

After running these `curl` commands, you should receive as output from `stdout` the public keys you provided (or didn't) in a JSON string. That also means the process was a success! You should now see yourself in the list of newly/pending validators to go into effect in future sessions. In the next era (up to 1 hour), if there is a slot available, your node will become an active validator.
