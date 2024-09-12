# Comprehensive guide to setting up a distributed Lido CSM validator cluster with Obol Network

The [Community Staking Module](https://operatorportal.lido.fi/modules/community-staking-module) (CSM) is the first permissionless module in the [Lido protocol](https://lido.fi), allowing anyone to start running validators on the Ethereum blockchain with much greater capital efficiency compared to running a regular ("vanilla") Ethereum validator.

A distributed validator ([DVT](https://ethereum.org/en/staking/dvt/)) is an Ethereum validator that runs on more than one node. [Obol Network](https://obol.org/) is a set of tools providing permissionless access to running distributed validators.

This tutorial uses the Holesky testnet for demonstration purposes, but the same steps can be applied to the mainnet.

# Hardware & system requirements

 - CPU: Quad-core
 - RAM: 16GB 
 - Storage: 512GB NVME SSD (For mainnet at least 2TB)

A full guide to setting up your operating system can be found [here](https://dvt-homestaker.stakesaurus.com/) or [here](https://docs.ethstaker.cc/ethstaker-knowledge-base). For this tutorial, I'm assuming that all cluster members are running Linux with Git and Docker installed, and have properly secured their servers.

# Getting started

Creating a trust-minimised distributed Valdator cluster requires a multi-sig wallet for management, a split contract to distribute rewards to operators, and the client software needed to run the Obol DVT - `Charon`. Operators also need a full Ethereum node with an execution and consensus client of their choice, and the MEV-boost client configured with at least one of the Lido approved relays.

## The Charon client

`Charon` (pronounced 'kharon') is the software that allows validators to be run on a group of independent nodes - a cluster. A complete multi-container `Docker` setup including execution client, consensus client, MEV-Boost and the `Charon` client can be found in this repository https://github.com/ObolNetwork/charon-distributed-validator-cluster and the first step is to clone it:

```
git clone https://github.com/ObolNetwork/charon-distributed-validator-node.git
```

Make sure your user has the `docker` role. If not you can use this command to add it:

```
sudo usermod -a -G docker $USER
```

You will then need to exit the ssh session and log in again.

Finally, you will need to create the `Charon` data folder and the `.env` configuration file:

```
cd charon-distributed-validator-node
mkdir .charon
cp .env.sample .env
```

Edit the `.env` in your favourite editor and set the variable `NETWORK=holesky`.

## Preparing the ENR (Ethereum Node Record)

All cluster members will need an ENR (Ethereum Node Record) to connect to the Obol Network. To create an ENR the operator can use this code:

```
docker compose run --rm charon create enr
```

![image](https://hackmd.io/_uploads/HJeGRFBhR.png)

## Creating the DV cluster wallet

Detailed instructions on how to create a Safe Wallet can be found [here](https://help.safe.global/en/articles/40868-creating-a-safe-on-a-web-browser). 
The Holesky Testnet Safe deployment can be found at this address: https://holesky-safe.protofire.io

One of the cluster members should obtain the signer addresses from all the cluster members, then connect his signer wallet and choose to create a new Safe. 

![chrome_ofxRcHQItb](https://hackmd.io/_uploads/HJImiiVh0.png)

After giving the Safe a name and selecting the Holesky network, he continues by clicking the `Next` button.

![chrome_0n8nPU5G5q](https://hackmd.io/_uploads/SkrQijV2R.png )

Then he adds all the signer addresses of the cluster members and proceeds to the final step by clicking the `Next` button.

![chrome_qvPajGtE0N](https://hackmd.io/_uploads/S18mijE3A.png)

Finally, he sends the transaction to create the Safe by clicking on the `Create` button.

![chrome_BfjGLxjtYM](https://hackmd.io/_uploads/SkrmjoV3C.png)

## Creating the reward split contract

One of the cluster members should obtain the reward addresses from all the cluster members. Then he should open https://app.splits.org and select to create a `new contract`. Then he should select `Holesky` for the network.

![image](https://hackmd.io/_uploads/B1VDt6S2A.png)

Select `Split` for the contract type.

![image](https://hackmd.io/_uploads/BknFFprh0.png)

Add the reward addresses of all cluster members. Then he can choose whether the contract is immutable (recommended option), whether he wants to sponsor the maintainers of [splits.org](https://splits.org), and whether there is a distribution bounty so that third parties can distribute the rewards in exchange for a small fee.

![image](https://hackmd.io/_uploads/H1q0KaS20.png)

Finally, click the `Create Split` button, execute the transaction and share the created split contract with all cluster members for review.

## Creating the DV cluster

The official Obol [documentation](https://docs.obol.org/docs/start/quickstart_group) contains detailed instructions on setting up a distributed cluster.

### Method 1 - Creating the cluster using the DV Launchpad

Unfortunately, it is possible to create a distributed cluster using the DV Launchpad only for a single validator key, as the DV Launchpad does not allow you to set the custom Withdrawal and Fee Recipient addresses when creating a cluster with more than one, which is a requirement for Lido CSM.

#### Creating the cluster configuration

One of the cluster members opens the Holesky DV Launchpad at this address - https://holesky.launchpad.obol.org, then connects his wallet and chooses to create a `Cluster with a group` button.

![image](https://hackmd.io/_uploads/B1arp242R.png)

Then he clicks on the `Getting Started` button on the next page.

![chrome_WW8rTcaqeQ](https://hackmd.io/_uploads/H1LXijNnR.png)

Accepts all the necessary advisories and signs the confirmation.

![chrome_rQIeibZpcj](https://hackmd.io/_uploads/HkL7ioN20.png)

On the next page is where the cluster is configured. First, he should select the cluster name and size. Then he enters all cluster members' signer addresses,

![chrome_HeH3K82wfh](https://hackmd.io/_uploads/S1HXjsE30.png)

sets the `validators` field to `1` and in the `Withdrawal Configuration` section selects the `Custom` tab, then set the `Withdrawal Address` to Lido's `Withdrawal Vault` - `0xF0179dEC45a37423EAD4FaD5fCb136197872EAd9` and `Fee Recipient` to Lido's `Execution Layer Rewards Vault` - `0xE73a3602b99f1f913e72F8bdcBC235e206794Ac8` as per Lido [documentation](https://docs.lido.fi/deployed-contracts/holesky/). Finally, he clicks on the `Create cluster configuration` button.

![chrome_Os8B70haUT](https://hackmd.io/_uploads/r1rXjsEhC.png)

Lastly, he shares the cluster configuration link with the other cluster members.

![chrome_wHk3z8Tz9S](https://hackmd.io/_uploads/By8XoiE2R.png)

#### Distributed Key Generation

All cluster members need to open the configuration link, connect their wallet, and check the cluster size, the threshold and the number of validators. 

![chrome_UjIAAKB8oi](https://hackmd.io/_uploads/r1I7jjVh0.png)

Then check that the `Withdrawal Address` mach Lido's `Withdrawal Vault` - `0xF0179dEC45a37423EAD4FaD5fCb136197872EAd9` and the `Fee Recipient` mach Lido's Execution Layer Rewards Vault - `0xE73a3602b99f1f913e72F8bdcBC235e206794Ac8` as per Lido [documentation](https://docs.lido.fi/deployed-contracts/holesky/), and finally click the `Getting Started` button.

![chrome_J8CvZeGsub](https://hackmd.io/_uploads/HyBQjiVhA.png)

Accept all the necessary advisories.

![image](https://hackmd.io/_uploads/B1D5DTN2R.png)

On the `Accept configuration` page, each cluster member submits its ENR (the whole address including the `enr:` prefix).

![image](https://hackmd.io/_uploads/HkmCO6NhR.png)

Finally, confirms and signs the configuration...

![image](https://hackmd.io/_uploads/BkhtYa4hR.png)

Then wait for all the other cluster members to accept it.

![image](https://hackmd.io/_uploads/B1wgcTV3A.png)

Once all members confirm the configuration they will see the `continue` button.

![image](https://hackmd.io/_uploads/HkHwc6VhC.png)

On the next page, they will find a CLI command.

![image](https://hackmd.io/_uploads/Bytw6pE3A.png)

After executing it they should wait for all the other cluster members to connect and complete the DKG ceremony.

![image](https://hackmd.io/_uploads/S1zcThHn0.png)

A `cluster-lock.json` file is created in the `.charon` folder as well as the `deposit-data.json` file and the `validator_keys` folder containing each operator's partial key signatures for the validators.

**At this point, each operator must make a backup of the `.charon` folder and keep it safe, as validator keys can't be recreated.**

### Method 2 - Creating the cluster using the Charon CLI

To create a cluster with more than one validator, the `Charon` CLI is required. One of the cluster members creates the cluster definition file. To do this, he must obtain the ENRs of all the members. Once all the ENRs have been collected, the following command is executed. The 'Withdrawal Address' argument must be set to the Lido 'Withdrawal Vault' - `0xF0179dEC45a37423EAD4FaD5fCb136197872EAd9` and the 'Fee Recipient' must be set to the Lido 'Execution Layer Rewards Vault' - `0xE73a3602b99f1f913e72F8bdcBC235e206794Ac8` as per Lido [documentation](https://docs.lido.fi/deployed-contracts/holesky/):

```
docker compose run --rm charon create dkg \
--network="holesky" \
--name="<Your Cluster Name>" \
--operator-enrs="<Operator1 ENR>,<Operator2 ENR>, ..<OperatorN ENR>" \
--num-validators="<The Nubmer of Validators>" \
--fee-recipient-addresses="0xE73a3602b99f1f913e72F8bdcBC235e206794Ac8" \
--withdrawal-addresses="0xF0179dEC45a37423EAD4FaD5fCb136197872EAd9" 
```

![image](https://hackmd.io/_uploads/SJonDhHn0.png)

A `cluster-definition.json` file is created in the `.charon` folder, which must be shared with all the other cluster members and they need to place it in the `.charon` folder on their machine. They must then open the file and check that their ENR is correct. 

![image](https://hackmd.io/_uploads/HJthdnr3C.png)

They must also confirm that the cluster threshold and the number of validators are correct and that the `Withdrawal Address` mach Lido's `Withdrawal Vault` - `0xF0179dEC45a37423EAD4FaD5fCb136197872EAd9` and the `Fee Recipient` mach Lido's Execution Layer Rewards Vault - `0xE73a3602b99f1f913e72F8bdcBC235e206794Ac8` as per Lido [documentation](https://docs.lido.fi/deployed-contracts/holesky/) for each validator.

![image](https://hackmd.io/_uploads/SyS0YhHh0.png)

After all cluster members have confirmed the cluster definition, each member starts the DKG Ceremony by executing this command:

```
docker compose run --rm charon dkg --publish
```

![image](https://hackmd.io/_uploads/SJjpohrnA.png)

The `--publish` argument is optional. If omitted the cluster will not be published to the Obol API. After all cluster members have executed the command the DKG ceremony is complete. A `cluster-lock.json` file is created in the `.charon` folder as well as the `deposit-data.json` file and the `validator_keys` folder containing each operator's partial key signatures for the validators.

![image](https://hackmd.io/_uploads/S1zcThHn0.png)

**At this point, each operator must make a backup of the `.charon` folder and keep it safe, as validator keys can't be recreated.**

## MEV Boost

### What is MEV

MEV stands for Maximal Extractable Value. This is the additional value that can be captured by the block proposer by optimising the selection and order of the transactions included in the proposed block. Such an optimisation often requires the use of sophisticated algorithms and access to resources not available to the regular node operator. The parties capable of doing this are called `Searchers`. They find the most profitable transactions, bundle them and provide the bundles to the `Block Builders` who assemble the bundles into a complete block. At the beginning of each epoch, node operators register the validators they control with a `Block Builder` (or `Relay`) of their choice and if they are selected to propose a block they can choose to propose the one provided by the `Relay` in exchange for an additional tip. If the operator wishes to connect to multiple `Relays` a software called `MEV-Boost` is required. Using `MEV-Boost` allows the operator to select the most profitable block from all the connected `Relays`, creating a kind of `Block Marketplace`. In the context of Lido CSM, it is worth noting that running `MEV-Boost` is a requirement. Although there are currently no penalties for proposing self-built blocks, this may change in the future.

### Configuring the MEV-boost client

To configure MEV-boost each cluster memeber needs to edit the `.env` file and set the `BUILDER_API_ENABLED=true` and `MEVBOOST_RELAYS=` to the URL of at least one of Lido's approved MEV relays [here](https://enchanted-direction-844.notion.site/6d369eb33f664487800b0dedfe32171e?v=985cb7e521de43d78c67b7ad29adec84). Multiple relays must be separated by a comma. **The use of unapproved relays is strictly forbidden! All cluster members must use identical configurations to avoid missing block proposals due to a lack of consensus!**

## Starting the Node

Each cluster member should start the node with the following command:

```
docker compose up -d
```

At this point, execution and consensus clients should start syncing, and Charon and the validator client should start waiting for the validator to be activated. 

## Deploy the keys to Lido CSM

One of the cluster members opens the Lido CSM widget using this address https://csm.testnet.fi/?mode=extended. Note the `mode=extended` parameter. This allows the Lido CSM reward address to be set to the split contract created earlier. He connects the cluster Safe to the widget using `WalletConnect`.

![image](https://hackmd.io/_uploads/HkNhlaHhA.png)

Copies the connection link...

![image](https://hackmd.io/_uploads/SyTJ-6r2R.png)

And pastes it into the Safe `WalletConnect` screen.

![image](https://hackmd.io/_uploads/HJxuZpH3R.png)

He clicks on the `Create Node Operator` button...

![image](https://hackmd.io/_uploads/BkVsfpr2R.png)

Pastes the contents of the `deposit-data.json` file into the `Upload deposit data` field. There should be enough ETH/stETH/wstETH deposited in the cluster Safe to cover the bond.

![image](https://hackmd.io/_uploads/BJG44pH30.png)

Expand the `Specify custom addresses` section...

![image](https://hackmd.io/_uploads/r1z4X6Bh0.png)

Set the `Reward Address` field to the `Split` contract address and the `Manager Address` field to the `Safe` wallet address. Check that the correct addresses are set and click the `Create Node Operator` button.

![image](https://hackmd.io/_uploads/Hkh34aBh0.png)

Sign the transaction in the safe and share it with the rest of the cluster members.

![image](https://hackmd.io/_uploads/r154TpSnC.png)

Before signing the transaction, the remaining members should check that the transaction details contain the correct manager address (the address of the Safe) and reward address (the address of the split contract).

![image](https://hackmd.io/_uploads/HkliC6Hn0.png)

Once the signature threshold has been reached and the transaction has been executed, the cluster is ready for deposit from Lido CSM.

![image](https://hackmd.io/_uploads/HJwzJRShA.png)

## Monitoring the CSM operator

You can follow [this](https://dvt-homestaker.stakesaurus.com/bonded-validators-setup/lido-csm/monitoring-and-address-management) guide for the steps required to monitor your CSM operator.

## Exiting Validators

You can follow the Obol [launchpad](https://holesky.launchpad.obol.org/cluster/exit/) instructions on how to exit the cluster validators.