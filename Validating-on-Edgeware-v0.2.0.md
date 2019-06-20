# Validating on Edgeware Testnet v0.2.0

Welcome to the first official, in-depth Edgeware guide to validating. We're happy that you're interested in validating on our testnet and we'll do our best to provide in-depth documentation on the process below. As always, reach out on Discord or Telegram if you have questions about the project.

## Setup
First ensure you have `yarn` or `npm` installed and have downloaded our [edgeware-cli](https://github.com/hicommonwealth/edgeware-cli). You can do this using
`npm i -g edgeware-cli`.

## Requirements
1. You will need 3 keypairs: a `stash` (ed25519 or sr25519), `controller` (ed25519 or sr25519), and `session` (only ed25519) account. You can generate these using the `subkey` utility. We will be using derived keys in the examples, if you do not use derived keys, simply input the seed/mnemonic needed to sign from these accounts.
2. You will need at least the existential balance in both the `stash` and `controller` accounts.
3. You will need a live, fully-synced Edgeware testnet node running with the `--validator` and `--key <SESSION_KEY>.

## Onboarding process
For each step we will post the CLI command that you will need to use with the [edgeware-cli](https://github.com/hicommonwealth/edgeware-cli).
### 1. Bonding
You will need to `bond` EDG tokens from your `stash` account to your `controller` account.
```
edge -r edgeware -s <MNEMONIC/SEED> staking bond <CONTROLLER_ADDRESS> <BOND> <staked/stash/controller>
```
For example, if my stash account uses the template: `<MNEMONIC>//stash` and I want to have staking rewards issued back to the `stash` account.
```
edge -r edgeware -s "eager exit major position method auction duck fix journey supply mad caught"//stash staking bond 0x986e9ef151fb823c56c03edffc94e9c17b7f724bc104bf53332b54c1c5600171 500 stash
```

### 2. Set session key
You will need to link your session public key to your validator account. The chain is `stash->controller->session`. When you link any account using the CLI, ensure you only pass in the public key NOT the private key.
```
edge -r edgeware -s <MNEMONIC/SEED> session setKey <SESSION_KEY>
```
For example
```
edge -r edgeware -s "eager exit major position method auction duck fix journey supply mad caught"//stash session setKey 0xa29aed52530899b4fbc14772cfb9b06284509b8733bfa8d8bf3e09db4a108bb2
```

### 3. Set validating preferences
You will now need to set your validator preferences using your `controller` account for your validating node. There are two parameters: `unstakeThreshold` and `validatorPayment` that will pass into the CLI.
- UnstakeThreshold - how often you want to be reported offline (and slashed) before being removed from the validator set.
- ValidatorPayment - the amount of the reward your validator will keep with the rest going to the nominators.
```
edge -r edgeware -s <MNEMONIC>//controller staking validate <UNSTAKE_THRESHOLD> <VALIDATOR_PAYMENT>
```
```
edge -r edgeware -s "eager exit major position method auction duck fix journey supply mad caught"//controller staking validate 3 0
```

### Finished!
You should now see yourself in the list of newly/pending validators to go into effect in future sessions. In the next era (up to 1 hour), if there is a slot available, your node will become an active validator.
