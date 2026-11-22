# KitaPay: a 20,000 TPS Digital Wallet on Azure

**Azure Architecture Series · Week 01**

A reference architecture for a fictional regional e-wallet and payments platform (18 million users, Malaysia and Singapore). The design target is simple to state and hard to meet: **money must never be wrong, and must not be down on sale night.**

![Architecture diagram](03-diagram/KitaPay-Azure-Architecture-Diagram.png)

## What's in this repository

| Folder | Contents |
|---|---|
| [`01-report/`](01-report) | 20-page PDF: scenario, NFRs, capacity math, component walkthrough, payment lifecycle, data architecture, 7 ADRs, resilience and DR runbook, security, cost and SLOs, senior↔junior design review, 7-day study guide |
| [`02-linkedin/`](02-linkedin) | Post thumbnail (1200×627 and 2× versions) and ready-to-paste post copy |
| [`03-diagram/`](03-diagram) | The architecture diagram as SVG (editable), PNG, and A3 PDF |

## Design at a glance

| Decision | Choice | Main trade-off |
|---|---|---|
| Topology | Two **cells** (Malaysia West, Southeast Asia); each user has one home cell | Cross-cell payments become sagas; standby cell sized for 100% load |
| Ledger | **Sharded Azure SQL Business Critical** (16 shards per cell), ledger tables | Per-shard limits force sharding discipline; BC tier costs more |
| Cross-shard flows | **Orchestrated saga** + transactional **outbox** | "Pending" is a real state; orchestrator is critical-path software |
| Messaging | **Service Bus** (commands) + **Event Hubs** (events) | Two systems to operate |
| Compute | **AKS** across 3 AZs, Istio, KEDA, Workload Identity | Real platform-engineering cost |
| Fraud | **Azure ML** endpoint in the payment path, 60 ms budget, tiered fail policy | Extra hop; fail-open accepts bounded loss |
| DR | Cross-border async replication, **RPO ≤ 5 s / RTO ≤ 15 min**, quarterly game days | Malaysia West has no paired region, so PII residency needs legal sign-off |

Targets: 99.99% payment-auth availability · p99 ≤ 800 ms · 20,000 TPS design peak · RPO 0 in-region.

## Honest notes

- **KitaPay is fictional.** Numbers (users, TPS, costs) are planning assumptions with shown derivations, not measurements. The first step in real life would be load-testing them.
- **Icons are simplified original glyphs**, not Microsoft's official icons. For a production document, use the official [Azure Architecture Icons](https://learn.microsoft.com/azure/architecture/icons/) and follow Microsoft's terms.
- **Region and feature availability changes.** Region facts (Malaysia West: 3 availability zones, no paired region; Southeast Asia 3 listed as coming soon) and the Azure Cache for Redis → Azure Managed Redis retirement dates were checked against Microsoft Learn on 21 Nov 2026. Per-service zone-redundancy in Malaysia West is **not** verified and is listed as an open action.
- **Open design decision:** Front Door Premium → API Management over Private Link is documented for v1 tiers only when APIM is *not* VNet-injected, so this design uses a private regional entry point in front of internal-mode APIM. Validate in a proof-of-concept.
- **Identity caveat:** Entra External ID supports passkeys, but with limits when checked (not via native authentication APIs; registration needs an email/password account plus recent MFA). The design uses MFA as the baseline and treats passkeys as an enhancement to verify. Front Door routes by hostname, not JWT claims; APIM validates the `home_cell` claim.
- This design supports the kind of evidence regulators ask for; it does not by itself make an organisation compliant with any specific rulebook.

## Series

A new enterprise architecture every week, alternating Azure and AWS.

| Week | Cloud | Scenario |
|---|---|---|
| 01 | Azure | Regional digital wallet at 20,000 TPS |
| 02 | AWS | *coming next* |

*Author: add your name, LinkedIn and contact links here.*
