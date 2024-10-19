 # Deploying an Obol DV on Lido CSM using Linux (Early Access)

To start, this guide assumes you're running Linux and you've installed Git and Docker. We also assume you're familiar with the [Community Staking Module](https://operatorportal.lido.fi/modules/community-staking-module) (CSM) and operating a node using [Obol DVT](https://obol.org/). Also, cluster and squad are used interchangabely within this document. 

This guide uses Holesky testnet. If deploying on mainnet, please adjust required addresses. 
<br><br><br><br>

# Getting started

This guide will be broken down into 3 main steps:

### Part 1: Create cluster multi-sig + 0xSplits contract
### Part 2: Use Obol Launchpad + CLI to create the cluster
### Part 3: Deploy the validator to CSM 

In this guide we'll be using CSM with `extendedManagerPermissions` where the `managerAddress` is set to the cluster multi-sig and the `rewardAddress` is set to the 0xSplits contract. 
<br><br><br><br>

# Part 1: Creating the Cluster multi-sig + 0xSplits Contract

Detailed instructions on how to create a Safe Wallet can be found [here](https://help.safe.global/en/articles/40868-creating-a-safe-on-a-web-browser). 
The Holesky Testnet Safe deployment can be found at this address: https://holesky-safe.protofire.io

Squad leader should obtain the signer addresses from all the cluster members, then connect their signer wallet and choose to create a new Safe. 

![chrome_ofxRcHQItb](https://hackmd.io/_uploads/HJImiiVh0.png)

After giving the Safe a name and selecting the Holesky network, they continue by clicking the `Next` button.

![chrome_0n8nPU5G5q](https://hackmd.io/_uploads/SkrQijV2R.png )

Then add all the signer addresses of the cluster members, select a threshold, and proceed to the final step by clicking the `Next` button.

![chrome_qvPajGtE0N](https://hackmd.io/_uploads/S18mijE3A.png)

Finally, send the transaction to create the Safe by clicking on the `Create` button.

![chrome_BfjGLxjtYM](https://hackmd.io/_uploads/SkrmjoV3C.png)

## Creating the reward split contract

Squad leader should obtain the reward addresses from all the cluster members. Then should open https://app.splits.org and select to create a `new contract`. Then he should select `Holesky` for the network.

![image](https://hackmd.io/_uploads/B1VDt6S2A.png)

Select `Split` for the contract type.

![image](https://hackmd.io/_uploads/BknFFprh0.png)

Add the reward addresses of all cluster members. Then choose whether the contract is immutable (recommended option), whether to sponsor the maintainers of [splits.org](https://splits.org), and whether there is a distribution bounty so that third parties can distribute the rewards in exchange for a small fee.

![image](https://hackmd.io/_uploads/H1q0KaS20.png)

Finally, click the `Create Split` button, execute the transaction and share the created split contract with all cluster members for review.
<br><br><br><br>


# Part 2: Use Obol Launchpad + CLI to create the cluster

`Charon` is the software that enables validators to be run on a group of independent nodes - a cluster or squad. A complete multi-container `Docker` setup including execution client, consensus client, validator client, MEV-Boost and the `Charon` client can be found in this repository https://github.com/ObolNetwork/charon-distributed-validator-node. 
<br><br>

### Step 1: Clone the repo and add give $USER permissions

```
git clone https://github.com/ObolNetwork/charon-distributed-validator-node.git
```
```
sudo usermod -a -G docker $USER
```

You will then need to exit the ssh session and log in again.
<br><br>

### Step 2: Create ENR and Backup your Private Key

Change into the CDVN directory:
```
cd charon-distributed-validator-node
```
Use docker to create an ENR

```
docker run --rm -v "$(pwd):/opt/charon" obolnetwork/charon:v1.1.1 create enr
```
### Back up the private key located in `.charon/charon-enr-private-key`

![Screenshot 2024-10-18 at 12 28 23 PM](https://github.com/user-attachments/assets/6bf9f7ac-6b9f-4a8b-b15a-49b5d2cb3c56)
<br><br>

### Step 3: Create the DV cluster configuration using the Launchpad

With CSM launching soon, we've inegrated a custom CSM configuration into the launchpad. Choosing this configuration allows you to create up to 12 validator keys (CSM EA Limit) with Lido's required withdrawal and fee recipient addresses.

To start, the squad leader opens the [Holesky DV Launchpad](https://holesky.launchpad.obol.org), then connects  their wallet and chooses `Create a cluster with a group`.

![image](https://hackmd.io/_uploads/B1arp242R.png)

Then click `Get Started` button on the next page.

![chrome_WW8rTcaqeQ](https://hackmd.io/_uploads/H1LXijNnR.png)

Accepts all the necessary advisories and signs the confirmation.

![chrome_rQIeibZpcj](https://hackmd.io/_uploads/HkL7ioN20.png)

Cluster configuration begins here. First, select the cluster name and size. Then enters all cluster members signer addresses.

![Screenshot 2024-10-18 at 12 56 24 PM](https://github.com/user-attachments/assets/0ccd49fb-3faa-4752-8599-d2210db3daed)

Next, select the number of validators (up to 12 for CSM EA) to deploy. Enter the complete Public ENR which was generated during step 2 above, this includes the enr prefix (`enr:-HW4QLS48i.........`). In the `Withdrawal Configuration` field, select `LIDO CSM`. This will automatically fill in the required Withdrawal Address and Fee Recipient Addresss per [Lido Documentation](https://operatorportal.lido.fi/modules/community-staking-module#block-d8e94f551b2e47029a54e6cedea914a7) Finally, click on the `Create cluster configuration` button.

![Screenshot 2024-10-18 at 2 54 38 PM](https://github.com/user-attachments/assets/794176f5-3ea1-4669-bdfb-87b1aff36bc8)

Lastly, share the cluster configuration link with the other cluster members.

![chrome_wHk3z8Tz9S](https://hackmd.io/_uploads/By8XoiE2R.png)
<br><br>

### Step 4: Distributed Key Generation

All cluster members need to open the cluster invite link, connect their wallet, accept all necessary advisories, and verify the cluster configuration. Each squad member will need to input their ENR, so see steps 1 and 2 above.

![Screenshot 2024-10-18 at 3 00 47 PM](https://github.com/user-attachments/assets/2e759026-2376-4777-9765-c62081d218f0)


Once all members confirm the configuration they will see the `continue` button.

![image](https://hackmd.io/_uploads/HkHwc6VhC.png)

On the next page, they will find a CLI command.

![Screenshot 2024-10-18 at 3 17 33 PM](https://github.com/user-attachments/assets/a207ef9c-bd16-4b93-afbd-2a18f35f11de)


All members need to syncronisly complete this step. Go back to terminal, make sure you're in the correct directory:

```
cd charon-distributed-validator-node
```
After copying and pasting the command they should wait for all the other squad members to connect and complete the DKG ceremony.

![Screenshot 2024-10-18 at 1 10 25 PM](https://github.com/user-attachments/assets/55e7b010-d669-4c7f-86cd-f190c5bd9f3a)


A `cluster-lock.json` file is created in the `.charon` folder as well as the `deposit-data.json` file and the `validator_keys` folder. This contains each operator's partial key signatures for the validators.

**At this point, each operator must make a backup of the `.charon` folder and keep it safe, as validator keys can't be recreated.**
<br><br>


### Step 5: Configuring the MEV-boost client

To configure MEV-boost each cluster memeber needs to edit the `.env` file and set the `BUILDER_API_ENABLED=true` and `MEVBOOST_RELAYS=` to the URL of at least one of Lido's approved MEV relays [here](https://enchanted-direction-844.notion.site/6d369eb33f664487800b0dedfe32171e?v=985cb7e521de43d78c67b7ad29adec84). Multiple relays must be separated by a comma. **The use of unapproved relays is strictly forbidden! 
<br><br>

### Step 6: Starting the Node

Each cluster member should start the node with the following command:

```
docker compose up -d
```

At this point, execution and consensus clients should start syncing, and Charon and the validator client should start waiting for the validator to be activated. 
<br><br><br><br>

# Part 3: Deploy the keys to Lido CSM

CSM is launching with a whitelisted set of apporved operators (Early Access). The squad member with EA should be the one to create the node through the CSM wiget. 
<br>
EA member will head to [CSM Extended Mode](https://csm.testnet.fi/?mode=extended) and connect their wallet. Note the `mode=extended` parameter. This allows the Lido CSM reward address to be set to the split contract created earlier. 

![image](https://hackmd.io/_uploads/HkNhlaHhA.png)

EA member clicks on the `Create Node Operator` button...

![image](https://hackmd.io/_uploads/BkVsfpr2R.png)

EA member pastes the contents of the `deposit-data.json` file into the `Upload deposit data` field. EA member should have enough ETH/stETH/wstETH  to cover the bond.

![image](https://hackmd.io/_uploads/BJG44pH30.png)

Expand the `Specify custom addresses` section...

![image](https://hackmd.io/_uploads/r1z4X6Bh0.png)

Set the `Reward Address` field to the `Split` contract address and the `Manager Address` field to the `Safe` wallet address. Verify that the `Extended` box is outlined. This ensures that the `Safe` address has the ability to change the reward address if necessary. Check that the correct addresses are set and click the `Create Node Operator` button.

![Screenshot 2024-10-18 at 3 41 06 PM](https://github.com/user-attachments/assets/a269104d-e298-4dd0-bf52-24311d05b956)


Sign the transaction, and the cluster is ready for deposit from Lido CSM.

![image](https://hackmd.io/_uploads/HJwzJRShA.png)

## Monitoring the CSM operator

You can follow [this](https://dvt-homestaker.stakesaurus.com/bonded-validators-setup/lido-csm/monitoring-and-address-management) guide for the steps required to monitor your CSM operator.

## Exiting Validators

You can follow the Obol [launchpad](https://holesky.launchpad.obol.org/cluster/exit/) instructions on how to exit the cluster validators.
