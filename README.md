# amsterDOT Acala Workshop

Docs: https://evmdocs.acala.network

## Get the basic running

- Prepare
  - Use Github Codespaces
  - OR use VSCode devcontainer
  - OR install deps by following instructions [here](#3-building)
- Run a local instant sealing devnet
  - `make run-eth`
  - OR use docker
    - `docker run --rm -p 9944:9944 -p 9933:9933 ghcr.io/acalanetwork/mandala-node:sha-ce49e64 --dev -levm=debug --instant-sealing --ws-external --rpc-external --rpc-cors=all`
- Open the polkadot-js apps
  - https://polkadot.js.org/apps/#/explorer
  - Choose local network
- Run `eth-rpc-adapter`
  - `LOCAL_MODE=1 npx @acala-network/eth-rpc-adapter@2.4.6`
- Setup Metamask:
  - Mnemonic: `fox sight canyon orphan hotel grow hedgehog build bless august weather swarm`
  - Add new network
    - Name: Mandala Local
    - RPC: http://localhost:8545
    - Chain ID: 595
    - Symbol: ACA
- Try send some ACA with Metamask
  - Note: Please don't modify gas price or it could fail
- Remix
  - Use injected Web3 provider
  - Create new workspace
  - Try deploy contracts
  - Modify gas price to `201.92480561` GWEI if contract deployment failed
    - Refer to https://evmdocs.acala.network/reference/common-errors for common errors

## Call EVM from Pallet

- Deploy ERC20 and use it with pallets
  - Deploy the [erc20.sol](./workshop-files/erc20.sol) using Remix
- Bind EVM account
  - Use https://evm.acala.network/#/Bind%20Account
    - Substrate address: `5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty`
    - Chain ID: 595
    - Genesis hash: Query from https://polkadot.js.org/apps/#/explorer/query/0
    - Sign
  - Use the extrinsic page: https://polkadot.js.org/apps/#/extrinsics
    - Account: Bob
    - evmAccounts.claimAccount
      - Address: `0x75E480dB528101a381Ce68544611C169Ad7EB342`
      - Signature: The signature
    - Submit
    - Check event to confirm account merge
- Transfer ERC20 from Substrate pallet
  - Extrinsic page
    - Account: Bob
    - currencies.transfer
      - Dest: Charlie
      - CurrencyId: Erc20(the erc20 address)
      - Amount: the amount
    - Submit
    - Check event to confirm transfer
- Behind the scenes:
  - https://github.com/xlc/amsterdot-workshop/blob/719a62365a0f5ab6d8556d05a741b887c0b2df42/modules/currencies/src/lib.rs#L338-L353
  - https://github.com/xlc/amsterdot-workshop/blob/719a62365a0f5ab6d8556d05a741b887c0b2df42/modules/evm-bridge/src/lib.rs#L172-L203

## Call Pallet from EVM

- Add ACA Mirrow Token `0x0000000000000000000100000000000000000000` to MetaMask
- Transfer the ERC20
  - The Native balance will also change
  - Native balance.transfer event will also be emitted
- Predeploy contracts: https://github.com/AcalaNetwork/predeploy-contracts
- Precompile contracts: https://github.com/xlc/amsterdot-workshop/blob/719a62365a0f5ab6d8556d05a741b887c0b2df42/runtime/common/src/precompile/mod.rs#L188-L267

## Build a Smart Contract Arena pallet

- Call
  - fn register(origin, contract: H160)
- Pallet
  - fn on_initialize
    - let game_points = BTreeMap::new()
    - for x in instances.filter(x => x.is_contract)
      - for y in instances
        - play(x, y)
        - if draw
          - game_points[x] += 1
          - game_points[y] += 1
        - else
          - game_points[winner] += 2
          - game_points[loser] -= 1
    - points = game_points sorted
    - remove loster instances
    - duplicate winner instances
    - add queued contracts
    - apply game_points to global point board
- Types
  - struct ContractInfo
    - point: u128
    - instance_count: u32
  - enum Contender
    - Contract(H160),
    - AlwaysZero,
    - BlockNumber,
    - CopyCat,
- Storages
  - ContenderInfos: Contender => ContractInfo
  - Contenders: u32 => Contender
  - ContentderInstancesCount: u32
  - NextContenderInstanceId: u32
  - ContenderQueue: Vec<Contender>
- Config
  - const PlayPerRound: u32
  - const MaxContenderInstancesCount: u32
  - const MaxQueueSize: u32
  - const WinnerCount: u32
  - const EnqueueCount: u32
  - const NewContenderPerGame: u32
  - const MaxInstances: u32
- interface IContender
  - owner() returns address
  - play(uint256 round, uint256 prevPlay, uint256 otherPrevPlay) returns uint256

## Copy the Echo contract

https://evmdocs.acala.network/tutorials/waffle-tutorials/echo-tutorial

- Repo: https://github.com/AcalaNetwork/waffle-tutorials
- `cd echo`
- `yarn`
- `yarn test`
- `yarn deploy`

## Implement and deploy your contender

- Simple
  - https://github.com/xlc/amsterdot-workshop/blob/master/workshop-files/arena/contracts/Simple.sol
- Random
  - https://github.com/xlc/amsterdot-workshop/blob/master/workshop-files/arena/contracts/Random.sol
- Advanced
  - https://github.com/xlc/amsterdot-workshop/blob/master/workshop-files/arena/contracts/Advanced.sol
- Deploy Locally
  - `yarn deploy`
- Deploy to testnet
  - `SEED="xxx xxx" yarn deploy:amsterdot`

## Check Game result

https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Famsterdot.aca-dev.network#/explorer

----

<p align="center">
  <img src="https://acala.subdao.com/logo/acala-logo-horizontal-gradient.png" width="460">
</p>

<div align="center">

[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/AcalaNetwork/Acala/Test?label=Actions&logo=github)](https://github.com/AcalaNetwork/Acala/actions?query=workflow%3ATest)
[![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/AcalaNetwork/Acala)](https://github.com/AcalaNetwork/Acala/tags)
[![Substrate version](https://img.shields.io/badge/Substrate-2.0.0-brightgreen?logo=Parity%20Substrate)](https://substrate.io/)
[![codecov](https://codecov.io/gh/AcalaNetwork/Acala/branch/master/graph/badge.svg?token=ERf7EDgafw)](https://codecov.io/gh/AcalaNetwork/Acala)
[![License](https://img.shields.io/github/license/AcalaNetwork/Acala?color=green)](https://github.com/AcalaNetwork/Acala/blob/master/LICENSE)
 <br />
[![Twitter URL](https://img.shields.io/twitter/url?style=social&url=https%3A%2F%2Ftwitter.com%2FAcalaNetwork)](https://twitter.com/AcalaNetwork)
[![Discord](https://img.shields.io/badge/Discord-gray?logo=discord)](https://discord.gg/vdbFVCH)
[![Telegram](https://img.shields.io/badge/Telegram-gray?logo=telegram)](https://t.me/AcalaOfficial)
[![Discourse](https://img.shields.io/badge/Forum-gray?logo=discourse)](https://acala.discourse.group/)
[![Medium](https://img.shields.io/badge/Medium-gray?logo=medium)](https://medium.com/acalanetwork)

</div>

<!-- TOC -->

- [amsterDOT Acala Workshop](#amsterdot-acala-workshop)
	- [Get the basic running](#get-the-basic-running)
	- [Call EVM from Pallet](#call-evm-from-pallet)
	- [Call Pallet from EVM](#call-pallet-from-evm)
	- [Build a Smart Contract Arena pallet](#build-a-smart-contract-arena-pallet)
	- [Copy the Echo contract](#copy-the-echo-contract)
	- [Implement and deploy your contender](#implement-and-deploy-your-contender)
	- [Check Game result](#check-game-result)
- [1. Introduction](#1-introduction)
- [2. Overview](#2-overview)
	- [2.1. aUSD and the Honzon stablecoin protocol](#21-ausd-and-the-honzon-stablecoin-protocol)
	- [2.2. Acala Network Economic Model](#22-acala-network-economic-model)
- [3. Building](#3-building)
	- [NOTE](#note)
- [4. Run](#4-run)
- [5. Development](#5-development)
- [6. Bug Bounty :bug:](#6-bug-bounty-bug)
- [7. Bench Bot](#7-bench-bot)
	- [Generate module weights](#generate-module-weights)
	- [Generate runtime weights](#generate-runtime-weights)
	- [Bench Acala EVM+](#bench-acala-evm)
- [8. Migration testing runtime](#8-migration-testing-runtime)
	- [Try testing runtime](#try-testing-runtime)
- [9. Run local testnet with parachain-launch](#9-run-local-testnet-with-parachain-launch)
- [10. Run local testnet with polkadot-launch](#10-run-local-testnet-with-polkadot-launch)
- [11. Build For Release](#11-build-for-release)

<!-- /TOC -->

# 1. Introduction
This project is initiated and facilitated by the Acala Foundation. Acala Foundation nurtures applications in the fields of decentralized finance protocols, particularly those that serve as open finance infrastructures such as stable currency and staking liquidity. The Acala Foundation is founded by [Laminar](https://laminar.one/) and [Polkawallet](https://polkawallet.io/) , participants and contributors to the Polkadot ecosystem. The Acala Foundation welcomes more industry participants as it progresses.

# 2. Overview
The significance of cross-chain communication to the blockchain is like that of the internet to the intranet. Polkadot empowers a network of public, consortium and private blockchains, and enables true interoperability, economic and transactional scalability. A cross-chain stablecoin system will:
1. create a sound, stable currency for low cost, borderless value transfer for all chains in the network
2. enable commerical lending with predictable risk
3. serve as a building block for more open finance services

The Acala Dollar stablecoin (ticker: aUSD) is a multi-collateral-backed cryptocurrency, with value stable against US Dollar (aka. 1:1 aUSD to USD soft peg). It is completely decentralized, that it can be created using assets from blockchains connected to the Polkadot network including Ethereum and Bitcoin as collaterals, and can be used by any chains (or digital jurisdictions) within the Polkadot network and applications on those chains.

By this nature, it is essential that the Acala Network eventually become community-owned with an economic model that can sustain its development and participation in the Polkadot network, as well as ensure its stability and security. The following section will provide a high-level overview of the following topics:
- aUSD and the Honzon stablecoin protocol
- the economic model and initial parachain offering

## 2.1. aUSD and the Honzon stablecoin protocol
Every aUSD is backed in excess by a crypto asset, the mechanism of which is known as an over-collateralized debt position (or CDP). Together with a set of incentives, supply & demand balancing, and risk management mechanisms, as the core components of the Honzon stablecoin protocol on the Acala Network, the stability of the aUSD is ensured. The CDP mechanism design is inspired by the first decentralized stablecoin project MakerDAO, which has become the DeFi building block in the Ethereum ecosystem. Besides, the Honzon protocol enables many unique features - native multi-asset support, cross-chain stablecoin capability, automatic liquidation to increase responsiveness to risk, and pluggable oracle and auction house to improve modularity, just to name a few.

The Honzon protocol contains the following components
- Multi Collateral Type
- Collateral Adapter
- Oracle and Prices
- Auction and Auction Manager
- CDP and CDP Engine
- Emergency shutdown
- Governance
- Honzon as an interface to other components

Note: This section is still work in progress, we will update more information as we progress. Refer to the [Github Wiki](https://github.com/AcalaNetwork/Acala/wiki) for more details.

## 2.2. Acala Network Economic Model
The Acala Network Token (ACA) features the following utilities, and the value of ACA token will accrue with the increased usage of the network and revenue from stability fees and liquidation penalties
1. As Network Utility Token: to pay for network fees and stability fees
2. As Governance Token: to vote for/against risk parameters and network change proposals
3. As Economic Capital: in case of liquidation without sufficient collaterals

To enable cross-chain functionality, the Acala Network will connect to the Polkadot in one of the three ways:
1. as parathread - pay-as-you-go connection to Polkadot
2. as parachain - permanent connection for a given period
3. as an independent chain with a bridge back to Polkadot

Becoming a parachain would be an ideal option to bootstrap the Acala Network, and maximize its benefits and reach to other chains and applications on the Polkadot network. To secure a parachain slot, the Acala Network will require supportive DOT holders to lock their DOTs to bid for a slot collectively - a process known as the Initial Parachain Offering (IPO). ACA tokens will be offered as a reward for those who participated in the IPO, as compensation for their opportunity cost of staking the DOTs.

Note: This section is still work in progress, we will update more information as we progress. Refer to the [token economy working paper](https://github.com/AcalaNetwork/Acala-white-paper) for more details.

# 3. Building

## NOTE

To connect on the "Mandala TC6" network, you will want the version `~0.7.10` code which is in this repo.

- **Mandala TC6** is in [Acala repo master branch](https://github.com/AcalaNetwork/Acala/tree/master/).

Install Rust:

```bash
curl https://sh.rustup.rs -sSf | sh
```

You may need additional dependencies, checkout [substrate.io](https://docs.substrate.io/v3/getting-started/installation) for more info

```bash
sudo apt-get install -y git clang curl libssl-dev llvm libudev-dev
```

Make sure you have `submodule.recurse` set to true to make life with submodule easier.

```bash
git config --global submodule.recurse true
```

Install required tools and install git hooks:

```bash
make init
```

Build Mandala TC native code:

```bash
make build-full
```

# 4. Run

You can start a development chain with:

```bash
make run
```

# 5. Development

To type check:

```bash
make check-all
```

To purge old chain data:

```bash
make purge
```

To purge old chain data and run

```bash
make restart
```

Update ORML

```bash
make update
```

__Note:__ All build command from Makefile are designed for local development purposes and hence have `SKIP_WASM_BUILD` enabled to speed up build time and use `--execution native` to only run use native execution mode.

# 6. Bug Bounty :bug:

The Bug Bounty Program includes only on-chain vulnerabilities that can lead to significant economic loss or instability of the network. You check details of the Bug Bounty or Submit a vulnerability here:
https://immunefi.com/bounty/acala/

# 7. Bench Bot
Bench bot can take care of syncing branch with `master` and generating WeightInfos for module or runtime.

## Generate module weights

Comment on a PR `/bench runtime module <module_name>` i.e.: `module_currencies`

Bench bot will do the benchmarking, generate `weights.rs` file push changes into your branch.

## Generate runtime weights

Comment on a PR `/bench runtime <runtime> <module_name>` i.e.: `/bench runtime mandala module_currencies`.

To generate weights for all modules just pass `*` as `module_name` i.e: `/bench runtime mandala *`

Bench bot will do the benchmarking, generate weights file push changes into your branch.

## Bench Acala EVM+

Comment on a PR `/bench evm` to benchmark Acala EVM+ and bench bot will generate precompile weights and GasToWeight ratio.


# 8. Migration testing runtime
If modify the storage, should test the data migration before upgrade the runtime.

## Try testing runtime

try-runtime on karura

```bash
# Use a live chain to run the migration test and save state snapshot to file `snapshot.bin`.
# Add `-m module_name` can specify the module.
cargo run --features with-karura-runtime --features try-runtime -- try-runtime --chain=karura-dev --wasm-execution=compiled on-runtime-upgrade live --uri wss://karura.api.onfinality.io:443/public-ws --at=0x9def608d5674f6d16574f53849218fe13d80ec1042ef7c2d4de7d4c50abab806 -s /tmp/snapshot.bin

 # Use a state snapshot to run the migration test.
cargo run --features with-karura-runtime --features try-runtime -- try-runtime --chain=karura-dev --wasm-execution=compiled on-runtime-upgrade snap -s /tmp/snapshot.bin
```

try-runtime on acala

```bash
cargo run --features with-acala-runtime --features try-runtime -- try-runtime --chain=acala-dev on-runtime-upgrade live --uri wss://acala-polkadot.api.onfinality.io:443/public-ws -s /tmp/snapshot.bin

cargo run --features with-acala-runtime --features try-runtime -- try-runtime --chain=acala-dev on-runtime-upgrade snap -s /tmp/snapshot.bin
```

# 9. Run local testnet with [parachain-launch](https://github.com/open-web3-stack/parachain-launch)
Build RelayChain and Parachain local testnet to develop.

```bash
cd launch

# install dependencies
yarn

# generate docker-compose.yml and genesis
# NOTE: If the docker image is not the latest, need to download it manually.
# e.g.: docker pull acala/karura-node:latest
yarn run start generate

# start relaychain and parachain
cd output
# NOTE: If regenerate the output directory, need to rebuild the images.
docker-compose up -d --build

# list all of the containers.
docker ps -a

# track container logs
docker logs -f [container_id/container_name]

# stop all of the containers.
docker-compose stop

# remove all of the containers.
docker-compose rm

# NOTE: If you want to clear the data and restart, you need to clear the volumes.
# remove volume
docker volume ls
docker volume rm [volume_name]
# prune all volumes
docker volume prune
```

# 10. Run local testnet with [polkadot-launch](https://github.com/paritytech/polkadot-launch)

copy acala related launch json to polkadot-launch:

```bash
# $polkadot-launch is the home of polkadot-launch
cp scripts/polkadot-launch/*.json $polkadot-launch/
```

build polkadot:

```bash
git clone -n https://github.com/paritytech/polkadot.git
cargo build --release
cp target/release/polkadot /tmp/polkadot
```

build karura:

```bash
make build-karura-release
```

launch polkadot and parachain with json config file in polkadot-launch:

```bash
polkadot-launch acala-launch.json
```

there're other json file that will run both karura and other parachain.
- scripts/polkadot-launch/acala-statemine.json: run karura and statemine
- scripts/polkadot-launch/acala-bifrost.json: run karura and bifrost

# 11. Build For Release

For release artifacts, a more optimized build config is used.
This config takes around 2x to 3x longer to build, but produces an more optimized binary to run.

```bash
make build-release
```
