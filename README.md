# Calimero<sup>TM</sup> Network: The Private Sharding and Privacy Toolkit

Calimero<sup>TM</sup> is a customizable sidechain that is built upon leveraging the NEAR protocol and provides ***private sharding***. Secure private shard infrastructure lets you protect your data while leveraging open-source blockchains' business benefits like being **high-performant**, **incredibly secure**, **fast**, and having **infinite scalability**, all while supporting **sustainability**.

Access our official [Documentation](https://docs.calimero.network/) & Claimero Bridge [CodeBase](https://github.com/calimero-is-near/bridge-contracts) for more insights.

Currently, Calimero<sup>TM</sup> Network features include:
- `Calimero Beacon`
    - This management console allows users to quickly ship their applications without having to manage their own infrastructure.
- `Calimero Borealis`
    - The Borealia allows you to launch a private EVM-Compatible Shard interoperable with both Ethereum and Aurora.

- `Calimero Genesis`
    - Genesis bootstraps a private shard with a single unique command that generates a customizable shard configuration for development, testing, or deploying into production.
- `Calimero Hive`
    - Hive installs applications and plugins for the marketplace immediately out of the box, without any requirements of coding
- `Calimero Ghost`
    - Ghost toolkit provides the developers the ability to generate and build private smart contracts between parties using encryption and zero-knowledge techniques
- `Calimero SDK`
    - Calimero SDK is NEAR's intuitive developer tooling that lets you build contracts in common languages like Rust, JavaScript or AssemblyScript
## Calimero's Self-Assessment
After you've gone through the official documentations and gathered knowledge, we'd like you to complete an assessment to test and get your fundamentals right and strong.
Please Access the **Assessment Here** and the **Summaries Here** 
# Developer Quick Start
These instructions are intended for those wishing to examine the Synthea source code, extend it or build the code locally. 

## Installation
To clone the Synthea<sup>TM</sup> repo:
```
  git clone https://github.com/calimero-is-near/bridge-contracts.git
```

## Bridge Service Workflow
![08](https://docs.calimero.network/assets/images/bridge_full-06a9ac4aa03ea36757f1824efe4cc4bd.jpg)

## Wallet Setup
To clone the NEAR wallet repository
```
git clone https://github.com/calimero-is-near/near-wallet.git
```
- rellocate to the front-end directory
```
~ % cd (.../home/..) near-wallet/packages/frontend 
```
- run the wallet frontend on https://localhost:1234/

```
~ % yarn && NEAR_WALLET_ENV=testnet yarn start
```
More detailed instructions for wallet setup can be found at [wallet-docs](https://github.com/calimero-is-near/near-wallet/blob/master/README.md)
## Setting Light Client
The light client contract will accept block headers that are being relayed to it if the blocks meet certain validation criteria
```
cd contracts/light_client
./build.sh
```

Deploying the contract on NEAR Testnet and Calimero network
```
./deploy.sh
./deploy_calimero.sh calimero_shard_name
```

## Setting Prover Contracts
The prover takes as input the proof data that contain a merkle path to the block where the transaction/receipt originated and a merkle path to the transaction/receipt, also as input the height of the known block to the light client contract needs to be provided, and this block needs to be ahead or on the block of the transaction that we are proving.
- Prerequisite for deploying the prover contract: the `light_client contract` is already deployed

```
cd contracts/prover
./build.sh
./deploy.sh
./deploy_calimero.sh calimero_shard_id
```

## Prerequisites for running the Calimero Bridge
1. A Calimero shard needs to be running ([Contact Us](https://www.calimero.network/contact) if you want to spin up your own private shard)
2. All contracts need to be deployed on both NEAR and Calimero
3. Relayer from Near to Calimero and a relayer from Calimero to Near need to be running
4. Bridge service that monitors events in realtime on both Near and Calimero need to be running

### Calimero Alias
You can setup a helper alias for [near-cli](https://docs.near.org/tools/near-cli) when interacting with a Calimero shard:
```
alias calimero='function x() { near ${@:2} --nodeUrl https://api.development.calimero.network/api/v1/shards/$1/neard-rpc --networkId $1;} ; x'
```

## Setting Connector Contracts
Calimero supports transfering Fungible tokens as well as Non Fungible tokens from one chain to another. Also, via the Calimero bridge cross shard calls can be executed.

Three Connector Contracts are:
1. Fungible Connector Contracts
2. Non-Fungible Connector Contracts
3. Cross Shard Connector Call Contracts

### FT Connector
With the fungible token connector ft's can be bridged from NEAR to Calimero and back. In order to bridge some ft from NEAR testnet to Calimero, a single transaction needs to be called. Just lock the wanted amount of tokens to the ft connector contract. Once they are transferred, the bridge service and the relayer will be notified about it and try to prove on the Calimero shard that the locking of tokens happened on NEAR. If proved, wrapped tokens are minted on Calimero shard.
```
near call usdn.testnet ft_transfer_call --args ‘{“receiver_id”:“ft_source_connector.cali99.apptest-development.testnet”,“amount”:“12345",“msg”:“”}’ --accountId igi.testnet --depositYocto 1 --gas 3000000000000
```
If the users want to get the tokens back on the original chain (in this case NEAR testnet), they simply call withdraw on the bridged token on the other chain (in this case Calimero shard) which will essentially burn tokens.
```
calimero cali99-calimero-testnet call usdn.ft_dest_connector.cali99.calimero.testnet withdraw --args '{"amount":"345"}' --accountId igi.testnet --depositYocto 1 --gas 300000000000000
```

- Prerequisite for deploying the connectors: the prover contract on each chain is already deployed

```
cd contracts/ft_bridge_token
./build.sh
./deploy.sh
cd ../bridge_token_deployer
./build_ft.sh
cd ../ft_connector
./build.sh
```

Deploy on both NEAR testnet and Calimero:

```
./deploy.sh
./deploy_calimero.sh calimero_shard_id
```

### NFT Connector
With the non fungible token connector NFTs can be bridged from NEAR to Calimero and back. In order to bridge some NFT from NEAR testnet to Calimero, a single transaction needs to be called. Just lock the wanted token to the nft connector contract. Once the token is transferred, the bridge service will be notified about it and try to prove on the Calimero shard that the locking of tokens happened on NEAR. If proved, wrapped token is minted on Calimero shard.

```
near call nft-test.igi.testnet nft_transfer_call --args '{"receiver_id":"nft_source_connector.cali99.apptest-development.testnet", "token_id":"0", "msg":""}' --accountId igi.testnet --depositYocto 1 --gas 300000000000000
```


If the users want to get the token back on the original chain (in this case NEAR testnet), they simply call withdraw on the bridged token on the other chain (in this case Calimero shard) which will essentially burn the token.
```
calimero cali99-calimero-testnet call nft-test_igi.nft_dest_connector.cali99.calimero.testnet withdraw --args '{"token_id":"0"}' --accountId igi.testnet --depositYocto 1 --gas 300000000000000
```

- Prerequisite for deploying the nft connectors: that the prover contract on each chain is already deployed
```
cd contracts/nft_bridge_token
./build.sh
./deploy.sh
cd ../bridge_token_deployer
./build_nft.sh
cd ../nft_connector
./build.sh
```
Deploy on both NEAR testnet and Calimero:

```
./deploy.sh
./deploy_calimero.sh calimero_shard_id
```