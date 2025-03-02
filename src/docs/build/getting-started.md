---
title: Getting Started
lang: en-US
---

::: tip
Please **be prepared to set aside approximately one hour** to get everything running properly and **make sure to read through the guide carefully**.
You don't want to miss any important steps that might cause issues down the line.
:::

This tutorial is **designed for developers** who want to learn about the OP Stack by spinning up an OP Stack testnet chain.
We'll walk through the full deployment process and teach you all of the components that make up the OP Stack, and **you'll end up with your very own OP Stack testnet**.

You can use this testnet to experiment and perform tests, or you can choose to modify the chain to adapt it to your own needs.
**The OP Stack is free (as in freedom) and open source software licensed entirely under the MIT license**.
You don't need permission from anyone to modify or deploy the stack in any configuration you want.

::: tip
Modifications to the OP Stack may prevent a chain from being able to benefit from aspects of the [Optimism Superchain](/op-stack/explainer).
Make sure to check out the [Superchain Explainer](/op-stack/explainer) to learn more.
:::

## What We're Going to Deploy

When deploying an OP Stack chain, you'll be setting up four different components.
It's useful to understand what each of these components does before you start deploying your chain.

### Smart Contracts

The OP Stack gives you the ability to deploy your own Rollup chains that use a Layer 1 blockchain to host and order transaction data.
OP Stack chains use several smart contracts on the L1 blockchain to manage aspects of the Rollup.
Each OP Stack chain has its own set of L1 smart contracts that are deployed when the chain is created.
We'll be using the L1 smart contracts found in the [`contracts-bedrock` package](https://github.com/ethereum-optimism/optimism/tree/develop/packages/contracts-bedrock) within the [Optimism Monorepo](https://github.com/ethereum-optimism/optimism).

### Sequencer Node

OP Stack chains use Sequencer nodes to gather transactions from users and publish them to the L1 blockchain.
Vanilla (unmodified) OP Stack chains rely on at least one of these Sequencer nodes, so we'll have to run one.
You can also run additional non-Sequencer nodes if you'd like (not included in this tutorial).

#### Consensus Client

OP Stack nodes, like Ethereum nodes, have a consensus client.
The consensus client is responsible for determining the list and ordering of blocks and transactions that are part of your blockchain.
Several implementations of the OP Stack consensus client exist, including `op-node` (maintained by OP Labs) and [`magi`](https://github.com/a16z/magi) (maintained by a16z).
In this tutorial we'll be using the [`op-node` implementation](https://github.com/ethereum-optimism/optimism/tree/develop/op-node) found within the [Optimism Monorepo](https://github.com/ethereum-optimism/optimism).

#### Execution Client

OP Stack nodes, like Ethereum nodes, also have an execution client.
The execution client is responsible to executing transactions and storing/updating the state of the blockchain.
Various implementations of the OP Stack execution client exist, including `op-geth` (maintained by OP Labs), [`op-erigon`](https://github.com/testinprod-io/op-erigon) (maintained by Test in Prod), and `op-nethermind` (coming soon).
In this tutorial we'll be using the [`op-geth` implementation](https://github.com/ethereum-optimism/op-geth) found within the [`op-geth` repository](https://github.com/ethereum-optimism/op-geth).

### Batcher

The Batcher is an entity for publishing transactions from the Sequencer to the L1 blockchain.
The Batcher runs continuously alongside the Sequencer and publishes transactions in batches (hence the name) on a regular basis.
We'll be using the [`op-batcher` implementation](https://github.com/ethereum-optimism/optimism/tree/develop/op-batcher) of the Batcher component found within the [Optimism Monorepo](https://github.com/ethereum-optimism/optimism).

### Proposer

The Proposer is an entity responsible for publishing transactions *results* (in the form of L2 state roots) to the L1 blockchain.
This allows smart contracts on L1 to read the state of the L2, which is necessary for cross-chain communication and user withdrawals.
It's likely that the Proposer will be removed in the future, but for now it's a necessary component of the OP Stack.
We'll be using the [`op-proposer` implementation](https://github.com/ethereum-optimism/optimism/tree/develop/op-proposer) of the Proposer component found within the [Optimism Monorepo](https://github.com/ethereum-optimism/optimism).

## Software Dependencies

| Dependency | Version | Version Check Command |
| --- | --- | --- |
| [git](https://git-scm.com/) | `^2` | `git --version` |
| [go](https://go.dev/) | `^1.21` | `go version` |
| [node](https://nodejs.org/en/) | `^20` | `node --version` |
| [pnpm](https://pnpm.io/installation) | `^8` | `pnpm --version` |
| [foundry](https://github.com/foundry-rs/foundry#installation) | `^0.2.0` | `forge --version` |
| [make](https://linux.die.net/man/1/make) | `^4` | `make --version` |
| [jq](https://github.com/jqlang/jq) | `^1.6` | `jq --version` |
| [direnv](https://direnv.net) | `^2` | `direnv --version` |

### Notes on Specific Dependencies

#### `node`

We recommend using the latest LTS version of Node.js (currently v20).
[`nvm`](https://github.com/nvm-sh/nvm) is a useful tool that can help you manage multiple versions of Node.js on your machine.
You may experience unexpected errors on older versions of Node.js.

#### `direnv`

Parts of this tutorial use [`direnv`](https://direnv.net) as a way of loading environment variables from `.envrc` files into your shell.
This means you won't have to manually export environment variables every time you want to use them.
`direnv` only ever has access to files that you explicitly allow it to see.

After [installing `direnv`](https://direnv.net/docs/installation.html), you will need to **make sure that [`direnv` is hooked into your shell](https://direnv.net/docs/hook.html)**.
Make sure you've followed [the guide on the `direnv` website](https://direnv.net/docs/hook.html), then **close your terminal and reopen it** so that the changes take effect (or `source` your config file if you know how to do that).

::: warning
Make sure that you have correctly hooked `direnv` into your shell by modifying your shell configuration file (like `~/.bashrc` or `~/.zshrc`).
If you haven't edited a config file thens you probably haven't configured `direnv` properly (and things might not work later).
:::

## Get Access to a Sepolia Node

We'll be deploying a OP Stack Rollup chain that uses a Layer 1 blockchain to host and order transaction data.
The OP Stack Rollups were designed to use EVM Equivalent blockchains like Ethereum, OP Mainnet, or standard Ethereum testnets as their L1 chains.

**This guide uses the Sepolia testnet as an L1 chain**.
We recommend that you also use Sepolia.
You can also use use other EVM-compatible blockchains, but you may run into unexpected errors.
If you want to use an alternative network, make sure to carefully review each command and replace any Sepolia-specific values with the values for your network.

Since we're deploying our OP Stack chain to Sepolia, you'll need to have access to a Sepolia node.
You can either use a node provider like [Alchemy](https://www.alchemy.com/) (easier) or run your own Sepolia node (harder).

## Build the Source Code

We're going to be spinning up our OP Stack chain directly from source code instead of using a container system like [Docker](https://www.docker.com/).
Although this adds a few extra steps, it means you'll have an easier time modifying the behavior of the stack if you'd like to do so.
If you want a summary of the various components we'll be using, take another look at the [What We're Going to Deploy](#what-were-going-to-deploy) section above.

::: warning
We're using the home directory `~/` as the work directory for this tutorial for simplicity.
You can use any directory you'd like but using the home directory will allow you to copy/paste the commands in this guide.
If you choose to use a different directory, make sure you're using the correct directory in the commands throughout this tutorial.
:::

### Build the Optimism Monorepo

#### 1. Clone the Optimism Monorepo

```bash
cd ~
git clone https://github.com/ethereum-optimism/optimism.git
```

#### 2. Enter the Optimism Monorepo

```bash
cd optimism
```

#### 3. Check your dependencies

::: warning
Don't skip this step! Make sure you have all of the required dependencies installed before continuing.
:::

Run the following script and double check that you have all of the required versions installed.
If you don't have the correct versions installed, you may run into unexpected errors.

```bash
./packages/contracts-bedrock/scripts/getting-started/versions.sh
```

#### 4. Install dependencies

```bash
pnpm install
```

#### 5. Build the various packages inside of the Optimism Monorepo

```bash
make op-node op-batcher op-proposer
pnpm build
```

### Build `op-geth`

#### 1. Clone op-geth

```bash
cd ~
git clone https://github.com/ethereum-optimism/op-geth.git
```

#### 2. Enter op-geth

```bash
cd op-geth
```

#### 3. Build op-geth

```bash
make geth
```


## Fill Out Environment Variables

You'll need to fill out a few environment variables before we can start deploying our chain.

#### 1. Enter the Optimism Monorepo

```bash
cd ~/optimism
```

#### 2. Duplicate the sample environment variable file

```bash
cp .envrc.example .envrc
```

#### 3. Fill out the environment variable file

Open up the environment variable file and fill out the following variables:

| Variable Name | Description |
| --- | --- |
| `L1_RPC_URL` | URL for your L1 node (a Sepolia node in this case). |
| `L1_RPC_KIND` | Kind of L1 RPC you're connecting to, used to inform optimal transactions receipts fetching. Valid options: `alchemy`, `quicknode`, `infura`, `parity`, `nethermind`, `debug_geth`, `erigon`, `basic`, `any`. |

## Generate Accounts

You'll need four accounts and their private keys when setting up the chain:

- The `Admin` account has the ability to upgrade contracts.
- The `Batcher` account publishes Sequencer transaction data to L1.
- The `Proposer` account publishes L2 transaction results (state roots) to L1.
- The `Sequencer` account signs blocks on the p2p network.

#### 1. Enter the Optimism Monorepo

```bash
cd ~/optimism
```

#### 2. Generate new accounts

::: danger
You should **not** use the `wallets.sh` tool for production deployments.
If you are deploying an OP Stack based chain into production, you should likely be using a combination of hardware security modules and hardware wallets.
:::

```bash
./packages/contracts-bedrock/scripts/getting-started/wallets.sh
```

#### 3. Check the output

Make sure that you see output that looks something like the following:

```text
Copy the following into your .envrc file:
 
# Admin account
export GS_ADMIN_ADDRESS=0x9625B9aF7C42b4Ab7f2C437dbc4ee749d52E19FC
export GS_ADMIN_PRIVATE_KEY=0xbb93a75f64c57c6f464fd259ea37c2d4694110df57b2e293db8226a502b30a34
 
# Batcher account
export GS_BATCHER_ADDRESS=0xa1AEF4C07AB21E39c37F05466b872094edcf9cB1
export GS_BATCHER_PRIVATE_KEY=0xe4d9cd91a3e53853b7ea0dad275efdb5173666720b1100866fb2d89757ca9c5a
 
# Proposer account
export GS_PROPOSER_ADDRESS=0x40E805e252D0Ee3D587b68736544dEfB419F351b
export GS_PROPOSER_PRIVATE_KEY=0x2d1f265683ebe37d960c67df03a378f79a7859038c6d634a61e40776d561f8a2
 
# Sequencer account
export GS_SEQUENCER_ADDRESS=0xC06566E8Ec6cF81B4B26376880dB620d83d50Dfb
export GS_SEQUENCER_PRIVATE_KEY=0x2a0290473f3838dbd083a5e17783e3cc33c905539c0121f9c76614dda8a38dca
```

#### 4. Save the accounts

Copy the output from the previous step and paste it into your `.envrc` file as directed.

#### 5. Fund the accounts

**You will need to send ETH to the `Admin`, `Proposer`, and `Batcher` accounts.**
The exact amount of ETH required depends on the L1 network being used.
**You do not need to send any ETH to the `Sequencer` account as it does not send transactions.**

We recommend funding the accounts with the following amounts when using Sepolia:

- `Admin` — 0.2 ETH
- `Proposer` — 0.2 ETH
- `Batcher` — 0.1 ETH

## Load Environment variables

Now that we've filled out the environment variable file, we need to load those variables into our terminal.

#### 1. Enter the Optimism Monorepo

```bash
cd ~/optimism
```

#### 2. Load the variables with direnv

::: warning
We're about to use `direnv` to load environment variables from the `.envrc` file into our terminal.
Make sure that you've [installed `direnv`](https://direnv.net/docs/installation.html) and that you've properly [hooked `direnv` into your shell](#configuring-direnv).
:::

Next you'll need to allow `direnv` to read this file and load the variables into your terminal using the following command.

```bash
direnv allow
```

::: warning
WARNING: `direnv` will unload itself whenever your `.envrc` file changes.
**You *must* rerun the following command every time you change the `.envrc` file.**
:::

#### 3. Confirm that the variables were loaded

After running `direnv allow` you should see output that looks something like the following (the exact output will vary depending on the variables you've set, don't worry if it doesn't look exactly like this):

```bash
direnv: loading ~/optimism/.envrc                                                            
direnv: export +DEPLOYMENT_CONTEXT +ETHERSCAN_API_KEY +GS_ADMIN_ADDRESS +GS_ADMIN_PRIVATE_KEY +GS_BATCHER_ADDRESS +GS_BATCHER_PRIVATE_KEY +GS_PROPOSER_ADDRESS +GS_PROPOSER_PRIVATE_KEY +GS_SEQUENCER_ADDRESS +GS_SEQUENCER_PRIVATE_KEY +IMPL_SALT +L1_RPC_KIND +L1_RPC_URL +PRIVATE_KEY +TENDERLY_PROJECT +TENDERLY_USERNAME
```

**If you don't see this output, you likely haven't [properly configured `direnv`](#configuring-direnv).**
Make sure you've configured `direnv` properly and run `direnv allow` again so that you see the desired output.

## Configure your network

Once you've built both repositories, you'll need head back to the Optimism Monorepo to set up the configuration file for your chain.
Currently, chain configuration lives inside of the [`contracts-bedrock`](https://github.com/ethereum-optimism/optimism/tree/129032f15b76b0d2a940443a39433de931a97a44/packages/contracts-bedrock) package in the form of a JSON file.

#### 1. Enter the Optimism Monorepo

```bash
cd ~/optimism
```

#### 2. Move into the contracts-bedrock package

```bash
cd packages/contracts-bedrock
```

#### 3. Generate the configuration file

Run the following script to generate the `getting-started.json` configuration file inside of the `deploy-config` directory.

```bash
./scripts/getting-started/config.sh
```

#### 4. Review the configuration file (Optional)

If you'd like, you can review the configuration file that was just generated by opening up `deploy-config/getting-started.json` in your favorite text editor.
We recommend keeping this file as-is for now so you don't run into any unexpected errors.

## Deploy the L1 contracts

Once you've configured your network, it's time to deploy the L1 contracts necessary for the functionality of the chain.

#### 1. Deploy the L1 contracts

```bash
forge script scripts/Deploy.s.sol:Deploy --private-key $GS_ADMIN_PRIVATE_KEY --broadcast --rpc-url $L1_RPC_URL
```

::: warning
If you see a nondescript error that includes `EvmError: Revert` and `Script failed` then you likely need to change the `IMPL_SALT` environment variable.
This variable determines the addresses of various smart contracts that are deployed via [CREATE2](https://eips.ethereum.org/EIPS/eip-1014).
If the same `IMPL_SALT` is used to deploy the same contracts twice, the second deployment will fail.
**You can generate a new `IMPL_SALT` by running `direnv allow` anywhere in the Optimism Monorepo.**
:::

#### 2. Generate contract artifacts

```bash
forge script scripts/Deploy.s.sol:Deploy --sig 'sync()' --rpc-url $L1_RPC_URL
```

## Generate the L2 config files

Now that we've set up the L1 smart contracts we can automatically generate several configuration files that are used within the Consensus Client and the Execution Client.

We need to generate three important files:

1. `genesis.json` includes the genesis state of the chain for the Execution Client.
1. `rollup.json` includes configuration information for the Consensus Client.
1. `jwt.txt` is a [JSON Web Token](https://jwt.io/introduction) that allows the Consensus Client and the Execution Client to communicate securely (the same mechanism is used in Ethereum clients).

#### 1. Navigate to the op-node package

```bash
cd ~/optimism/op-node
```

#### 2. Create genesis files

Now we'll generate the `genesis.json` and `rollup.json` files within the `op-node` folder:

```bash
go run cmd/main.go genesis l2 \
    --deploy-config ../packages/contracts-bedrock/deploy-config/getting-started.json \
    --deployment-dir ../packages/contracts-bedrock/deployments/getting-started/ \
    --outfile.l2 genesis.json \
    --outfile.rollup rollup.json \
    --l1-rpc $L1_RPC_URL
```

#### 3. Create an authentication key

Next you'll create a [JSON Web Token](https://jwt.io/introduction) that will be used to authenticate the Consensus Client and the Execution Client.
This token is used to ensure that only the Consensus Client and the Execution Client can communicate with each other.
You can generate a JWT with the following command:

```bash
openssl rand -hex 32 > jwt.txt
```

#### 4. Copy genesis files into the op-geth directory

Finally, we'll need to copy the `genesis.json` file and `jwt.txt` file into `op-geth` so we can use it to initialize and run `op-geth`:

```bash
cp genesis.json ~/op-geth
cp jwt.txt ~/op-geth
```

## Initialize `op-geth`

We're almost ready to run our chain!
Now we just need to run a few commands to initialize `op-geth`.
We're going to be running a Sequencer node, so we'll need to import the `Sequencer` private key that we generated earlier.
This private key is what our Sequencer will use to sign new blocks.

#### 1. Navigate to the op-geth directory

```bash
cd ~/op-geth
```

#### 2. Create a data directory folder

```bash
mkdir datadir
```

#### 3. Initialize op-geth

```bash
build/bin/geth init --datadir=datadir genesis.json
```

## Start `op-geth`

Now we'll start `op-geth`, our Execution Client.
Note that you won't start seeing any transactions until you start the Consensus Client in the next step.

#### 1. Open up a new terminal

We'll need a terminal window to run `op-geth` in.

#### 2. Navigate to the op-geth directory

```bash
cd ~/op-geth
```

#### 3. Run op-geth

::: tip
We're using `--gcmode=archive` to run `op-geth` here because this node will act as our Sequencer.
It's useful to run the Sequencer in archive mode because the `op-proposer` requires access to the full state.
Feel free to run other (non-Sequencer) nodes in full mode if you'd like to save disk space.
:::

```bash
./build/bin/geth \
    --datadir ./datadir \
    --http \
    --http.corsdomain="*" \
    --http.vhosts="*" \
    --http.addr=0.0.0.0 \
    --http.api=web3,debug,eth,txpool,net,engine \
    --ws \
    --ws.addr=0.0.0.0 \
    --ws.port=8546 \
    --ws.origins="*" \
    --ws.api=debug,eth,txpool,net,engine \
    --syncmode=full \
    --gcmode=archive \
    --nodiscover \
    --maxpeers=0 \
    --networkid=42069 \
    --authrpc.vhosts="*" \
    --authrpc.addr=0.0.0.0 \
    --authrpc.port=8551 \
    --authrpc.jwtsecret=./jwt.txt \
    --rollup.disabletxpoolgossip=true
```

## Start `op-node`

Once we've got `op-geth` running we'll need to run `op-node`.
Like Ethereum, the OP Stack has a Consensus Client (`op-node`) and an Execution Client (`op-geth`). 
The Consensus Client "drives" the Execution Client over the Engine API.

#### 1. Open up a new terminal

We'll need a terminal window to run the `op-node` in.

#### 2. Navigate to the op-node directory

```bash
cd ~/optimism/op-node
```

#### 3. Run op-node

```bash
./bin/op-node \
	--l2=http://localhost:8551 \
	--l2.jwt-secret=./jwt.txt \
	--sequencer.enabled \
	--sequencer.l1-confs=5 \
	--verifier.l1-confs=4 \
	--rollup.config=./rollup.json \
	--rpc.addr=0.0.0.0 \
	--rpc.port=8547 \
	--p2p.disable \
	--rpc.enable-admin \
	--p2p.sequencer.key=$GS_SEQUENCER_PRIVATE_KEY \
	--l1=$L1_RPC_URL \
	--l1.rpckind=$L1_RPC_KIND
```

Once you run this command, you should start seeing the `op-node` begin to sync L2 blocks from the L1 chain.
Once the `op-node` has caught up to the tip of the L1 chain, it'll begin to send blocks to `op-geth` for execution.
At that point, you'll start to see blocks being created inside of `op-geth`.

::: tip
**By default, your `op-node` will try to use a peer-to-peer to speed up the synchronization process.**
If you're using a chain ID that is also being used by others, like the default chain ID for this tutorial (42069), your `op-node` will receive blocks signed by other sequencers.
These requests will fail and waste time and network resources.
**To avoid this, we start with peer-to-peer synchronization disabled (`--p2p.disable`).**

Once you have multiple nodes, you may want to enable peer-to-peer synchronization.
You can add the following options to the `op-node` command to enable peer-to-peer synchronization with specific nodes:

```
	--p2p.static=<nodes> \
	--p2p.listen.ip=0.0.0.0 \
	--p2p.listen.tcp=9003 \
	--p2p.listen.udp=9003 \
```

You can alternatively also remove the `--p2p.static` option but you may see failed requests from other chains using the same chain ID.
:::

## Start `op-batcher`

The `op-batcher` takes transactions from the Sequencer and publishes those transactions to L1.
Once these Sequencer transactions are included in a finalized L1 block, they're officially part of the canonical chain.
The `op-batcher` is critical!

It's best to give the `Batcher` account at least 1 Sepolia ETH to ensure that it can continue operating without running out of ETH for gas.
Keep an eye on the balance of the `Batcher` account because it can expend ETH quickly if there are a lot of transactions to publish.

#### 1. Open up a new terminal

We'll need a terminal window to run the `op-batcher` in.

#### 2. Navigate to the op-batcher directory

```bash
cd ~/optimism/op-batcher
```

#### 3. Run op-batcher

```bash
./bin/op-batcher \
    --l2-eth-rpc=http://localhost:8545 \
    --rollup-rpc=http://localhost:8547 \
    --poll-interval=1s \
    --sub-safety-margin=6 \
    --num-confirmations=1 \
    --safe-abort-nonce-too-low-count=3 \
    --resubmission-timeout=30s \
    --rpc.addr=0.0.0.0 \
    --rpc.port=8548 \
    --rpc.enable-admin \
    --max-channel-duration=1 \
    --l1-eth-rpc=$L1_RPC_URL \
    --private-key=$GS_BATCHER_PRIVATE_KEY
```

::: tip
The `--max-channel-duration=n` setting tells the batcher to write all the data to L1 every `n` L1 blocks.
When it is low, transactions are written to L1 frequently and other nodes can synchronize from L1 quickly.
When it is high, transactions are written to L1 less frequently and the batcher spends less ETH.
If you want to reduce costs, either set this value to 0 to disable it or increase it to a higher value.
:::

## Start `op-proposer`

Now start `op-proposer`, which proposes new state roots.

#### 1. Open up a new terminal

We'll need a terminal window to run the `op-proposer` in.

#### 2. Navigate to the op-proposer directory

```bash
cd ~/optimism/op-proposer
```

#### 3. Run op-proposer

```bash
./bin/op-proposer \
    --poll-interval=12s \
    --rpc.port=8560 \
    --rollup-rpc=http://localhost:8547 \
    --l2oo-address=$(cat ../packages/contracts-bedrock/deployments/getting-started/L2OutputOracleProxy.json | jq -r .address) \
    --private-key=$GS_PROPOSER_PRIVATE_KEY \
    --l1-eth-rpc=$L1_RPC_URL
```

## Connect Your Wallet to Your Chain

You now have a fully functioning OP Stack Rollup with a Sequencer node running on `http://localhost:8545`.
You can connect your wallet to this chain the same way you'd connect your wallet to any other EVM chain.
If you need an easy way to connect to your chain, just [click here](https://chainid.link?network=opstack-getting-started).

## Get ETH On Your Chain

Once you've connected your wallet, you'll probably notice that you don't have any ETH to pay for gas on your chain.
The easiest way to deposit Sepolia ETH into your chain is to send funds directly to the `L1StandardBridge` contract.

#### 1. Navigate to the contracts-bedrock directory

```bash
cd ~/optimism/packages/contracts-bedrock
```

#### 2. Get the address of the L1StandardBridgeProxy contract

```bash
cat deployments/getting-started/L1StandardBridgeProxy.json | jq -r .address
```

#### 3. Send some Sepolia ETH to the L1StandardBridgeProxy contract

Grab the L1 bridge proxy contract address and, using the wallet that you want to have ETH on your Rollup, send that address a small amount of ETH on Sepolia (0.1 or less is fine).
This will trigger a deposit that will mint ETH into your wallet on L2.
It may take up to 5 minutes for that ETH to appear in your wallet on L2.

## See Your Rollup in Action

You can interact with your Rollup the same way you'd interact with any other EVM chain.
Send some transactions, deploy some contracts, and see what happens!

## Next Steps

- You can use this Rollup the same way you'd use any other test blockchain.
- You can [modify the blockchain in various ways](hacks/overview). 
- If you run into any problems, please visit the [Chain Operators Troubleshooting Guide](chain-troubleshooting) for help.
