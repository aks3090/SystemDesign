# Amazon Leadership Principles ↔ DocuSign Projects Map (L6+ only)

**Start here.** For a 10-minute refresh, read [11-Quick-Recall-Sheet.md](./11-Quick-Recall-Sheet.md). The rehearsal script is [10-STAR-Answers-All-16-LPs.md](./10-STAR-Answers-All-16-LPs.md). Why each story is in or out is in [09-L6-Calibration-and-Hire-Develop.md](./09-L6-Calibration-and-Hire-Develop.md).

| # | Project | Deep dive |
|---|---|---|
| P1 | First-Party Access Management | [01](./01-First-Party-Access-Management.md) |
| P2 | AuthZ and Non-Human Identities | [02](./02-AuthZ-and-Non-Human-Identities.md) |
| P3 | Maestro and Workspaces Delegated Access | [03](./03-Maestro-and-Workspaces-Delegated-Access.md) |
| P4 | Unified eSign Sharing Migration to AuthZ | [04](./04-Unified-eSign-Sharing-Migration-to-AuthZ.md) |
| P5 | Runtime Configuration and Secrets Modernization | [05](./05-Runtime-Configuration-and-Secrets-Modernization.md) |
| P6 | Global Directory and Multi-Region Identity Caching | [06](./06-Global-Directory-and-Multi-Region-Identity-Caching.md) |
| P7 | Adaptive Service Protection and Fraud Controls | [07](./07-Adaptive-Service-Protection-and-Fraud-Controls.md) |
| P8 | Agent Platform and MCP Governance | [08](./08-Agent-Platform-and-MCP-Governance.md) |

**Rules:**
- Only L6+ episodes from these 8 projects.
- Where none qualifies, use **Microsoft** (➡️ MS).
- No side stories (no DSS cleanup, no alert tuning, no CLM, no history-derived items).

---

## 1. Final allocation (primary = lead with it; backup = also L6+)

| Leadership Principle | Primary | Backup |
|---|---|---|
| 1. Customer Obsession | **P7**: fraud blocks → trusted 100× tier + self-serve IDV (758 verified, 97.3% success) | P8: Docusign inside Claude/ChatGPT/Copilot + citations, 50% faster · P4: no loss of access for $59M MRR |
| 2. Ownership | **P7**: my ramp blocked an enterprise customer → class fix + ramp mechanism | P3: end to end across 4 teams |
| 3. Invent and Simplify | **P1**: 51 allowlists + 13 auth classes → one taxonomy, SDK, approval flow | P2: one permission projected 3 ways · P5: shims + shadow mode |
| 4. Are Right, A Lot | **P6**: Redis (~1 ms) + change feed vs. Cosmos cache (~10 ms) / EventHub | P2: standards adopt/partial/defer calls · P8: MAF → LangGraph · P1: Entra POC |
| 5. Learn and Be Curious | **P8**: MCP + agent runtimes + eval → platform choices, MCP security model, readiness bar | P2: agent-identity standards → decision table |
| 6. Hire and Develop the Best | ➡️ **MS** | — |
| 7. Insist on the Highest Standards | **P3**: fail closed, void on failed association write, guard ahead of platform fix | P8: readiness checklist + eval gates (95% vs. 80%) · P4: 0 drift before cutover |
| 8. Think Big | **P2**: agents as a third identity type; 25-item, 4-scenario, P0–P4 program; one permission across REST, MCP, and agent tools | P8: every AI front door, one governed agent platform · P7: one limiting platform for every MSF service |
| 9. Bias for Action | **P8**: MCP Server Open Beta at Momentum (May) → GA Sep 30 (~4.5 months) | P6: GDS API on-prem first, DNS-flip cutover · P7: rate-limit POC |
| 10. Frugality | **P4**: ESDB $1.68M/yr → Postgres (~42% cheaper/vCore); parity scoped to migration | P2: deferred 3 items to fit 101–148 engineer-weeks · P5: −500 MB heap per workload |
| 11. Earn Trust | **P2**: exact revocation exposure + kill switch, not "instant" | P1: published where Entra was better before recommending against it |
| 12. Dive Deep | **P3**: whole-resource delete would silently revoke another team's access | P7: TOCTOU mode drift, counter resets, token-bucket burst |
| 13. Have Backbone; Disagree and Commit | **Backbone P7**: parity bar on my rate-limit platform before migrating eSign · **D&C P1**: Phase 1 skipped | P2: no agent traffic to unmigrated APIs · P5: the config service must not read secrets |
| 14. Deliver Results | **P8**: MCP GA Sep 30, 7 API groups, 6 AI clients; 5+ agents across 4 teams on one platform | P3: Workspaces + Maestro shipped across 4 teams · P4: Gate 1 signed off |
| 15. Strive to be Earth's Best Employer | ➡️ **MS preferred** | P5: removing 2,500+ requests/yr of config toil, safely |
| 16. Success and Scale Bring Broad Responsibility | **P7**: phishing vector; fraud-recipient rate 0.5% → < 0.2% target | P2/P8: agents on legal agreements need HITL, attribution, kill switch |

**Every project is used:** P1 ×2 · P2 ×2 (+5 backups) · P3 ×3 · P4 ×1 · P5 (backups) · P6 ×1 · P7 ×4 · P8 ×3 (+4 backups).

---

## 2. Coverage matrix (● primary · ○ L6+ backup)

| Leadership Principle | P1 | P2 | P3 | P4 | P5 | P6 | P7 | P8 | MS |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Customer Obsession | | | | ○ | | | ● | ○ | |
| Ownership | | | ○ | | | | ● | | |
| Invent and Simplify | ● | ○ | | | ○ | | | | |
| Are Right, A Lot | ○ | ○ | | | | ● | | ○ | |
| Learn and Be Curious | | ○ | | | | | | ● | |
| Hire and Develop the Best | | | | | | | | | ● |
| Insist on the Highest Standards | | | ● | ○ | | | | ○ | |
| Think Big | ○ | ● | | | | | ○ | ○ | |
| Bias for Action | | | | | | ○ | ○ | ● | |
| Frugality | | ○ | | ● | ○ | | | | |
| Earn Trust | ○ | ● | | | | | | | |
| Dive Deep | | | ● | | | | ○ | | |
| Have Backbone; Disagree and Commit | ● | ○ | | | ○ | | ● | | |
| Deliver Results | | | ○ | ○ | | | | ● | |
| Strive to be Earth's Best Employer | | | | | ○ | | | | ● |
| Success and Scale Bring Broad Responsibility | | ○ | | | | | ● | ○ | |

---

## 3. Common prompts → story

| Prompt | Use |
|---|---|
| Most impactful / largest-scope project | **P2** NHI program (25 items, 4 scenarios) · **P8** MCP GA on 6 AI clients · **P4** (scale) |
| A failure / mistake | **P7** BulkSend ramp blocked an enterprise customer |
| Disagreed with leadership | **P1** Phase 1 skipped (Disagree and Commit) |
| Disagreed with peers / held a bar | **P7** parity before migrating onto my rate-limit platform |
| Ambiguous problem | **P2** agents as an identity type · **P8** MCP + agent platform |
| Simplified something complex | **P1** 51 allowlists → 1 platform |
| Data-driven decision | **P6** Redis vs. Cosmos vs. EventHub · **P8** MAF → LangGraph |
| Changed your mind | **P3** IK-ownership → user-based model |
| Found a hidden problem | **P3** whole-resource delete bug |
| Speed vs. quality | **P8** Open Beta → GA in ~4.5 months · **P6** API-first · **P1** AuditOnly → Enforce |
| Raised the bar | **P3** fail-closed guards · **P4** 0-drift gate |
| Customer pain you fixed | **P7** trusted tier + self-serve IDV · **P4** no loss of access |
| Long-term vs. short-term | **P4** don't launch individual sharing before unifying |
| Saved cost | **P4** ESDB → Postgres |
| Security vs. availability | **P1** per-capability failover · **P2** fail closed for agents |
| Mentoring / hiring | **Microsoft** |

---

## 4. Spreading stories across a 5-person Amazon loop (avoid repeats)

| Interviewer | LPs | Stories |
|---|---|---|
| A | Customer Obsession, Frugality, Earn Trust | P7 (IDV), P4, P2 (NHI) |
| B | Think Big, Invent and Simplify, Disagree and Commit | P2 (NHI program), P1, P1 |
| C | Dive Deep, Highest Standards, Are Right | P3, P3, P6 |
| D (bar raiser) | Have Backbone, Ownership, Deliver Results | P7 (platform parity), P7 (ramp incident), P8 (MCP GA) |
| Hiring manager | Hire and Develop, Bias for Action, Learn and Be Curious | **MS**, P8 (beta → GA), P8 (platform learning), or swap L&BC to P2 |

---

## 5. Coinbase values → stories (Staff SWE, Customer Experience)

| Coinbase value | Stories |
|---|---|
| Act like an owner | P7 incident → class fix, P3 end to end |
| Top talent / championship team | **Microsoft** people story, P4 cross-team program |
| Customer focus | P4 no loss of access, P7 trusted limits |
| Repeated innovation | P2 agents as principals, P8 MCP governance |
| Efficient execution | P1 AuditOnly → Enforce, P6 API-first migration |
| Clear communication | P7 gap analysis, P6 consistency contract |
| Mission first (trust, security) | P7 fraud, P2 kill switch + attribution, P1 least privilege + audit |

**One-line CX translations:**
- P1 → least-privilege support tooling
- P2/P8 → safe AI support agents acting on accounts
- P3 → delegated actions on a customer's behalf
- P4 → zero-regression permission migrations
- P5 → auditable config and secrets
- P6 → global low-latency account lookups
- P7 → step-up KYC and risk-tiered limits

---

## 6. STAR timing (2–3 minutes)
1. **Situation (15 s):** scale + why it mattered.
2. **Task (10 s):** what *you* owned.
3. **Action (60–90 s):** 3 decisions as "options → choice → why," plus one tension you resolved.
4. **Result (20 s):** a number + the mechanism left behind + what you'd do differently.
