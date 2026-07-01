# Lido on Ethereum Standard Node Operator Protocol – Validator Exits

```markdown!
STATUS: V3.0
```

## A – Purpose
This Standard Node Operator Protocol (SNOP) outlines
* the general validator exit mechanisms available in Ethereum (see [Appendix A](#appendix-a--ethereum-validator-exit-mechanisms)),
* the role that validator exits play in Lido on Ethereum (hereinafter, "Lido"),
* the scope and rules around the usage of this SNOP,
* the algorithmic validator exit order in Lido,
* the framework for permissionless triggerable withdrawals (TWs) of validators from the protocol,
* the responsibilities Node Operators (NOs) using Lido have in processing validator exit requests, and
* the actions that NOs can expect in case of non-conformance with Lido and this SNOP.

In Lido, validator exits may be utilized for the following reasons:
* To satisfy unfinalized withdrawal requests from users,
* To allow permissionless NOs exits at their discretion,
* To reallocate stake across the NO set — e.g., to maintain NO target, and Staking Module (SM) share limits,
* To rotate signing keys,
* To eject validators regardless of demand in the [Withdrawal Queue (WQ)](https://docs.lido.fi/contracts/withdrawal-queue-erc721), if
  * the rules and expectations of Lido and this SNOP are disregarded,
  * a NO in a permissionless SM underperforms for an extended period of time,
  * there's conflict between parties of a joint permissionless operation,
  * the access to signing keys is lost, or
  * the security of signing keys is compromised.

## B – Scope
This SNOP applies to the Lido protocol, the NOs who use any of the [Staking Router (SR)'s](https://docs.lido.fi/contracts/staking-router) existing SMs — [Curated Module (CM)](https://operatorportal.lido.fi/modules/curated-module), [Simple Distributed Validator Technology Module (SDVTM)](https://operatorportal.lido.fi/modules/simple-dvt-module), and [Community Staking Module (CSM)](https://operatorportal.lido.fi/modules/community-staking-module) — any future SM, and the validators the NOs run as part of the protocol.

The following operational flows and expectations apply to all above-mentioned sources, except where alternative processes or expectations are defined due to technical or operational differences.

_NOTE: Terminology may vary in meaning depending on context. The below table aims to clarify the usage of key terms for the purpose of this SNOP and within the context of Lido._

Term|Definition|Context
:---:|---|---
Node Operator|An entity, individual, or cluster thereof that operates nodes using any of the SR's SMs or staking avenues to run Ethereum validators|_Solo_:<br>A single entity or individual that operates validators<br><br>_Cluster_:<br>\* An intra-operator cluster by a single entity or individual utilizing [distributed validator technology (DVT)](https://ethereum.org/en/staking/dvt/#distributed-validator-technology) that operates distributed validators (DVs), or<br>\* An inter-operator cluster by multiple entities and/or individuals utilizing DVT that operates DVs
Validator|A validator — also referred to as a "key" in this SNOP — is the technical component of participating in Ethereum's consensus. If active, it represents an [effective balance](https://eth2book.info/capella/part2/incentives/balances/) ranging from 16 up to 2048 ETH (in 1 ETH increments), according to their corresponding [0x01 or 0x02 withdrawal credentials](https://ethereum.org/en/roadmap/pectra/maxeb/#whats-a-withdrawal-credential); and whether the validator has been slashed. Each validator is controlled by its public-private validator signing key pair and withdrawal credentials, and performs duties for the network — attesting to and proposing new blocks, participating in sync committees, and reporting network violations of the slashing rules|_Standalone_:<br>A validator run by a solo NO on a standalone node and controlled by an unsplit validator key<br><br>_Distributed_:<br>\* A validator run by a solo NO on a multi-node setup and controlled by an unsplit validator key, and/or<br>\* A validator run by a solo NO with distributed remote key management and controlled by the threshold aggregate of the validator key shares generated at the NO's discretion, or<br>\* A DV run by a cluster utilizing DVT and controlled by the threshold aggregate of the validator key shares generated during a distributed key generation (DKG) ceremony

## C – Validator Exit Infrastructure

### C.1 – Validator Exit Order
The validator exit order in Lido follows a deterministic sequence, which allows independent and trustless computation of exits if needed. The order is programmatic and, in tandem with the [allocation algorithm](https://docs.lido.fi/contracts/staking-router#allocation-algorithm) that distributes inflows, works to organically balance the protocol stake among NOs within and across each SM. By default, keys are requested to exit from the NOs that run the highest number and longest-standing active validators with the lowest indices. Other factors that can play a conditional role are the `targetLimit` (a per-NO per-SM property, see section [C.1.1](#c11--target-limit)) and the `stakeShareLimit` (an SM-level property, see section [C.1.2](#c12--stake-share-limit)).

Following the implementation of TWs into the protocol (see section [C.2](#c2--triggerable-withdrawals-framework)), the priority of validator exits is determined by the following [sorting predicate](https://github.com/lidofinance/lido-oracle/blob/develop/src/services/exit_order_iterator.py#L32) in descending order.

Sorting|Staking Module|Node Operator|Validator
:---:|---|---|---
:arrow_down:||Highest number of targeted validators to boosted exit|
:arrow_down:||Highest number of targeted validators to smooth exit|
:arrow_down:|Highest deviation from the exit share limit (i.e., highest positive difference between `currentShare` and `priorityExitShareThreshold`)||
:arrow_down:||Highest number of active validators|
:arrow_down:|||Lowest index

_NOTE: With the [adoption of consolidations in Lido](https://blog.lido.fi/lidos-roadmap-to-pectra-delivering-validator-consolidations-in-the-protocol), criteria that were previously normalized to the number of active validators — each representing Ethereum's minimal deposit size of 32 ETH — will instead be mapped to individual key balances._

#### C.1.1 – Target Limit
The `targetLimit` and `targetLimitMode` parameters apply to NOs on a per-SM basis — i.e., the `targetLimit` is not shared by the same entity across different SMs. The `targetLimitMode` determines the approach through which exits are requested, and — if a `targetLimitMode` is enabled — the `targetLimit` defines the target number of active validators for an NO in a given SM.

Value|`targetLimitMode`|Deposits|Exits
:---:|---|---|---
`0`|Disabled|The NO can receive stake without limitation up to the capacity of its depositable [`vettedSigningKeysCount`](https://docs.lido.fi/contracts/node-operators-registry#setnodeoperatorstakinglimit)|The NO's validators have default exit priority
`1`|Smooth exit|The NO has a limit to its number of active validators. As long as the NO's number of active validators does not exceed the specified `targetLimit`, the NO may receive stake from the allocation algorithm. However, once the value is reached, the distribution of further stake is paused|If the NO's active keys exceed the `targetLimit`, its validators are prioritized for exit when there is demand in the WQ
`2`|Boosted exit|The NO has a limit to its number of active validators. As long as the NO's number of active validators does not exceed the specified `targetLimit`, the NO may receive stake from the allocation algorithm. However, once the value is reached, the distribution of further stake is paused|If the NO's active keys exceed the `targetLimit`, exits will be requested regardless of demand in the WQ

An NO's `targetLimitMode` and `targetLimit` for a given SM are set by the Lido community via [Easy Track motion](https://easytrack.lido.fi/) or [on-chain vote](https://vote.lido.fi/).

#### C.1.2 – Stake Share Limit
Each SM has a `stakeShareLimit`, `priorityExitShareThreshold` and `currentShare` parameters, where `stakeShareLimit` <= `priorityExitShareThreshold`:
* `stakeShareLimit` is the maximum percentage of total ETH in Lido that can be distributed to an SM during stake allocation
* `priorityExitShareThreshold` is the stake share threshold beyond which validator exits from an SM are prioritized
* `currentShare` is defined by the percentage of ETH Staked in the SM with respect to the total ETH staked in Lido

Since the total stake in the protocol is dynamic as a result of inflows and outflows, it is possible that SMs temporarily end up with a `currentShare` higher than their `stakeShareLimit`. The community members monitoring Lido (see section [C.2.1](#c21--validator-exit-reporting)) take this into account when determining the next validators to be exited, and if over-allocated, report that an SM must be prioritized for exits (multiple SMs can be prioritized in this manner simultaneously). Thus, each SM is in one of three states at any given time.

When:
* `currentShare` < `stakeShareLimit`, an SM has default allocation priority, and validators in the SM do not have priority for exit
* `stakeShareLimit` <= `currentShare` <= `priorityExitShareThreshold`, the SR does not allocate further stake to an SM, but validators in the SM do not have priority for exit
* `priorityExitShareThreshold` < `currentShare`, the SR does not allocate further stake to an SM, and validators in the SM have priority for exit

### C.2 – Triggerable Withdrawals Framework
Lido continues to rely on validator exit reports submitted by oracles and on Node Operators to process them through voluntary exits, with the Triggerable Withdrawals framework introducing an additional fallback mechanism if those steps are not carried out. The [Triggerable Withdrawals framework](https://snapshot.box/#/s:lido-snapshot.eth/proposal/0x7d7f0e1a6d181310f8752af37e20515a9be258f30b211872f9acca99bc478851) adds to the protocol the option of permissionless, secure, and verifiable exits that can be initiated from Ethereum's execution layer (EL) (see [Appendix A.3](#a3--triggerable-withdrawal)). This improves the fault tolerance, reduces trust assumptions and strengthens the foundation for permissionless staking. An overview of the key features and components is provided below — the complete specification can be found in [LIP-30](https://github.com/lidofinance/lido-improvement-proposals/blob/develop/LIPS/lip-30.md).

#### C.2.1 – Validator Exit Reporting
As part of the framework, the [Validators Exit Bus Oracle (VEBO)](https://docs.lido.fi/contracts/validators-exit-bus-oracle) — the interface for oracle-driven validator exit reporting — has been incorporated in the [Validators Exit Bus (VEB)](https://docs.lido.fi/guides/oracle-spec/validator-exit-bus), which in turn has taken over the role from the VEBO as the main coordinator for such requests in the protocol. Following this change, exit requests can be submitted by a broader set of actors and via more mechanisms. In addition to the established path — where oracles, via [HashConsensus](https://docs.lido.fi/contracts/hash-consensus), reconcile the on-chain states of Lido and the validators run using the protocol, along with the list of keys requested to exit derived from the Validator Exit Order (see section [C.1](#c1--validator-exit-order)) — exits, can also be requested ad-hoc via Easy Track motions that interface with the VEB and are callable by trusted entities like Curated NOs or the [Simple DVT Module Committee (SDVTMC)](https://research.lido.fi/t/simple-dvt-module-committee-multisig/6520). Whether received directly or through the VEBO, the hashes of all reports are stored in the VEB smart contract, which subsequently waits for the underlying data to be revealed. Not only the [Lido Oracle Committee](https://docs.lido.fi/guides/oracle-operator-manual#committee-membership), but anyone is allowed to submit report data matching archived hashes. When data is received and unpacked, the VEB records a timestamp linked to the associated hash, enabling anyone to prove that a validator has been requested to exit and when the corresponding event has been emitted via a VEB report.

#### C.2.2 – Triggerable Withdrawal Request Submission
Under the framework, a permissionless ejection can theoretically be initiated immediately once a key has been requested to exit. This ensures that Lido can adequately respond to exceptional situations such as those listed in the Purpose (see section [A](#a--purpose)). Moreover, several [security considerations](https://github.com/lidofinance/lido-improvement-proposals/blob/develop/LIPS/lip-30.md#6-security-considerations) were taken into account during the development of the framework to establish [limits](https://github.com/lidofinance/lido-improvement-proposals/blob/develop/LIPS/lip-30.md#appendix-a--limit-implementation) and [parameters](https://github.com/lidofinance/lido-improvement-proposals/blob/develop/LIPS/lip-30.md#7-proposed-params) that harden the protocol against various attack vectors, while keeping TWs economically sensible and suitable for a wide range of use cases. In practice, however, the [demand-adjusted request fee](https://eips.ethereum.org/EIPS/eip-7002#rate-limiting-using-a-fee) renders TWs prohibitively expensive compared to voluntary exits (see [Appendix A.1](#a1--voluntary-exit)), making them a fallback mechanism rather than a practical alternative for regular validator exit processing.

A Triggerable Withdrawal Request (TWR) can be initiated in two ways
* permissionlessly by anyone specifying an array of keys that has been requested to exit from any combination of Lido SMs or staking avenues and providing proof of the corresponding VEB report(s), or
* by an NO that elects or has been requested to exit its validators run for the CSM (or any future permissionless SM).

Depending on the case, the TWR must be created through either the VEB, or ejector specific to the permissionless SM or staking avenue (at the time this version of the SNOP was enacted, only the CSM Ejector existed).

The VEB checks that
* the exit data hashes provided match some of those stored in the contract,
* the underlying data has been revealed, and
* the corresponding exit events have been emitted via VEB report(s),

while the ejector verifies that
* the requested keys belong to the permissionless SM or staking avenue,
* the validators were deposited via the [Deposit Security Module (DSM)](https://docs.lido.fi/contracts/deposit-security-module),
* there are no duplicates in the request to avoid drying out limits, and
* there is additional logic implemented to confirm that the one calling the TWR is authorized (i.e., controls the keys or is creating a TWR for validators that are permitted, respectively requested to exit).

The subsequent flow is the same for both cases. Once validated, the request is forwarded to the Triggerable Withdrawal Gateway (TWG) — the smart contract responsible for handling triggerable exit requests in Lido —, which then:
* enforces frame-level rate limits,
* forwards the request to the [Withdrawal Vault](https://docs.lido.fi/contracts/withdrawal-vault), which interacts with the EIP-7002 precompile to trigger the actual exits,
* notifies the associated SM or staking avenue of the exits through the SR, and
* refunds the requester any potential surplus fee provided.

#### C.2.3 – Late Validator Exit Reporting
A late validator exit refers to a key that has been requested to exit, for which the responsible NO has failed to initiate the exit processing on Ethereum's consensus layer (CL) within the predefined time frame (see section [D.3](#d3--validator-exit-request-processing)). To enable automated enforcement of consequences for NOs non-conformant with the rules and expectations of the SM and this SNOP, late validators must be reportable on-chain. The Triggerable Withdrawals framework ensures this by allowing anyone to permissionlessly submit to the [Validator Exit Delay Verifier](https://github.com/lidofinance/lido-improvement-proposals/blob/develop/LIPS/lip-30.md#44-validator-exit-delay-verifier) a CL proof of a validator's continued active state, along with reference to the VEB report that requested the key to exit. The smart contract then confirms with the VEB that the exit has been requested and calculates the difference between the timestamps of the validator state proof and the exit request emission. In rare cases, e.g., when a key is requested to exit shortly after activation — even before exit initiation is technically possible — the difference is instead calculated between the timestamp of the validator state proof and the moment the key first becomes eligible to exit. The resulting time delta is passed through the SR to the SM, which then decides what further action is warranted based on its internal logic.

#### C.2.4 – Validator Exit Automation
The Triggerable Withdrawals framework defines all exit enforcement actions, including the submission of lateness proofs and the initiation of validator exits, as permissionless. Any Ethereum participant with sufficient funds for transaction fees can execute these actions, and no privileged role is required. The protocol therefore does not depend on designated operators to ensure validator exits are carried out.

To reduce reliance on manual monitoring and intervention, two open-source off-chain bots are available. The [Validator Late Prover Bot](https://github.com/lidofinance/lido-improvement-proposals/blob/develop/LIPS/lip-30.md#55-validator-late-prover-bot) monitors validators that have been requested to exit but remain active beyond the defined processing window (see section [D.3](#d3--validator-exit-request-processing)), and submits the corresponding proofs. The  [Trigger Exits Bot](https://github.com/lidofinance/lido-improvement-proposals/blob/develop/LIPS/lip-30.md#54-trigger-exits-bot) initiates the exits of validators that have been verified as late. Both bots can be operated by any party, and their function is to ensure that permissionless enforcement is executed in a timely and continuous manner.

## D – Node Operator Responsibilities
Lido on-chain signalling mechanisms notify NOs when to process validator exits. If exits for keys under their operation have been requested via VEB report(s), NOs have the responsibility to fully, correctly, and timely exit the validators in accordance with the rules and expectations of Lido and this SNOP. To determine whether NOs have appropriately performed the required actions, the following section outlines some exceptions and the expected timelines for regular exit processing.

The actual mechanisms for validator exits are at the discretion of the NOs, with community-built, open-source tooling available (see [Appendix B](#appendix-b--tooling)) to aid in identifying and processing signalled requests in addition to any internal tooling, APIs, or manual processes the NOs may use.

### D.1 – Out of Order Exits
Out of order exits refer to exits initiated by CM or SDVTM NOs for validators that have not been requested to exit via VEB report(s). In such cases, the NO concerned must notify the community that an out of order exit has been processed, specify the number of validators affected, their indices, and the reason for the exit. CM NOs should communicate this via the [Node Operator category of the Lido Research Forum](https://research.lido.fi/c/node-operators/12) and SDVTM NOs to the SDVTMC. As the CSM is permissionless, and CSM validators are [bonded](https://operatorportal.lido.fi/modules/community-staking-module#block-e4a6daadca12480d955524247f03f380), this expectation does not extend to them, and CSM NOs are free to exit out of order at their discretion.

### D.2 – Operational Continuity
If at any point in time a CM or SDVTM NO is unable to continue operations, they must notify the community. This notice must take place  before any further action is taken (14 days in advance for SDVT and at least 60 days in the case of the CM), outlining the circumstances and signalling their intent to exit all validators run using the protocol. CM NOs should communicate this via the Node Operator category of the Lido Research Forum and SDVTM NOs to the SDVTMC.<br>If the Lido community does not instruct the NO concerned otherwise — e.g., via an unobjected Easy Track motion or approved Snapshot vote in some instances — the NO must trigger the exits within the prescribed time frame.

An exception to this is an inter-operator cluster (see section [B](#b--scope)) that has not yet utilized the validator key reshare ceremony, which in Lido can be permitted with community approval — e.g., to replace a participant unable to continue without having to exit the cluster's active DVs and thus maintain operational efficiency. In order to avoid risks associated with validator key security, reshare ceremonies are limited to one per cluster.

### D.3 – Validator Exit Request Processing
The VEB regularly and on an ad-hoc basis settles on-chain reports that indicate which validators should be exited by which NO, and constitute the exit request signal. Scheduled reports are published at intervals referred to as frames. Each frame spans a duration defined by a specific number of CL epochs (~6.4 min each) with the exact length determined by the instance of the HashConsensus smart contract that the VEB is called by. NOs are responsible for promptly setting up infrastructure to capture and process signalled exit requests pertaining to validators they run using Lido.

A validator exit request is considered
* `signalled` once it is retrievable from a VEB report,
* `processed` once either a voluntary exit message (VEM) for the key has been broadcast to the CL (see [Appendix A.1](#a1--voluntary-exit)), or a TWR to the EL (see [Appendix A.3](#a3--triggerable-withdrawal)), and the corresponding transaction has been included in a block proposed to the Beacon Chain (BC), and
* `fulfilled` once the validator has been fully exited and withdrawn.

Although the process can largely be automated, differences in infrastructure, working hours, and mechanism timings were considered to outline validator exit processing time frames that NOs must adhere to. If a NO has processed a `signalled` request immediately upon its availability via a VEB report, the minimum time for the request to move from `signalled` to `processed` status is typically between a few minutes to one hour.

To avoid lateness, different SMs and NOs types are granted a specific amount of time (time frames) for the request processing:

Module|Node Operator|Time to process
:---:|---|---
Curated|Curated NO|96h
Simple DVT|Simple DVT NO|96h
CSM|[Identified Community Stakers (ICS)](https://blog.lido.fi/unlock-exclusive-benefits-as-an-identified-community-staker/)|120h
CSM|Other, excluding ICS|96h



## E – Consequences of Non-conformance
In cases where an NO may not meet the expectations and rules outlined by this SNOP, the following actions may be taken. The specific actions and consequences may differ depending on the SM or staking avenue. For example, bonded SMs, like the CSM, allow additional options for automated conformance mechanisms compared to bondless SMs such as the CM and SDVTM.

The table below outlines the potential consequences for non-conformance with the Node Operator Responsibilities (see section [D](#d--node-operator-responsibilities)) regarding validator exits in Lido, along with the SM to which each action applies.

Action|CM|SDVTM|CSM
---|:---:|:---:|:---:
If (1) an out of order exit (see section [D.1](#d1--out-of-order-exits)) is initiated without communication, (2) an exit — in the event of operational discontinuation (see section [D.2](#d2--operational-continuity)) — is initiated without prior notice or outside of the prescribed time frame, or (3) a validator exit request is processed late (see section [D.3](#d3--validator-exit-request-processing)), any Lido DAO contributor or community member may reach out to the NO and request a clarification. Depending on the severity of the case, community members may consider to raise a formal issue with the NO on the Lido Research Forum, initiate ejection of the key(s) (see section [C.2](#c2--triggerable-withdrawals-framework)), request compensation for any potentially incurred costs to Lido, propose to set a limit to the NO's depositable `vettedSigningKeysCount` and priority to exit for the remaining active validators (see section [C.1.1](#c11--target-limit)), or other actions, including the offboarding of the NO|:heavy_check_mark:|:heavy_check_mark:|:x:
If a validator exit request is processed late (see section [D.3](#d3--validator-exit-request-processing)), the corresponding key(s) will be automatically ejected (see section [C.2.4](#c24--validator-exit-automation)). The demand-adjusted TW request fee  (capped at 0.1 ETH per validator by the [framework, see section [C.2.2](#c22--triggerable-withdrawal-request-submission), will be paid from the affected NO's bond. Additionally, a exit delay penaltylate exit disincentive will be burned for the benefit of all stETH holders — 0.05 ETH per key for ICS and 0.1 ETH per key for permissionless CSM NOs. If any of the NO's validators are unable to provide a full bond — i.e., become unbonded — as a result of the enforcement, those validators are requested to exit the protocol.|:x:|:x:|:heavy_check_mark:

# Appendix

## Appendix A – Ethereum Validator Exit Mechanisms
There are two ways a validator can exit from Ethereum's CL. The first is a voluntary exit — an NO may choose for its validator to stop performing duties for the network by submitting a VEM to the BC (see [Appendix A.1](#a1--voluntary-exit)). The second mechanism is forced ejection from the protocol (see [Appendix A.2](#a2--forced-exit)) — this could be triggered by slashing or an insufficient effective balance. 

Additionally, the [Pectra hardfork](https://eips.ethereum.org/EIPS/eip-7600) introduced a third validator exit option — EL withdrawal credentials initiable TWs (see [Appendix A.3](#a3--triggerable-withdrawal)).

### A.1 – Voluntary Exit
An NO can initiate a voluntary exit at any time, as long as the associated validator has been active for at least 256 epochs (~27 h) and not been slashed. The NO does so by signing a VEM using the private validator key and broadcasting the message to the network to have it processed, either directly through a local CL client, or indirectly — e.g., via an external CL node or API. Once the validator reaches its exit epoch and the NO confirms it has not been selected for [delayed sync committee participation](https://hackmd.io/1wM8vqeNTjqt4pC3XoCUKQ), the validator ceases to perform duties and receive rewards and penalties — i.e, enters the withdrawable state. In this state, the validator waits until its index is parsed by the withdrawals sweep operation and for its balance to be withdrawn to the associated withdrawal address.

_NOTE: Validators with [BLS 0x00 credentials](https://notes.ethereum.org/@launchpad/withdrawals-faq#Q-What-are-0x00-and-0x01-withdrawal-credentials-prefixes) have to rotate to 0x01 for the withdrawal to be processed; otherwise, they are skipped._

### A.2 – Forced Exit
Forced exits are not relevant for the purpose of this SNOP, but can occur either as a result of a validator being slashed or its effective balance leaking below Ethereum's current [ejection balance](https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/beacon-chain.md#validator-cycle) threshold of 16 ETH.

### A.3 – Triggerable Withdrawal
[EIP-7002](https://eips.ethereum.org/EIPS/eip-7002) introduced a new functionality to Ethereum that enables EL withdrawal credentials to signal and initiate partial or full withdrawals of associated validators on the CL. This improves assurances around the timely and orderly exit of validators for staking solutions, as well as potential countermeasures against malicious or underperforming NOs. The downside, however, is that TWs incur on-chain gas for transactions that do not natively occur on the CL, as well as variable, demand-adjusted request fees (see section [C.2.2](#c22--triggerable-withdrawal-request-submission)). Moreover, the combined costs are partially non-refundable in the case of accidental underspending. This makes TWs less practical than CL-initiated variants for regular exit use cases, nor are they a cure-all, since validators can still misbehave while in the exit queue. Nevertheless, for Lido, the introduction of TWs marks a key functional advancement in the offering of increasingly trustless, permissionless staking.

## Appendix B – Tooling
A range of community-built, open-source tooling is available for Lido and Ethereum staking in general. These tools can assist NOs in monitoring and semi- or fully-automated processing of validator exits. NOs may choose to adopt third-party implementations, develop custom solutions, or any combination thereof — e.g., by connecting the [Validator Ejector](https://docs.lido.fi/guides/validator-ejector-guide) to a custom API or generating requests on a just-in-time basis.

Detailed descriptions and the source codes of tools can be found on the [Node Operator Portal](https://operatorportal.lido.fi/existing-operator-portal/ethereum-onboarding/no-resources-tooling).
