Welcome to the official, in-depth Edgeware guide to validating. We're happy that you're interested in validating on Edgeware and we'll do our best to provide in-depth documentation on the process below. As always, reach out on [Discord](https://discord.gg/CJRfb3) or [Telegram](https://t.me/heyedgeware) if you have questions about the project.

This document contains all the information one should need to start validating on Edgeware using the **polkadot-js web interface**. We will start with how to setup one's node and proceed to how to key management and monitoring. To start, we will use the following terminology of keys for the guide:

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

**Check the [telemetry dashboard](https://github.com/hicommonwealth/edgeware-node/wiki), and make sure your node is syncing up to the latest block.** If any of these steps do not check out, stop now; you will not be able to validate until they are corrected.

### 2. Connect via the interface

Go to the [polkadot-js web interface](https://polkadot.js.org/apps/#/settings) and connect to a custom node, e.g. testnet1.edgewa.re, testnet2.edgewa.re, or testnet3.edgewa.re.

The interface should show the correct latest block.

### 3. Create a stake

Go to the **Staking** tab, and select **Account actions** at the top. Click on **New stake**.

Select your controller and stash accounts. You will also be able to choose where your validator rewards are deposited, e.g. to the stash or the controller.

Sign and send the transaction.

### 4. Set your session keys, using `rotateKeys`

Click on **Set Session Keys** on the stake you just created above.

Go to the command line where your validator is running (e.g. SSH into the server, etc.) and enter this command. It will tell your validator to generate a new set of session keys:

```
curl -H 'Content-Type: application/json' --data '{ "jsonrpc":"2.0", "method":"author_rotateKeys", "id":1 }' localhost:9933
```

The output should look like this:

```
{"jsonrpc":"2.0","result":"0x0ca0fbf245e4abca3328f8bba4a286d6cb1796516fcc68864cab580f175e6abd2b9107003014fc6baab7fd8caf4607b34222df62f606248a8a592bcba86ff9eec6e838ae8eb757eb77dffc748f1443e60c4f7617c9ea7905f0dd09ab758a8063","id":1}
```

Copy the hexadecimal key from inside the JSON object, and paste it into the web interface.

Sign and send the transaction. This registers your intent to validate.

### 5. Start validating

You should now be able to see your validator in the **Next up** section of the staking tab.

At the beginning of the next **era**, if there are open slots and your validator has adequate
stake behind it, your validator should join the set of active validators and automatically start
producing blocks.

(On the testnet, sessions are 100 blocks or 10 minutes long, and eras are 300 blocks or 30 minutes long.)

### 6. Receiving rewards and slashing

Your validator will publish an `im_online` heartbeat when each session begins. This prevents it from being slashed for unavailability.

