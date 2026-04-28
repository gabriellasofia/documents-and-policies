# APMs Allowed List

```markdown!
STATUS: V1.5
```

## A – Allowed APMs & Configurations
This allow list is the registry of the [Auxiliary Proposer Mechanisms](https://github.com/sssngth/documents-and-policies/blob/BlockProposerRewardsV3/Lido%20Ethereum%20Block%20Proposer%20Rewards%20Policy.md#a4--auxiliary-proposer-mechanisms), implementations, associated parameters and values, that have been assessed and vetted by the community as sufficiently de-risked and demonstrably beneficial to both Lido and the broader Ethereum ecosystem. Maintained by the [APM Committee](https://research.lido.fi/t/establishing-the-apm-committee/9998), this list is regularly revised and updated to adequately reflect the rapidly evolving landscape of software offerings and network conditions.

### A.1 – Out-Of-Protocol Proposer-Builder Separation

#### A.1.1 – MEV-Boost Protocol & Builder API
Software|Configuration Location|Parameter|Value
---|---|---|---
[Commit-Boost Client PBS Module](https://github.com/Commit-Boost/commit-boost-client)|Sidecar|[`relays`](https://commit-boost.github.io/commit-boost-client/get_started/configuration)|[MEV Boost Relay Allowed List](https://etherscan.io/address/0xf95f069f9ad107938f6ba802a3da87892298610e#readContract#F4)
||Sidecar|[`min_bid_eth`](https://github.com/Commit-Boost/commit-boost-client/blob/f776bca71db413ab0a0d83488cbcdbe96a5216c8/config.example.toml#L43)|[`0.07`](https://research.lido.fi/t/lido-node-operator-mev-boost-min-bid-guidance/3347)
[MEV-Boost](https://github.com/flashbots/mev-boost)|Sidecar|[`relay`](https://github.com/flashbots/mev-boost/blob/d65d507f1133614aa650e169df27eeb12845a466/README.md?plain=1#L257),[`relays`](https://docs.flashbots.net/flashbots-mev-boost/getting-started/usage)|[MEV Boost Relay Allowed List](https://etherscan.io/address/0xf95f069f9ad107938f6ba802a3da87892298610e#readContract#F4)
||Sidecar|[`min-bid`](https://github.com/flashbots/mev-boost/blob/d65d507f1133614aa650e169df27eeb12845a466/README.md?plain=1#L255)|[`0.07`](https://research.lido.fi/t/lido-node-operator-mev-boost-min-bid-guidance/3347)
[Grandine](https://github.com/grandinetech/grandine)|Validator Client|[`suggested-fee-recipient`](https://docs.grandine.io/cli_options.html)|[Execution Layer Rewards Vault](https://docs.lido.fi/deployed-contracts/)
||Validator Client|[`default_builder_boost_factor`](https://github.com/grandinetech/grandine/blob/c56a555ed422a1260bef421c616719b8c0a1388f/grandine/src/grandine_config.rs#L48)|[`100`](https://research.lido.fi/t/lido-node-operator-mev-boost-min-bid-guidance/3347)
||Consensus Client|[`builder-api-url`](https://docs.grandine.io/builder_api_and_mev.html)|[MEV Boost Relay Allowed List](https://etherscan.io/address/0xf95f069f9ad107938f6ba802a3da87892298610e#readContract#F4)
[Lighthouse](https://github.com/sigp/lighthouse)|Validator Client|[`suggested-fee-recipient`](https://lighthouse-book.sigmaprime.io/validator_fee_recipient.html#how-to-configure-a-suggested-fee-recipient)|[Execution Layer Rewards Vault](https://docs.lido.fi/deployed-contracts/)
||Validator Client|[`builder-boost-factor`](https://lighthouse-book.sigmaprime.io/advanced_builders.html#how-to-connect-to-a-builder)|[`100`](https://research.lido.fi/t/lido-node-operator-mev-boost-min-bid-guidance/3347)
||Consensus Client|[`builder`](https://lighthouse-book.sigmaprime.io/advanced_builders.html#how-to-connect-to-a-builder)|[MEV Boost Relay Allowed List](https://etherscan.io/address/0xf95f069f9ad107938f6ba802a3da87892298610e#readContract#F4)
[Lodestar](https://github.com/ChainSafe/lodestar)|Validator Client|[`suggestedFeeRecipient`](https://chainsafe.github.io/lodestar/run/validator-management/vc-configuration#configuring-the-fee-recipient-address)|[Execution Layer Rewards Vault](https://docs.lido.fi/deployed-contracts/)
||Validator Client|[`builder.boostFactor`](https://chainsafe.github.io/lodestar/run/validator-management/vc-configuration#configure-your-builder-selection-andor-builder-boost-factor)|[`100`](https://research.lido.fi/t/lido-node-operator-mev-boost-min-bid-guidance/3347), `maxprofit`
||Consensus Client|[`builder.url`](https://chainsafe.github.io/lodestar/run/beacon-management/mev-and-builder-integration#configure-lodestar-setup-for-mev), [`builder.urls`](https://github.com/ChainSafe/lodestar/blob/ac2653dd8b767a24f9b1580977f8f560b1e3352d/packages/cli/src/options/beaconNodeOptions/builder.ts#L38)|[MEV Boost Relay Allowed List](https://etherscan.io/address/0xf95f069f9ad107938f6ba802a3da87892298610e#readContract#F4)
[Nimbus](https://github.com/status-im/nimbus-eth2)|Validator Client|[`suggested-fee-recipient`](https://nimbus.guide/suggested-fee-recipient.html)|[Execution Layer Rewards Vault](https://docs.lido.fi/deployed-contracts/)
||Beacon Node|[`local-block-value-boost`](https://nimbus.guide/options.html)|[`0`](https://research.lido.fi/t/lido-node-operator-mev-boost-min-bid-guidance/3347)
||Consensus Client|[`payload-builder-url`](https://nimbus.guide/external-block-builder.html)|[MEV Boost Relay Allowed List](https://etherscan.io/address/0xf95f069f9ad107938f6ba802a3da87892298610e#readContract#F4)
[Prysm](https://github.com/OffchainLabs/prysm)|Validator Client|[`suggested-fee-recipient`](https://www.offchainlabs.com/prysm/docs/execution-node/fee-recipient/)|[Execution Layer Rewards Vault](https://docs.lido.fi/deployed-contracts/)
||Validator Client|[`local-block-value-boost`](https://www.offchainlabs.com/prysm/docs/advanced/builder/#prioritizing-local-blocks)|[`0`](https://research.lido.fi/t/lido-node-operator-mev-boost-min-bid-guidance/3347)
||Consensus Client|[`http-mev-relay`](https://www.offchainlabs.com/prysm/docs/advanced/builder/#2-beacon-node-connect-to-the-builder)|[MEV Boost Relay Allowed List](https://etherscan.io/address/0xf95f069f9ad107938f6ba802a3da87892298610e#readContract#F4)
[Teku](https://github.com/Consensys/teku)|Validator Client|[`validators-proposer-default-fee-recipient`](https://docs.teku.consensys.io/reference/cli#validators-proposer-default-fee-recipient), [`fee_recipient`](https://docs.teku.consensys.io/how-to/configure/use-proposer-config-file)|[Execution Layer Rewards Vault](https://docs.lido.fi/deployed-contracts/)
||Beacon Node|[`builder-bid-compare-factor`](https://docs.teku.consensys.io/reference/cli#builder-bid-compare-factor)|[`100`](https://research.lido.fi/t/lido-node-operator-mev-boost-min-bid-guidance/3347)
||Consensus Client|[`builder-endpoint`](https://docs.teku.consensys.io/reference/cli#builder-endpoint)|[MEV Boost Relay Allowed List](https://etherscan.io/address/0xf95f069f9ad107938f6ba802a3da87892298610e#readContract#F4)
[Vero](https://github.com/serenita-org/vero)|Multi-Beacon Node Validator Client|[`fee-recipient`](https://github.com/serenita-org/vero/blob/master/docs/running_vero.md#--fee-recipient)|[Execution Layer Rewards Vault](https://docs.lido.fi/deployed-contracts/)
||Multi-Beacon Node Validator Client|[`builder-boost-factor`](https://github.com/serenita-org/vero/blob/master/docs/running_vero.md#--builder-boost-factor)|[`100`](https://research.lido.fi/t/lido-node-operator-mev-boost-min-bid-guidance/3347)
[Vouch](https://github.com/attestantio/vouch)|Multi-Beacon Node Validator Client|[`fallback-fee-recipient`](https://github.com/attestantio/vouch/blob/master/docs/configuration.md?plain=1#L226)|[Execution Layer Rewards Vault](https://docs.lido.fi/deployed-contracts/)
||Multi-Beacon Node Validator Client|[`builder-boost-factor`](https://github.com/attestantio/vouch/blob/master/docs/configuration.md?plain=1#L94)|[`100`](https://research.lido.fi/t/lido-node-operator-mev-boost-min-bid-guidance/3347)
[Charon](https://github.com/ObolNetwork/charon)|Distributed Validator Client|[`fee_recipient_address`](https://docs.obol.org/sdk/type-aliases/clustervalidator)|[Execution Layer Rewards Vault](https://docs.lido.fi/deployed-contracts/)
||Distributed Validator Client|[`builder_boost_factor`](https://github.com/ObolNetwork/charon/blob/3c374e3995b9228d159ce9718b88a8e23b13e9d1/docs/architecture.md?plain=1#L145C1-L145C98)|[`100`](https://research.lido.fi/t/lido-node-operator-mev-boost-min-bid-guidance/3347)
[SSV Node](https://github.com/ssvlabs/ssv)|Distributed Validator Middleware (alternatively via [WebApp](https://docs.ssv.network/stakers/validator-management/setting-fee-recipient-address/))|[`feeRecipientAddress`](https://docs.ssv.network/developers/smart-contracts/ssvnetwork#setfeerecipientaddressfeerecipientaddress)|[Execution Layer Rewards Vault](https://docs.lido.fi/deployed-contracts/)
