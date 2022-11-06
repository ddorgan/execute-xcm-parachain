# Execute XCM Parachain Setup

A parachain setup for testing the use of `pallet_xcm::execute` based on the parachain template.


## Run Example

### Create chainspec container

```
kubectl create -f https://raw.githubusercontent.com/paritytech/testnet-manager/main/local-kubernetes/kube-setup/validators-chainspec.yml
```
### Deploy testing network

```
helmfile sync
```

### Onboard paras via gui or cmd line:

```
curl -X 'POST' 'http://localhost:8080/api/onboard_parachain/2000' -H 'accept: application/json' -d ''
curl -X 'POST' 'http://localhost:8080/api/onboard_parachain/3000' -H 'accept: application/json' -d ''
```

### Setup hrmp channels

```
yarn run:api tx.parasSudoWrapper.sudoEstablishHrmpChannel 2000 3000 8 1024  --ws ws://127.0.0.1:9944 --seed "//Alice" --sudo
yarn run:api tx.parasSudoWrapper.sudoEstablishHrmpChannel 3000 2000 8 1024  --ws ws://127.0.0.1:9944 --seed "//Alice" --sudo
```

### Transfer asset from parachain 2k to parachain 3k:

```
yarn run:api tx.polkadotXcm.limitedReserveTransferAssets '{"v1":{"parents":1,"interior":{"x1":{"parachain":3000}}}}' '{"v1":{"parents":0,"interior":{"x1":{"AccountId32": {"id": "0x90b5ab205c6974c9ea841be688864633dc9ca8a357843eeacf2314649965fe22", "network": "Any"}}}}}' '{"v1": [ {"id": { "Concrete": {"parents":0, "interior":{"x2": [ {"PalletInstance":12}, {"GeneralIndex":0}] }}}, "Fun": { "Fungible": "320008009"}}]}' 0 Unlimited  --ws ws://127.0.0.1:9945 --seed "//Alice"
```

### Result

See ```assets.Issued``` to ```Charlie``` on parachain 3k. 


## Setup

Here is how to get this parachain up and running:
```sh
# clone this repo
git clone https://github.com/apopiak/execute-xcm-parachain.git
# clone the polkadot repo as a sibling to this repo, which is where polkadot-launch expects it
git clone --depth 1 --branch release-v0.9.29 https://github.com/paritytech/polkadot.git
# build it
pushd polkadot
cargo build --release
popd
# go into our repo
cd execute-xcm-parachain
# install polkadot-launch
yarn
# build the node
cargo build --release
# run the setup, will spin up the relay chain and two basically identical parachains
yarn run polkadot-launch polkadot-launch/config.json
```

Connect to the parachains with polkadot-js apps at `ws://127.0.0.1:9988` (parachain 2000) and
 `ws://127.0.0.1:9999` (parachain 3000).

# Usage
The following limited reserve transfer should work from parachain 2000 to parachain 3000:
```ts
dest: XcmVersionedMultiLocation
{
  V1: {
    parents: 1
    interior: {
      X1: {
        Parachain: 3,000
      }
    }
  }
}
beneficiary: XcmVersionedMultiLocation
{
  V1: {
    parents: 0
    interior: {
      X1: {
        AccountId32: {
          network: Any
          id: 0x8eaf04151687736326c9fea17e25fc5287613693c912909cb226aa4794f26a48
        }
      }
    }
  }
}
assets: XcmVersionedMultiAssets
{
  V1: [
    {
      id: {
        Concrete: {
          parents: 0
          interior: {
            X2: [
              {
                PalletInstance: 12
              }
              {
                GeneralIndex: 0
              }
            ]
          }
        }
      }
      fun: {
        Fungible: 33,333
      }
    }
  ]
}
feeAssetItem: u32
0
weightLimit: XcmV2WeightLimit
Unlimited
```

A new [Cumulus](https://github.com/paritytech/cumulus/)-based Substrate node, ready for hacking ☁️..

This project is originally a fork of the
[Substrate Node Template](https://github.com/substrate-developer-hub/substrate-node-template)
modified to include dependencies required for registering this node as a **parathread** or
**parachain** to a **relay chain**.

The stand-alone version of this template is hosted on the
[Substrate Devhub Parachain Template](https://github.com/substrate-developer-hub/substrate-parachain-template/)
for each release of Polkadot. It is generated directly to the upstream
[Parachain Template in Cumulus](https://github.com/paritytech/cumulus/tree/master/parachain-template)
at each release branch using the
[Substrate Template Generator](https://github.com/paritytech/substrate-template-generator/).

👉 Learn more about parachains [here](https://wiki.polkadot.network/docs/learn-parachains), and
parathreads [here](https://wiki.polkadot.network/docs/learn-parathreads).


🧙 Learn about how to use this template and run your own parachain testnet for it in the
[Devhub Cumulus Tutorial](https://docs.substrate.io/tutorials/v3/cumulus/start-relay/).
