Welcome to the official, in-depth Edgeware guide to validating. We're happy that you're interested in validating on Edgeware and we'll do our best to provide in-depth documentation on the process below. As always, reach out on [Discord](https://discord.gg/CJRfb3) or [Telegram](https://t.me/heyedgeware) if you have questions about the project.

This document contains all the information one should need to start validating on Edgeware using the **command line interface**. We will start with how to setup one's node and proceed to how to key management and monitoring. To start, we will use the following terminology of keys for the guide:

- ***stash*** - the stash keypair is where most of your funds should be located. It can be kept in cold storage if necessary.
- ***controller*** - the controller is the keypair that will control your validator settings. It should have a smaller balance, e.g. 10-100 EDG
- ***session*** - the 3 session keypairs are hot keys that are stored on your validator node. They do not need to have balances.

## Requirements

1. You should have balances in your `stash` (ed25519 or sr25519) and `controller` (ed25519 or sr25519) accounts.
2.  Instructions for setting up a node are [here](https://github.com/hicommonwealth/edgeware-node/wiki/Setting-up-a-Node) and you will need to 
3. You should have a wallet, such as the `polkadot-js` extension, installed in your browser with the stash and controller keypairs. If you don't have it, get it [here](https://github.com/polkadot-js/extension).

If you need to request a testnet EDG balance, ask on [Discord](1).

### 1. Install the Edgeware node

Set up a live, fully-synced Edgeware node running with the `--validator` flag.

If you are starting a validator node that needs to stay up, we recommend [following the default instructions](https://github.com/hicommonwealth/edgeware-node/wiki/Setting-up-a-Node) and [setting up monitoring](https://github.com/hicommonwealth/edgeware-node/wiki/Setting-up-monitoring). 

Then, you should go to the line where the edgeware executable is called in `/etc/systemd/system/edgeware.service`, and add the `--validator` flag. Reload the service configuration and check to see the node has started properly:

```
systemctl daemon-reload
systemctl restart edgeware
systemctl status edgeware
```

You should see this output:

```
2019-10-03 10:28:59 Edgeware
2019-10-03 10:28:59   version 1.0.0-3f34fba-x86_64-macos
2019-10-03 10:28:59   by Commonwealth Labs, 2018-2019
2019-10-03 10:28:59 Chain specification: Edgeware
2019-10-03 10:28:59 Node name: naughty-light-7646
2019-10-03 10:28:59 Roles: AUTHORITY
```

**Make sure the Chain specification is correct, and the node is running with `Roles: AUTHORITY.`** If you don't see the correct Chain, you should provide the correct `--chain` parameter, and if you don't see the Authority role, you should make sure you're actually running Edgeware with the `--validator` flag.

**Make sure you can access the node through the command line.** If you enter the `curl` command below, you should see this output:

```
curl --include --no-buffer --header "Connection: Upgrade" --header "Upgrade: websocket" --header "Sec-WebSocket-Key: SGVsbG8sIHdvcmxkIQ==" --header "Sec-WebSocket-Version: 13" localhost:9944

HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Sec-WebSocket-Accept: qGEgH3En71di5rrssAZTmtRTyFk=
Upgrade: websocket
```

**Check the [telemetry dashboard](https://github.com/hicommonwealth/edgeware-node/wiki), and make sure your node is syncing up to the latest block.**

### 2. Connect via the interface

### 3. Create a stake

### 4. Set your session keys, using `rotateKeys`

### 5. Start validating

### 6. Receiving rewards and slashing