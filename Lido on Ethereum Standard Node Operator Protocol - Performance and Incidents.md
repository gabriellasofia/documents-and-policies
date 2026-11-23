# Lido on Ethereum Standard Node Operator Protocol – Performance and Incidents

```markdown!
STATUS: V1
```

## A – Purpose
This Standard Node Operator Protocol (SNOP) outlines
* the performance expectations for validators run using the Lido on Ethereum protocol ("Lido"), and how performance is measured,
* the rules around the usage of this SNOP,
* the considerations that guide Lido's expectations around validator performance and incident handling,
* the responsibilities Node Operators (NOs) using Lido have in maintaining performance, keeping their software up to date, and detecting, communicating and reporting incidents, as well as
* the actions NOs can expect in case of non-conformance with this SNOP.


## B – Scope
This SNOP applies to the Lido protocol, the NOs who use any of the [Staking Router (SR)'s](https://docs.lido.fi/contracts/staking-router) existing SMs — [Curated Module (CM)](https://operatorportal.lido.fi/modules/curated-module), [Curated Module v2 (CMv2)](https://research.lido.fi/t/future-of-the-curated-module-cmv2-landscape/10929), [Simple Distributed Validator Technology Module (SDVTM)](https://operatorportal.lido.fi/modules/simple-dvt-module), and [Community Staking Module (CSM)](https://operatorportal.lido.fi/modules/community-staking-module) — any future SM, and the validators the NOs run as part of the protocol.

The following operational flows and expectations apply to all above-mentioned sources, except where alternative processes or expectations are defined due to technical or operational differences.

_NOTE: Terminology may vary in meaning depending on context. The below table aims to clarify the usage of key terms for the purpose of this SNOP and within the context of Lido._

Term|Definition|Context
:---:|---|---
Node Operator|An entity, individual, or cluster thereof that operates nodes using any of the SR's SMs or staking avenues to run Ethereum validators|_Solo_:<br>A single entity or individual that operates validators<br><br>_Cluster_:<br>\* An intra-operator cluster by a single entity or individual utilizing [distributed validator technology (DVT)](https://ethereum.org/en/staking/dvt/#distributed-validator-technology) that operates distributed validators (DVs), or<br>\* An inter-operator cluster by multiple entities and/or individuals utilizing DVT that operates DVs
Validator|A validator — also referred to as a "key" in this SNOP — is the technical component of participating in Ethereum's consensus. If active, it represents an [effective balance](https://eth2book.info/capella/part2/incentives/balances/) ranging from 16 up to 2048 ETH (in 1 ETH increments), according to their corresponding [0x01 or 0x02 withdrawal credentials](https://ethereum.org/en/roadmap/pectra/maxeb/#whats-a-withdrawal-credential); and whether the validator has been slashed. Each validator is controlled by its public-private validator signing key pair and withdrawal credentials, and performs duties for the network — attesting to and proposing new blocks, participating in sync committees, and reporting network violations of the slashing rules|_Standalone_:<br>A validator run by a solo NO on a standalone node and controlled by an unsplit validator key<br><br>_Distributed_:<br>\* A validator run by a solo NO on a multi-node setup and controlled by an unsplit validator key, and/or<br>\* A validator run by a solo NO with distributed remote key management and controlled by the threshold aggregate of the validator key shares generated at the NO's discretion, or<br>\* A DV run by a cluster utilizing DVT and controlled by the threshold aggregate of the validator key shares generated during a distributed key generation (DKG) ceremony
Sub-operator|A subsection of infrastructure that forms a distinct operational unit within a Node Operator and is responsible for operating a subset of that Node Operator’s validators under a defined stake weight.|Applies in CMv2| Performance | Share of a validator's assigned duties (attestations, block proposals, sync committee participation) performed correctly and on time over a frame, as measured by [metric / oracle] | [Per SM]
Performance threshold | Minimum performance a validator must reach in a frame: [value] | [Per SM or bond curve]
Frame | Measurement period for performance and strikes: [length] | [Per SM]
Strike | Record assigned to a validator for each frame its performance falls below the threshold; expires after [lifetime] | CM (CMv2 Phase 2), CSM
Downtime | Period in which a validator does not perform its duties; planned (announced maintenance) or unplanned | All SMs
Missed proposal | A slot in which the validator assigned to propose does not produce a canonical block | All SMs 
Pending exit | An exit request not yet fulfilled; late once past the window set in the Validator Exits SNOP | All SMs
Incident | An event that degrades, or risks degrading, validator performance, security or staker funds; classified per Appendix B | All SMs
Client release | A new version of EL, CL or VC software, or of a sidecar (DVT clients, APM sidecars); mandatory when required for a network upgrade | All SMs
Security release | A release fixing a vulnerability, often under coordinated or embargoed disclosure, that requires expedited application | All SMs

## C – Guiding Considerations

The considerations below follow from the goals of the Lido DAO, as set out in the Lido DAO Vibe and GOOSE, and from the goals of the Curated Module v2 (CMv2). They shape the NO responsibilities in section D. 

### C.1 – Economic Security

Lido Core provides the safety, performance and decentralization guarantees that stETH holders, and the stVaults built on top of it, rely on. Staker rewards come directly from the duties NOs' validators perform, so performance is the main lever NOs have on what stakers earn and on Lido's competitiveness. Downtime costs more than the rewards it forgoes: missed attestations are penalized, and if the chain stops finalizing, offline validators also face an inactivity penalty that grows the longer the outage lasts.

Slashing is the largest single risk to staker funds. On top of the initial penalty, a correlation penalty scales with how much stake is slashed around the same time, so one compromised signing setup across many validators costs far more than isolated mistakes. Consolidated 0x02 validators of up to 2048 ETH put more stake behind each key, raising the cost of every failure. This is why the security requirements in secion D aim to remove any path from a compromised host to a slashable signature.

### C.2 – Decentralization & Resilience

Diversity limits correlated failures, which hurt stakers and the network most. A bug in a client run by more than a third of stake can stop finality; in a client run by more than two thirds, it could finalize an invalid chain, leaving validators that attested to it unable to leave without risking slashing. Given Lido's share of staked ETH, NOs' client choices add up to a meaningful part of network-level client diversity.

The same reasoning applies to infrastructure: concentration in one cloud provider, region or remote-signing setup turns a single outage into a set-wide event. Upgrades should be timely, so the set is ready well before a network upgrade, but also staggered and tested on testnet first, so a faulty release doesn't reach every validator at once. Within each NO, redundancy and safe failover (D.6) keep a local failure from becoming downtime, without trading it for double-signing risk.

### C.3 – Measurable & Proportionate Accountability

Accountability for Curated NOs has rested mainly on reputation, and that has worked well. CMv2 complements it with economic alignment: bonds, strikes and penalties tie sustained underperformance and serious incidents to the NO's own capital, rather than leaving the cost with stakers alone. This SNOP turns expectations into objective, measurable standards so NOs know in advance what is expected and how conformance is assessed.

The goal is for enforcement to remain fair and transparent. Automated consequences follow on-chain rules; other penalties are reported by the relevant module committee and remain subject to DAO oversight, e.g. through Easy Track challenge periods.
Expectations are proportionate. They scale with the stake an NO runs and the module it operates in, and can be tailored by operator type: Decentralization Operators, for example, may be granted greater performance tolerance for their setups. Issues caused by an NO are distinguished from network-wide events outside its control. Timely incident communication lets Lido assess impact across the set.

## D – Node Operator Responsibilities

Based on the Guiding Considerations (section C), the following responsibilities arise for NOs using Lido with respect to the performance of their validators and the handling of incidents. This section also outlines how stakers and the community can assess conformance, e.g. via [monitoring dashboards, superApp?].

Lido monitors conformance continuously, and NOs can expect:
* Monitoring: Lido tracks performance, missed proposals, pending exits and client versions using [sources, dashboards?], which stakers and the community can also use to assess conformance.
* Automated alerts (AutoNOM): [NO-facing alerts, delivery channel and expected NO action?].
* Incident process: In the CM, NOM notices and investigates any non-compliance regarding this policy, while the CMC decides on penalties and strikes.
* Public reporting: incidents are raised on the Lido Research Forum.


### D.1 – Validator Performance

#### D.1.1 – Duty Performance

- Keep every validator performing its attestation, proposal and sync committee duties at or above the benchmark for its module; methodology in Appendix A.
- For CM: an incident is opened when performance (RAVER) stays below the network average minus 3% for 7 days. The tolerance may differ for Decentralization Operators.

#### D.1.2 – Downtime

- For CM: an incident is opened when more than 2048 ETH of stake is offline for more than 30 minutes, or for another material downtime pattern. Notify Lido under D.5 without waiting to be contacted.
- Planned maintenance: announce via Telegram chat at least [an hour] in advance, stagger it so validators don't go offline at once, and avoid network upgrade windows.
- Where staying offline is the safer option (e.g. to avoid slashing risk during a key migration), agree it with NOM first. Conservative action taken in coordination with NOM is assessed in light of the risk it avoided.

#### D.1.3 – Missed Proposals

- Missed proposals count toward performance (D.1.1); [X] misses within [period] are handled as an incident under D.5.
- Missed or misdirected rewards from fee recipient, relay or builder configuration are covered by the Block Proposals SNOP and, in the CMv2, by the Penalty Framework.

#### D.1.4 – Strikes

The CMC can issue strikes to a sub-Node Operator for misbehavior, poor performance or other relevant issues. Strikes reduce the NO's stake allocation (ValMart) weight, and their effect is cumulative.

- Each strike states its name, category, description and lifetime.
- Once a strike's lifetime ends, the NO can delete it, removing its effect; reminders come through the UI or Sentinel.
- The CMC can cancel any strike at any time.


### D.2 – Pending Exits

- Process protocol-requested exits within the Validator Exit Processing Window set in the Validator Exits SNOP: 4 days for all CM operator types.
- CM: an exit not processed in time can be forced via EIP-7002 triggerable withdrawals; the NO is charged the exit delay fee for its operator type and reimburses the triggerable withdrawal fee.
- CM: exits outside a protocol request are discouraged. Announce any in advance to the CMC and in the Node Operators category of the Lido Research Forum; out-of-order exits may be penalized.
- If an exit will miss the window, handle it as an incident under D.5: notify and give an estimate for completion.

### D.3 – Client & Security Releases

#### D.3.1 – Client Releases

- Node Operators should run supported versions of EL, CL and VC software and sidecars (MEV-Boost, APM sidecars, DVT clients); apply non-mandatory releases within [X].
- Network upgrades: upgrade testnet nodes ahead of the testnet fork, and mainnet nodes no later than [the day] before the fork epoch; confirm readiness for all upgrades.
- Stagger upgrades across infrastructure rather than upgrading every node at once.

#### D.3.2 – Security Releases

- NO must acknowledge Lido's security announcement within [X].
- Apply the release within [X hours] of availability, or the window stated in the announcement, and confirm completion.
- Any information embargoes must be respected and no public disclosure before the coordinated release should be made.

### D.4 – Security Requirements

Under the updated Security Requirements [forum post?], Curated Module NOs must sign validator duties from inside an attested Trusted Execution Environment (TEE), with keys protected by a Hardware Security Module (HSM) or a cloud Key Management Service (KMS) with Secure Key Release; with threshold signing strongly encouraged. 

#### D.4.1 – Signing & Key Protection

- Validator duties are signed only inside the TEE.
- Keys, or key shares, are protected by an HSM or a cloud KMS with SKR. 
- Keys must be generated through an isolated process or an approved distributed generation scheme, with all keys, shares, and recovery material protected throughout.
- Operators remain responsible for access controls and every potential point of entry to key material.

#### D.4.2 – Compliance Paths

- Accepted solutions to comply with the security requirements include: Lido tee-signer, Dirk on TEE, or a NOs own solution, subject to the review of the Lido contributors and previous sucessful testing. [add others?]


#### D.4.4 – Operational Security Baseline

Baseline controls beyond signing, drawn from [ValOS](https://www.valos.global/valos-spec.html):

- Slashing protection data kept and never disabled or deleted; doppelgänger protection on every (re)start (ValOS 4.3.2, 4.9.5).
- Least-privilege access, strong authentication, network segmentation and logging of privileged access (ValOS 4.5, 5.6.2, 5.7.2).
- Releases verified before deployment and tested in a staging environment, against supply-chain malware (ValOS 4.8.3, 4.8.4).
- Security events monitored and analysed; suspected key compromise handled as an incident under D.5 (ValOS 4.9.3, 4.10.3).

### D.5 – Incident Management

#### D.5.1 – Classification

- Incidents are internally classified by severity per the Lido Incident Response Policy (Appendix B).
- The CMv2 Penalty Framework includes these incident types: validators offline (D.1.2), low performance (D.1.1), any slashing, out-of-order exits, EL rewards rerouting, and relay, builder or MEV policy violations. Other incidents that cause or risk losses can also be opened.

#### D.5.2 – Notification & Coordination

- Notify Lido via Telegram chat within [X] of identifying an incident of any severity, even if Lido's monitoring has not flagged it yet.
- An initial notice should include the following information: affected validators, impact, suspected cause, estimated time for resolution. [check?]
- Coordinate with NOM before actions that could affect validators or protocol performance, such as key migrations, failovers or exits.

#### D.5.3 – Investigation & Updates

- Update Lido until resolution, and on any material change.
- Requests might be redirected to the NO by NOM, including: timeline to resolution, affected validators, root cause analysis, infrastructure and configuration details, and remediation steps. Late or incomplete information counts against the NO when a penalty is assessed.

#### D.5.4 – Post-Incident Report & Compensation

- NOs are expected to produce a Post-Incident Report or Postmortem for any incidents under the policy, which includes causes and remediations applied.
- CM: where losses are attributable to the NO, it can compensate voluntarily once NOM, the CMC and the NO agree the amount, by sending it to the Lido Execution Layer Rewards Vault. Otherwise the CMC may report a General Delayed Penalty against the bond; the NO can contest this penalty in the relevant Lido Research Forum thread.

### D.6 – Business Continuity

NOs should keep documented, tested plans to keep validating, or to recover safely, after losing a provider, site, key person or critical dependency. These expectations draw on [ValOS](https://www.valos.global/valos-spec.html) sections 4.6 and 4.10. 

#### D.6.1 – Continuity Plan & Testing

- Maintain a plan covering loss of a cloud or data-centre provider, region or site; loss of signer, TEE or KMS access; key-person unavailability; and loss of a critical vendor, with recovery objectives for validator duties (ValOS 4.10.5).
- Review the plan at least annually and after major infrastructure changes. Test it at least annually, including a failover drill on testnet, and share a summary with Lido on request (ValOS 4.10.6).

#### D.6.2 – Resilience & Backups

- Avoid single points of failure: redundant beacon nodes, provider and geographic distribution, and a documented replacement path for each third-party dependency (cloud, KMS, remote signer, DVT) (ValOS 4.6.2, 5.1.2).
- Back up configuration and slashing protection data with deletion protection, and keep runbooks current so recovery doesn't depend on one person (ValOS 5.5.3, 5.5.4).

#### D.6.3 – Organizational Continuity

- Notify Lido in advance of material organizational changes: change of control, merger, insolvency risk, loss of key staff.
- If a NO would need to be fully wind down, it should notify the community via the Lido Research Forum, with exits and stake migration coordinated alogside the CMC and NOM, and following the Validator Exits SNOP and exit ordering rules. Under no circumstance should the NO initiate any critical steps by itself.

### D.7 – Communications & Announcements

- Monitor official channels: the Lido Research Forum (Node Operators category), Lido Validators Telegram chat, individual NO chats  or Discord, and the Lido Node Operator Portal.
- Acknowledge time-critical announcements (security releases, network upgrades).
- Keep technical and escalation contacts current, reachable 24/7 for any incident response.
- Raise questions or escalations with the CMC in its Research Forum section, where CMC actions on penalties and operator types are also published.

## E – Consequences of Non-Conformance

Consequences differ by SM. Under CMv2, a Node Operator is bonded and  automated mechanisms (strikes, bond penalties) aply, alongside the general outreach path.

The number of potential strikes issued by the CMC per incident (opening triggers in D.1.1, D.1.2 and D.5.1) is set:

| Incident type | Strikes per incident | Dependent on |
| --- | --- | --- |
| Validators offline | Up to 2 | Cause, preventability, coordination, responsiveness, affected validators and balance, missed rewards and penalties |
| Low performance | Up to 2 | Deviation from the network average, affected validators and balance, root cause, remediation, recurrence |
| Validator slashing | Up to 6 | Size and cause of the slashing |
| Out-of-order exit | Up to 5 | Cause, number of validators exited and their balances |
| EL rewards rerouting | Up to 2 | Whether rewards were refunded, amount rerouted, intent, recurrence |
| Relay, builder and MEV policy violations | Up to 2 | Cause, affected proposals, configuration, duration, missed or misdirected rewards |

The correspondent effect on ValMart weight is the following:

| Strikes held | 1 | 2 | 3 | 4 | 5 | 6 |
| --- | --- | --- | --- | --- | --- | --- |
| Weight reduction | None | 5% | 10% | 20% | 50% | 100% |

- Under catastrophic scenarios (e.g.; if the CMC cannot operate normally), the DAO can activate CSM-style per-validator strikes by setting a leeway that defines acceptable performance for CM operators.

Other aspects of non-conformance can be handled separately and outside the strike system:


| Non-conformance | Consequence |
| --- | --- |
| Exit pending past the window (D.2) | Per Validator Exits SNOP v4 |
| Mandatory or security release not applied in time (D.3) | [TBD] |
| Security requirements not met by the deadline (D.4) | Offboarding of the NO |
| Incident not notified, or report not submitted (D.5) | [TBD] |
| No continuity plan, or plan not tested (D.6) | [TBD] |
| Unresponsive to Lido outreach (D.7) | [TBD] |

## Appendix

### Appendix A – Performance Methodology

- [Metric definition, duty weights, threshold per SM, frame length, strike lifetime and strike threshold.]

### Appendix B – Incident Severity Matrix

- [Severity levels, criteria and response times from the Incident Response Policy.]

### Appendix C – Incident Process Flow

- [Diagram from the Incident Process Flow page.]

### Appendix D – Post-Incident Report Template

- Summary
- Timeline: detection, notification, mitigation, resolution
- Affected validators and impact: missed rewards, penalties
- Root cause
- Remediation
- Prevention measures
- Compensation, if applicable

### Appendix E – References

- [Validator Exits SNOP v4, Block Proposals SNOP v4, CMv2 and CSM documentation, Lido Node Operator Portal.]
