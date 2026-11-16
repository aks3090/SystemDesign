# 10 — L6+ STAR Answers (all from the 8 DocuSign projects)

**Rules for this file**
- Every answer comes from one of the 8 DocuSign projects (P1–P8) and is pitched at **L6+**: multi-team scope, real ambiguity or tension, a decision *I* made, a **mechanism** that outlived me, and an outcome with a number.
- If no DocuSign project clears that bar for a principle, it is marked **➡️ Use Microsoft story**. Don't force a weak one.
- **Number legend:**
  - ✅ = verified from docs, Jira, Slack, or the repo (sources at the bottom).
  - 📈 = projected or target, with its basis. Say "projected" or "on track to" out loud. Never present a projection as a measured actual.
- **Open with scope:** teams, customers, $ at stake. Then say what *you* decided. Use "I" for decisions and "we" for execution.

## Allocation (each project used for distinct episodes)

| # | Leadership Principle | Project | Episode |
|---|---|---|---|
| 1 | Customer Obsession | **P7** Service Protection (IDV) | Fraud limits were blocking paying customers → trusted tier + self-serve IDV |
| 2 | Ownership | **P7** Service Protection | Ramp blocked an enterprise customer → fixed the policy class |
| 3 | Invent and Simplify | **P1** 1P Access Mgmt | 51 allowlists → one capability platform |
| 4 | Are Right, A Lot | **P6** Global Directory | Redis + change feed vs. Cosmos cache / EventHub |
| 5 | Learn and Be Curious | **P8** Agent Platform + MCP | Learned MCP, agent runtimes, and eval deeply enough to set the platform and governance bar |
| 6 | Hire and Develop the Best | ➡️ **Use Microsoft story** | No L6-grade people story in the 8 projects |
| 7 | Insist on the Highest Standards | **P3** Maestro / Workspaces | Fail-closed, no orphan envelopes, loud failures |
| 8 | Think Big | **P2** NHI program | Agents as a third identity type: 25-item, 4-scenario, P0–P4 program |
| 9 | Bias for Action | **P8** Agent Platform + MCP | MCP Server Open Beta at Momentum → GA in ~4.5 months |
| 10 | Frugality | **P4** Unified Sharing | ESDB → Postgres, and parity scoped to what migration needs |
| 11 | Earn Trust | **P2** AuthZ / NHI | Exact revocation exposure, not "instant" |
| 12 | Dive Deep | **P3** Maestro / Workspaces | Delete call that would silently revoke another team's access |
| 13a | Have Backbone | **P7** Service Protection | Held the parity bar on the rate-limit platform I designed before migrating eSign |
| 13b | Disagree and Commit | **P1** 1P Access Mgmt | Phase 1 skipped; committed and re-planned |
| 14 | Deliver Results | **P8** Agent Platform + MCP | MCP Server GA Sep 30: 7 API groups, 6 AI clients, 5+ agents on one platform |
| 15 | Strive to be Earth's Best Employer | **P5** Config / Secrets *(or Microsoft)* | Removing org-wide config toil safely |
| 16 | Success and Scale Bring Broad Responsibility | **P7** Service Protection | Our product was a phishing vector |

**Backups (all L6+):** CO → P8 (work from any AI front door, citations, 50% faster) or P4 (no loss of access for $59M MRR) · Invent and Simplify → P2 (one permission projected 3 ways) · Are Right → P2 (standards adopt/defer calls) or P8 (MAF → LangGraph) · Highest Standards → P8 (readiness checklist + eval gates, 95% vs. 80%) · Think Big → P8 (every AI front door, one governed platform) · Bias for Action → P6 (API-first migration) · Frugality → P2 (deferred 3 items to fit 101–148 engineer-weeks) · Have Backbone → P2 (no agent traffic to unmigrated APIs) or P5 (config service must not read secrets) · Deliver Results → P3 (Workspaces + Maestro shipped) · Success & Scale → P8 / P2 (agents acting on legal agreements).

---

## 1. Customer Obsession — "Turning fraud blocks into a 5-minute self-serve unlock" (P7, IDV)

> 🧠 **Recall:** Rate-limit platform + Trusted Account / ID verification (P7). Fraud limits blocked paying customers; verified accounts now self-unlock elevated limits.
> **Key numbers:** 758 verified (97.3%) · 100× trusted limits · fraud recipients 0.5% → 0.15% · ~80 tickets/mo baseline


**Likely questions:** customer needs vs. a business or technical goal · went above and beyond for customers

**Headline:** "Our anti-fraud limits were blocking paying customers and pushing them into support queues. I designed and shipped the Trusted Account / ID-Verification integration. Verified customers now unlock 100× limits themselves in minutes, and fraudsters hit a wall they can't pass at scale."

- **S:**
  - Phishing through envelopes was rising, so we had imposed:
    - default send limits;
    - **5 recipients per envelope** on fraud-impacted plans ✅;
    - spam-filter suppression;
    - branding disabled for **~240K eligible accounts** ✅.
  - Legitimate paying customers got caught. About **80 support cases a month** were just limit increases ✅, each waiting on support SLAs.
- **T:** Give legitimate customers a self-serve path out, **without** giving fraudsters the same headroom.
- **A:**
  1. **Picked the right step-up signal.** Fraudsters can't pass government ID plus liveness at scale. So I made `IDV_STATE = VERIFIED` an **account-level** trust signal: one admin verifies, and every eligible user benefits.
  2. **Designed the integration end to end:**
     - RAdmin entry point.
     - Service Protection orchestration API with the IDV vendor flow.
     - Account verification state.
     - Limit-tier resolution (Default → Verified).
     - Spam-filter exemption.
     - IDV-gated features (Integrator Key promotion, next branding and bulk send).
  3. **Elevate, don't remove.** Verified accounts move up a tier (and trusted accounts get **100× limits** ✅), but plan allowances and billing still apply. Risk can rise again, and inactive accounts reverify after about 90 days.
  4. **Built abuse controls in from day one:**
     - **3 attempts**, then lockout, with a support reset path so no legitimate user is permanently locked out.
     - **A hashed ID can verify at most 5 accounts**.
     - IDs from fraud-closed accounts are blocked.
     - A TSOps override path (`IDV_Backfilled` / `IDV_Bypassed`) with SLAs.
  5. Made blocks **explainable**: distinct errors for a billing allowance vs. a protection limit, plus reset times, so customers and support know exactly what to do.
- **R:**
  - Trusted 100× limits in Prod **Apr 17 2026** ✅.
  - Verified accounts got higher limits **Jul 31** ✅, spam-filter exemption **Aug 10** ✅, and Integrator Key promotion **Oct 15** ✅.
  - **IDV self-serve 100% ramped in Prod (Sep 2026)**. **~150 accounts self-verified** in the first weeks, each one a support case that never happened ✅. Leadership called out the "high success rate" ✅.
  - **758 successful verifications, 97.3% success rate (758 of 779)** ✅.
  - Fraud-envelope recipient rate **~0.5% → 0.15% (−70%)**, beating the < 0.2% target ✅.
  - Limit-increase cases (~80/month baseline): **projected ≥ 50% reduction**, because about 250 self-serve unlocks a month now replace manual tickets 📈.
- **Reflection:** "I'd instrument 'time from block to unblock' on day one. That's the metric customers actually feel."
- **Backup (P4):** "No loss of access" while migrating **40.5M sharing configs** behind **$59M MRR**. Cohort cutover only at **0 drift**, never fall back on a deny, rollback rehearsed, and the public API unchanged for 33 partners ✅.


### Numbers to defend

| Claim | Basis |
|---|---|
| 758 / 779 = **97.3%** IDV success | Anti-Fraud tracker (Oct 2026) ✅ |
| Fraud-recipient rate **0.5% → 0.15% (−70%)** | Anti-Fraud tracker, Sep 26 – Oct 26 2026 ✅ |
| **100×** limits for trusted accounts, Prod Apr 17 2026 | Support release note CSRM-884 ✅ |
| ~80 limit-increase cases/month baseline | Trusted Account PRD ✅ |
| ~250 self-serve unlocks/month | 758 verifications ÷ ~3 months (Jul 31 – Oct 27) 📈 |
| ≥ 50% fewer limit tickets | Self-serve volume ≈ 3× the ticket baseline 📈 |

### Likely follow-ups (with answers)
1. **Why not just raise limits for everyone?** Fraudsters would get the same headroom. IDV is a step-up that legitimate customers pass and fraud rings can't scale (government ID + liveness).
2. **What about customers who won't share an ID?** Trust & Safety Ops can review and set `IDV_Backfilled` / `IDV_Bypassed` (1 business day to respond, 3 to resolve). Nobody is forced through IDV.
3. **How did you stop IDV itself being abused?** 3 attempts then support reset; a hashed ID verifies at most 5 accounts; IDs from fraud-closed accounts are blocked; vendor document-reuse checks.
4. **How do you know customers were better off?** 97.3% pass rate, 758 self-serve unlocks replacing tickets, and fraud still fell 70%. Friction went down while protection went up.
5. **What would you do differently?** Instrument time from block to unblock from day one, and ship the limit multiplier (instead of a fixed tier) earlier.

### 2-minute spoken version
"Our anti-fraud send limits were catching paying customers. About 80 a month were filing tickets just to raise limits, and roughly 240 thousand accounts had branding disabled. Raising limits for everyone would have helped fraudsters too. So I designed a trust-based path. Fraud rings can't pass government ID plus liveness at scale, so I made identity verification an account-level trust signal: one admin verifies and the whole account moves to an elevated tier. Limits are elevated, not removed, and billing still applies. I built in abuse controls: three attempts, one ID for at most five accounts, fraud-linked IDs blocked, plus a manual Trust and Safety path for anyone who won't share an ID. Trusted accounts got 100x limits, IDV went 100% live in production, and 758 accounts verified with a 97.3% success rate. At the same time, the fraud-envelope recipient rate dropped from about 0.5% to 0.15%. Less friction and more protection at the same time."

---

## 2. Ownership — "My ramp blocked an enterprise customer; I fixed the policy class" (P7)

> 🧠 **Recall:** Bulk-send fraud check (P7). My percentage ramp blocked a direct enterprise customer; I excluded all direct customers and made segment exclusions part of every ramp.
> **Key numbers:** Demo 25 → 50 → 100%, AU 25% · same-day fix (EMT-8388) · 48M limit evaluations/week


**Likely questions:** a mistake you made · owning a failure end to end

**Headline:** "A fraud-limit ramp I owned blocked a direct enterprise customer. I owned it publicly, unblocked them the same day, fixed it for every direct customer, and changed how limit ramps are targeted."

- **S:** I own eSign's send-limit enforcement. I built an **upfront atomic cumulative-recipient check for BulkSend** to close a "split into many batches" fraud evasion, and ramped it by percentage: Demo 25 → 50 → 100% ✅ (EMT-7768), AU 25% ✅.
- **T:** Support escalated that a **direct (enterprise) customer was blocked on bulk sends, even after being marked Trusted**. Policy exempts direct customers from send limits, except the 20K cap for accounts under 30 days old.
- **A:**
  1. **Owned it in-thread immediately.** No deflecting to "the rule engine."
  2. Root-caused it from trace tokens: my ramp targeted **all accounts by %**, not **by segment**.
  3. Got the customer unblocked the same day, then **changed the ramp to exclude every direct customer**, matching existing BulkSend enforcement (EMT-8388) ✅.
  4. Turned it into a **mechanism**. Every limit ramp now states *who must never be affected* (segment exclusions), includes a parity check against existing enforcement, a telemetry query, and a rollback step.
- **R:**
  - Same-day unblock ✅. Class-level fix shipped (EMT-8388) ✅.
  - The fix protects **every direct (enterprise) account** on the BulkSend path, while trusted-path limit evaluation runs at **~48M per week** ✅.
  - **Zero repeat** segment-targeting incidents since the ramp checklist was adopted 📈.
- **Reflection:** "Percentage ramps protect you from bugs. Segment exclusions protect you from policy mistakes. I design both up front now."


### Numbers to defend

| Claim | Basis |
|---|---|
| Ramp ladder Demo 25 → 50 → 100%, AU 25% | Jira EMT-7768 ✅ |
| Policy: direct customers exempt except 20K cap for accounts < 30 days | Support thread ✅ |
| Fix shipped to exclude **all** direct customers | EMT-8388 ✅ |
| Trusted-path limit evaluations ~48M/week | SPR-3010 ✅ |
| Same-day unblock | Support channel timeline ✅ |

### Likely follow-ups (with answers)
1. **How did you find out?** Support escalated in the limits channel. I traced it with trace tokens within the hour: the batch-throttle flag was on and the ramp targeted all accounts by percentage.
2. **Why didn't shadow mode catch it?** Shadow compared decisions against legacy logic, not against business segments. Legacy BulkSend already excluded direct customers, but the new check didn't. The parity check now includes segment exclusions.
3. **Did you blame anyone?** No. I owned the flag, so I owned it publicly in-thread and fixed it the same day.
4. **What changed permanently?** Every limit ramp now lists who must never be affected, runs a parity check against existing enforcement, attaches a telemetry query, and has a rollback step.
5. **How do you know it worked?** No repeat segment-targeting incidents since 📈, and the fix covers every direct account, not just the one that complained.

### 2-minute spoken version
"I own eSign's send-limit enforcement. I built an upfront check for bulk sends to close a fraud evasion where people split sends into many small batches, and ramped it by percentage in Demo and AU. Support escalated that a direct enterprise customer was blocked, even after being marked trusted, and our policy exempts direct customers. I owned it in the thread immediately. Within the hour I traced it: my ramp targeted accounts by percentage, not by segment. We unblocked the customer the same day. Then I fixed the class, not the case: the ramp now excludes every direct customer, matching existing bulk-send policy. And I changed the mechanism: every limit ramp now has an explicit 'who must never be affected' list, a parity check against existing enforcement, a telemetry query, and a rollback. The lesson I took: percentage ramps protect you from bugs, segment exclusions protect you from policy mistakes, and you need both."

---

## 3. Invent and Simplify — "51 allowlists → one capability platform" (P1)

> 🧠 **Recall:** 1P Access Management (P1). A portal plus an in-process SDK decide which internal app may call which eSign action.
> **Key numbers:** 51 allowlists + 13 auth classes → 1 · 13 capabilities · 5 callers · < 10 µs · kill switch < 2 min


**Headline:** "Every first-party integration added another hard-coded allowlist. I replaced that pattern with one capability platform, so onboarding became a config change instead of a deploy."

- **S:** Maestro, Webforms, Workspaces, and Agreement Desk called eSign with app tokens or PRTs. Access was governed by **51 DSS allowlists across 22 files** ✅ and **13 copy-pasted `RestApiInternal*AuthenticationAttribute` classes** ✅. Each was all-or-nothing, onboarding needed a deploy or a manual production DSS override, and there was **no audit trail** (a SOC 2 gap).
- **T:** I authored and led the 1P Access Management design (EMT-7755): fine-grained, auditable, **zero added latency**.
- **A:**
  1. **One taxonomy:** `Service:Action`, with separate grants per token type (AppToken vs. PRT).
  2. **One SDK** (.NET Standard 2.0) for both the **.NET 4.8 monolith and .NET 8 MSF**. It evaluates **in-process in < 10 µs** using an atomic `Interlocked.Exchange` swap of a lock-free cache.
  3. **Checks at both ends:** when the Participant Service mints a token, and when eSign REST consumes it.
  4. **One governance flow:** request → approve (requester ≠ approver) → audit, with a target-owned capability catalog.
  5. **Simple to operate safely:** `EnforcementMode` (AuditOnly / Enforce) and an `EmergencyDenyList` kill switch in **< 2 min, no deploy** ✅.
- **R:**
  - Approved across eSign, Participant Service, 1DS, and Security ✅.
  - **5 first-party callers** on the governed app-token path ✅ and **13 capabilities** in the catalog ✅.
  - **13 bespoke auth classes → 1 SDK filter**. **~20 of 51** DSS allowlists retired in wave 1 📈.
  - Onboarding time **3–5 days → < 1 business day** 📈. **100%** of grants auditable ✅.
- **Reflection:** "The final simplification was reusing DUCS segments on the request path instead of adding a new service."


### Numbers to defend

| Claim | Basis |
|---|---|
| 51 DSS allowlists across 22 files | Core repo scan ✅ |
| 13 bespoke `RestApiInternal*` auth classes → 1 SDK filter | Repo scan ✅ / 📈 wave 1 |
| 13 capabilities in the catalog; 5 callers onboarded | Design docs; app-token tracker ✅ |
| < 10 µs evaluation, 0 network calls | Design (lock-free in-memory dictionary) ✅ |
| Kill switch < 2 min, no deploy | Execution strategy ✅ |
| Onboarding 3–5 days → < 1 business day | 📈 approval SLA |

### Likely follow-ups (with answers)
1. **Why not just keep allowlists in config?** Allowlists were all-or-nothing per client and had no audit trail. The simplification was one capability taxonomy and one evaluation path; where it's stored is secondary (we later reused DUCS segments).
2. **How does the SDK stay current without a network call per request?** A background poll every 2 hours with an atomic swap of the in-memory cache. Revocation goes through an emergency deny list that is checked before the cache.
3. **What happens if the policy service is down?** The SDK serves the last-known-good cache. Failover is per capability: reads fail open, system writes fail closed.
4. **Why not buy Entra?** A POC showed client-ID ↔ service-principal mapping pain, Graph throttling during IIS recycles, and flat groups that can't express capabilities.
5. **How do you stop someone approving their own access?** Requester ≠ approver, enforced in both the API and the UI; the target team owns its catalog.

### 2-minute spoken version
"Every first-party integration with eSign added another hard-coded allowlist. We had 51 of them across 22 files, and 13 near-identical auth classes. Each was all-or-nothing, onboarding needed a deploy or a manual production change, and there was no audit trail. I replaced that pattern with one platform. A single taxonomy, Service:Action. A single SDK that runs in both the .NET Framework monolith and .NET 8 microservices and evaluates in-process in under 10 microseconds, with no network call. One approval flow with separation of duties. Checks happen both when a token is minted and when it's used. Operating it is simple too: one enforcement-mode switch and an emergency deny list that works in under two minutes without a deploy. Five first-party callers and 13 capabilities now run on it, onboarding is projected to drop from days to under a day, and every grant is auditable."

---

## 4. Are Right, A Lot — "Picking the cache and propagation design with numbers, not preference" (P6)

> 🧠 **Recall:** Global Directory (P6). Global source of truth for users and orgs; writes go to the home region, reads come from regional Redis, the change feed invalidates.
> **Key numbers:** 5 regions · Redis ~1 ms vs. Cosmos ~10 ms · ≥ 95% hit target · < 5 s propagation


**Headline:** "For Global Directory's multi-region reads, the team defaulted to Cosmos-native caching and EventHub. I showed with latency and locality numbers why Redis plus the Cosmos change feed was right, and defined the consistency contract callers rely on."

- **S:**
  - Identity data (users, memberships, orgs) was moving from **~8 site-partitioned eSign DBs** plus PubSub-synced Central DBs ✅ to a cloud Global Directory.
  - Reads had to be fast in every region. Writes needed a single authority.
  - The "User Model Problem" (one email, many users) caused a pick-which-user login challenge ✅.
- **T:** Choose the cache, the propagation path, and the consistency guarantees.
- **A:**
  1. **Redis over Cosmos-native options.** A Redis read is **about 1 ms vs. about 10 ms** for a Cosmos point read ✅. The integrated cache still pays **about 10 ms on a miss** ✅. A foreign-copy-with-TTL approach doesn't fix cross-region latency.
  2. **Cosmos change feed over EventHub/Service Bus** as the cross-region notification bus, because EventHub and Service Bus **lack global locality** ✅. Redis writes at about 1 ms let the processor keep up. The scim-service precedent was **20–100 ms per Service Bus write** ✅.
  3. **Writes go to the authoritative region** (no last-writer-wins on identity fields). Uniqueness ("one principal per email") is enforced with an **etag-conditional write to the GlobalIndex** ✅.
  4. **Invalidate in `finally`**, so even partially failed writes can't leave a stale cache. Tests prove invalidation on failed saves ✅.
  5. **Explicit contract:** read-your-writes in your own region; other regions eventually consistent; security-sensitive reads go to the authority.
- **R:**
  - Design adopted ✅.
  - Live in **5 residency regions** (NA, CA, AU, EU, IL4) ✅.
  - Cache-hit reads **~10 ms → ~1 ms (−90%)** ✅. Intercontinental 150–200 ms hops avoided on hits 📈.
  - **≥ 95% hit-rate target** (caller-side cache ran ~96.5% ✅). Propagation **< 5 s p99** 📈.
- **Backup (P1, build vs. buy):** an Entra POC exposed three blockers (IK↔Service Principal mapping, Graph throttling during IIS recycles, flat groups). The build decision held up ✅.
- **When I was wrong (P3):** I switched Maestro from IK-ownership to a **user-based model** once integration showed downstream calls needed the real user's permission. Shipped as a CAB patch (EMT-8869) ✅.


### Numbers to defend

| Claim | Basis |
|---|---|
| Redis read ~1 ms vs. Cosmos point read ~10 ms | Global Directory caching design ✅ |
| Integrated-cache miss still ~10 ms | Caching design ✅ |
| Service Bus write 20–100 ms (scim precedent) | Caching design ✅ |
| 5 residency regions (NA, CA, AU, EU, IL4) | Global Directory design ✅ |
| Caller-side cache ran ~96.5% hit rate | Incident thread ✅ (supports a ≥ 95% target 📈) |
| Propagation < 5 s p99 | 📈 target |

### Likely follow-ups (with answers)
1. **Why not multi-region writes?** Identity and membership need one authority; last-writer-wins on security fields is unacceptable. Writes route to the home region.
2. **What if Redis is down?** Fall through to the database at ~10 ms. Never fail a read because of the cache. Coalesce requests to avoid a stampede.
3. **Out-of-order change-feed events?** Compare versions and apply only newer. When in doubt, invalidate. A TTL is the backstop.
4. **How do you avoid stale membership on security checks?** Security-sensitive reads go to the authoritative region or bypass the cache. The consistency contract says so explicitly.
5. **Were you ever wrong?** Yes: on Maestro I moved from an app-ownership model to a user-based model once integration showed downstream calls needed the real user's permission.

### 2-minute spoken version
"For the Global Directory we needed fast identity reads in five residency regions, while writes needed a single authority. The default instinct was Cosmos-native caching and Event Hubs. I decided with numbers. A Redis read is about 1 millisecond against about 10 for a Cosmos point read, and Cosmos's integrated cache still costs about 10 milliseconds on a miss, so it doesn't fix latency. For cross-region invalidation, Event Hubs and Service Bus lack global locality, while the Cosmos change feed is already multi-region. So: writes go to the authoritative region, Redis serves each region, and the change feed is the invalidation bus. We invalidate in a finally block so partial failures can't leave stale data, and enforce uniqueness with conditional writes. I also wrote down an explicit contract: read-your-writes in your own region, eventual consistency elsewhere, and security reads go to the authority. Cache hits are about 90% faster, and teams know exactly what consistency they get."

---

## 5. Learn and Be Curious — "Going deep on MCP, agent runtimes and evaluation to set the bar for the whole platform" (P8)

> 🧠 **Recall:** Agent Platform + MCP (P8). Shared agent platform plus the Docusign MCP Server for external AI assistants.
> **Key numbers:** Beta May → GA Sep 30 2026 · 7 API groups · 6 AI clients · 5+ agents / 4 teams · 95% vs. 80% eval bar


**Likely questions:** something you learned fast that mattered · how you stay ahead technically

**Headline:** "When agents and MCP arrived, nobody here had shipped them at scale. I went deep on the MCP protocol and its OAuth model, agent runtimes, and LLM evaluation. That became our platform choices, the MCP Server's security model, and the readiness bar every agent team now ships against."

- **S:**
  - Product teams (Agreement Desk, Agent Studio, IAM, Platform) wanted agents now. Customers wanted Docusign inside Claude, ChatGPT, Copilot and Slack.
  - We had **300M+ agreements** and **40K IAM customers** at stake ✅.
  - There was no shared runtime, no evaluation practice, and no MCP security model.
- **T:** Turn a fast-moving, unfamiliar space into decisions the whole company could build on.
- **A:**
  1. **MCP:** studied the spec in both directions (outbound server, inbound connectors) and its OAuth step-up semantics. A 403 with `insufficient_scope` (RFC 6750) plus `resource_metadata` (RFC 9728) became our **denial contract**, which every MCP client already understands.
  2. **Runtimes:** compared Microsoft Agent Framework with LangGraph on agent patterns, ecosystem and lock-in. We **standardized on LangGraph** and migrated existing agents ✅.
  3. **Evaluation:** learned dataset and scorer methods (single- and multi-turn, tool-call correctness, citation faithfulness). Set the release bar at **95%+ task success vs. an 80% baseline** 📈.
  4. **Turned learning into artifacts:**
     - "MCP is the integration layer, not authorization."
     - Verb-level permissions.
     - Pre-prompt tool filtering.
     - Risk tiers with human approval.
     - The **production-readiness checklist**.
- **R:**
  - MCP Server **Open Beta May 2026 → GA Sep 30 2026**, with **7 API groups** reachable from **6 AI clients** ✅.
  - **5+ production agents across 4 teams** on one platform ✅. Iris made complex MNDA summarization **50% faster** in user research ✅.
- **Reflection:** "Learning only scales when it becomes a checklist other teams can ship against without me in the room."


### Numbers to defend

| Claim | Basis |
|---|---|
| MCP Open Beta May 2026 → GA Sep 30 2026 | GA announcement ✅ |
| 7 API groups; 6 named AI clients | Developer docs; GA announcement ✅ |
| 5+ agents across 4 teams; MAF → LangGraph | Agent platform newsletter / FAQ ✅ |
| 50% faster MNDA summarization | EAP research report, Aug 2026 ✅ |
| 95% task-success release bar vs. 80% baseline | 📈 proposed eval gate |

### Likely follow-ups (with answers)
1. **What did you learn that changed a decision?** MCP's OAuth step-up semantics: a 403 with `insufficient_scope` is something every MCP client already handles, so that became our denial contract. And LangGraph's pattern support led us off Microsoft Agent Framework.
2. **How did you learn so fast?** Read the specs, built small prototypes against real clients, studied how other platforms did evaluation, then wrote decision docs others could challenge.
3. **What was hardest?** Evaluation. Agent quality isn't a single number, so we used multi-turn scorers, tool-call correctness and citation faithfulness.
4. **How do you spread what you learn?** As artifacts: the readiness checklist, the governance model, and the denial contract. Teams ship against them without needing me.
5. **What are you learning now?** Agent identity standards (OAuth Actor Profile, cross-app access) and cost-aware model routing.

### 2-minute spoken version
"When agents and MCP showed up, nobody here had shipped them at scale, and customers wanted Docusign inside Claude, ChatGPT and Copilot right away. I went deep on three things. First, the MCP protocol in both directions and its OAuth model. The step-up response every MCP client understands became our denial contract. Second, agent runtimes: comparing Microsoft Agent Framework with LangGraph, we standardized on LangGraph and migrated existing agents. Third, evaluation: datasets, multi-turn scorers, tool-call and citation checks, with a release bar of 95% task success against an 80% baseline. I turned that learning into artifacts: MCP is integration not authorization, verb-level permissions, pre-prompt tool filtering, human approval for state changes, and a readiness checklist. The MCP Server went from open beta in May to GA on September 30 with seven API groups on six AI clients, and five-plus agents from four teams now run on one governed platform."

---

## 6. Hire and Develop the Best — ➡️ **Use your Microsoft story**

> 🧠 **Recall:** Use your Microsoft people story.
> **Key numbers:** Named person · whole component handed off · hard feedback · promotion or bigger scope · one hiring beat


None of the 8 DocuSign projects has an L6-grade *people* story: a named person you grew to a promotion or bigger scope, hard feedback you gave, a hiring-bar decision, or a mentoring program. Partner enablement (onboarding guides, coaching another team on a client) reads as L5 here.

**L6 template for your Microsoft story:**
- **S:** "[Engineer], [level], strong at X but not yet owning designs or cross-team dependencies."
- **T:** "Deliver [project] *and* grow them into an owner instead of being the bottleneck."
- **A:**
  - Handed them a **whole component end to end** (design, review, rollout).
  - Weekly 1:1s on **trade-off thinking**, not just code.
  - Made them the point of contact with a partner team, and **coached before the hard conversations**.
  - Gave **direct feedback** when the bar was missed (e.g., no rollback plan).
- **R:** "[Promoted to __ / led the next design alone / now owns on-call for __]."
- **Plus one hiring beat:** "In a debrief I [held a no-hire with evidence / down-levelled / wrote rubric questions], because __."


### What a strong L6 answer must contain (fill from Microsoft)
- **A named person** + their level + the specific gap (e.g., "strong coder, never owned a cross-team design").
- **What you handed them:** a whole component or track, not tasks.
- **How you coached:** weekly 1:1s on trade-offs, pre-briefs before hard conversations, design-review practice.
- **Hard feedback:** one moment where you told them they missed the bar, and how they responded.
- **Outcome with evidence:** promotion, led the next design alone, now owns on-call or a service, other teams ask for them.
- **Hiring beat:** one debrief where you raised the bar (a no-hire with evidence, a down-level, or rubric questions you wrote).

### Likely follow-ups (prepare answers from your Microsoft story)
1. **How did you pick who to develop?** Potential plus the right opportunity at the right time.
2. **What did you do when they struggled?** Narrow the scope, pair on the hardest part, don't take it back.
3. **How do you give hard feedback?** Specific behavior, impact, and expectation, delivered privately and soon.
4. **How do you raise the hiring bar?** Calibrated rubrics, evidence-based debriefs, willingness to say no-hire.
5. **What did you learn about yourself as a mentor?** For example: "I delegated too late at first."

### 2-minute spoken skeleton
"[Name] was a [level] engineer who [gap]. On [project] I handed them [whole component] instead of doing it myself. Every week we spent 30 minutes on trade-offs, not code. Before [partner review] I pre-briefed them, then let them run it. When [missed bar], I told them directly: [feedback]. They [changed behavior]. Six months later they [promotion / led design / owned service]. Separately, in hiring I [debrief example]. What I learned: [lesson]."

---

## 7. Insist on the Highest Standards — "No orphan envelopes, no silent access loss" (P3)

> 🧠 **Recall:** Maestro / Workspaces delegated access (P3). eSign is the only enforcer; an app token acts on a consenting user's behalf.
> **Key numbers:** 4 teams · AuthZ calls 7 → 2 · 0 orphaned envelopes · 123K Maestro runs / 30 days


**Headline:** "When I moved Maestro to app tokens acting on a user's behalf, I set three non-negotiables: fail closed, never leave an unprotected envelope, and never let one integration silently remove another's access. I held them even when that meant shipping a guard ahead of a platform fix."

- **S:** An app token acting on behalf of a user is a **new impersonation primitive** on legal agreements, spanning **4 teams** (Maestro, Workspaces, Identity/AuthZ, Security).
- **T:** I authored the design and owned the eSign implementation.
- **A:**
  1. **Fail closed** on AuthZ deny, error, or no decision. **No opt-out:** enforcement triggers on principal type, not a query flag.
  2. **Atomicity:** if the association write fails, **void or roll back the envelope**. No orphans.
  3. **Client ID only from verified token claims**, never a header. Actor must be an active account member.
  4. **Shipped a multi-association guard** before the platform's unlink API existed, so a dangerous unassign fails loudly (see Dive Deep).
  5. AuditOnly before Enforce, a per-integration DSS mode, and an alerts page.
- **R:**
  - Shipped in **Sep 2026** (PRs #117683, #118061, #118142) ✅.
  - **Removed 5 of 7 originally planned AuthZ calls** ✅.
  - Maestro runs **123K workflows per 30 days** ✅, so about **120K+ envelopes a month** move off impersonation tokens at full ramp 📈.
  - **Zero** orphaned or unprotected envelopes by construction ✅.
- **Reflection:** "A standard is real only when it's a guard or a test, not a review comment."


### Numbers to defend

| Claim | Basis |
|---|---|
| 4 teams (Maestro, Workspaces, Identity/AuthZ, Security) | Design docs ✅ |
| AuthZ calls 7 → 2 on the creation path | Maestro design ✅ |
| AuthZ client timeout 3,000 ms, fail closed | Maestro design ✅ |
| 0 orphaned or unprotected envelopes | By construction (void on write failure) ✅ |
| Maestro 123K workflow runs / 30 days | IAM planning review ✅ |

### Likely follow-ups (with answers)
1. **Why void the envelope instead of retrying the write?** A just-created, unsent envelope has no external side effects. An unprotected envelope is a security hole, so voiding is the safe default and the caller retries idempotently.
2. **Doesn't failing closed hurt Maestro availability?** Only on the app-token path, which has no legacy behavior to preserve. User-token traffic is untouched. Availability without authorization isn't acceptable for legal documents.
3. **Why ship a guard before AuthZ's unlink API existed?** The bug would have silently removed another integration's access. A loud failure today beats a silent one waiting on another team's roadmap.
4. **How did you test it?** Negative tests for each failure path: missing consent, unallow-listed client, invalid resource type, AuthZ error, failed association write. Plus AuditOnly before Enforce.
5. **Did these standards slow delivery?** No. They were designed in from day one, and the design was simplified by removing 5 of the 7 originally planned AuthZ calls.

### 2-minute spoken version
"Moving Maestro to app tokens meant creating a new impersonation primitive, an app acting on a user's behalf, on legal agreements, across four teams. I set three non-negotiables. One: fail closed on any authorization deny, error or missing decision, with no opt-out flag; enforcement keys off the principal type. Two: never leave an unprotected envelope. If the association write fails after creation, we void the envelope. Three: one integration must never silently remove another's access. I found the platform's delete call removed every association on an envelope, so I shipped a guard that refuses risky unassignments, even before the platform fix existed. Client identity comes only from verified token claims, never from headers. We ran AuditOnly before Enforce. It shipped in September, the design dropped from seven AuthZ calls to two, and there are zero orphaned envelopes by construction, while Maestro runs 123 thousand workflows a month on this path."

---

## 8. Think Big — "Agents as a third kind of identity: a 25-item program across the platform" (P2)

> 🧠 **Recall:** Agent-identity (NHI) program (P2). AI agents became a first-class identity with an actor chain, the 4-way AND, and per-agent revocation.
> **Key numbers:** 25 work items · 4 scenarios · P0–P4 · 101–148 eng-weeks · 4–5 engineers · permission models 3 → 1


**Headline:** "Instead of bolting an 'agent scope' onto tokens, I made AI agents a first-class identity across Docusign: in the directory, in tokens, in authorization, and in revocation. One permission model governs agent tools, REST and MCP. I scoped it into a 25-work-item, 4-scenario program that the org could fund and execute."

- **S:**
  - Agents were arriving through three surfaces (agent-platform tools, the public API, MCP), each drifting to its own allow-lists.
  - The identity stack knew only Users and ClientApplications.
  - Revocation was user-wide (`RefreshTokenNotBefore` kills every app).
  - MCP was heading to GA on **6 AI clients** ✅.
- **T:** Define the identity and authorization foundation for **Docusign-owned and customer-owned, delegated and autonomous** agents, and a plan to land it.
- **A:**
  1. **Agent as a principal type:**
     - Agent identity in the Global Directory (GDS), with an owner/DRI.
     - Workload credentials (SPIFFE inside our mesh, token exchange for third parties) instead of shared secrets.
  2. **"Action, not tool":** one canonical permission `{subject, verb, resource, constraints}`, projected 3 ways: the consent grant (RAR), the runtime check (AuthZService), and the stored tuple.
  3. **Delegated check = 4-way AND:** consent ∧ token scope ∧ the user's own permission ∧ the owner's bounding permissions. An agent can never exceed its principal.
  4. **Migration is the security boundary:** an API accepts agent tokens only after joining the agent-token ramp. Enforcement ships as both a sidecar and a .NET library.
  5. **Program design:**
     - **25 work items, 4 scenarios, phases P0–P4**.
     - **101–148 engineer-weeks** with **4–5 engineers** over **2–3 quarters** ✅.
     - Explicit exit criteria per phase.
- **R:**
  - Program plan adopted ✅. GDS Agent identity **in progress, on track** in the H2 plan ✅. eSign live as the first enforcement point ✅.
  - Permission models across agent tools / REST / MCP: **3 → 1** ✅. Revocation blast radius: **all of a user's apps → one agent** ✅.
  - Addressable base: **40K IAM customers, $350M+ ARR** ✅.
- **Backup (P8):** "Every AI front door reaches Docusign agreements, and every agent runs on one governed platform." MCP GA on 6 clients; 5+ agents across 4 teams.


### Numbers to defend

| Claim | Basis |
|---|---|
| 25 work items, 4 scenarios, phases P0–P4 | Agentic & NHI requirements spec ✅ |
| 101–148 engineer-weeks, 4–5 engineers, 2–3 quarters | NHI spec ✅ |
| 3 items deferred (SSF/CAEP, CIBA, ID-JAG) | NHI spec ✅ |
| Permission models 3 → 1 | Design ✅ |
| 40K IAM customers, $350M+ ARR; MCP on 6 AI clients | Company data; GA announcement ✅ |

### Likely follow-ups (with answers)
1. **Why not just add agent scopes to tokens?** Scopes are static and coarse. They can't express 'this envelope, if the user can, within the owner's limits', and relationships change after the token is issued.
2. **How did you get the org to fund it?** I sized it (25 items, 101–148 engineer-weeks), sequenced it so Scenario 1 ships first, and tied it to MCP GA and IAM revenue.
3. **How do customer-owned agents fit?** Phase P4: OIDC federation plus token exchange as the default, SPIFFE federation for customers who already run SPIRE.
4. **What's the biggest risk?** Draft standards (OAuth Actor Profile) changing. We isolated them behind our own canonical permission so the wire format can change.
5. **What does success look like in a year?** Every agent-facing API in the ramp, Docusign-owned and customer agents governed by one model, and per-agent revocation within seconds.

### 2-minute spoken version
"Agents were arriving through three doors: our agent platform's tools, the public API, and MCP, which was heading to GA on six AI clients. Each door was drifting toward its own allow-lists, our identity stack only knew users and client apps, and revocation was user-wide. Instead of bolting on an agent scope, I made agents a first-class identity across Docusign. Agents get a directory identity with an owner, and workload credentials instead of shared secrets. One canonical permission becomes three artifacts: the consent grant, the runtime check, and the stored relationship. A four-way rule: an agent can never exceed what the user delegated, what the user can do, or what its owner allows. And migration is the security boundary: an API takes agent traffic only once it's ready. I scoped it into a program the org could fund: 25 work items, four scenarios, five phases, about 101 to 148 engineer-weeks with four or five engineers. Permission models went from three to one, and revocation went from every app a user has to one agent."

---

## 9. Bias for Action — "Open Beta at Momentum, GA in ~4.5 months" (P8)

> 🧠 **Recall:** Docusign MCP Server (P8). Lets Claude, ChatGPT, Copilot and others act on Docusign agreements.
> **Key numbers:** Open Beta May 2026 → GA Sep 30 (~4.5 months) · 7 API groups · 6 clients · 0 tool calls authorized from model text


**Headline:** "Customers wanted Docusign inside their AI assistants now, and the full scope would have taken many more months. I shipped the Docusign MCP Server as an **Open Beta at Momentum in May 2026**, learned from real clients, hardened it, and took it to **GA on Sep 30 2026** with 7 API groups across 6 AI clients."

- **S:** Competitors were moving fast. Customers wanted to act on agreements from Claude, ChatGPT and Copilot. Waiting for complete tool coverage and every governance feature risked missing the moment.
- **T:** Get a safe, useful MCP Server into customers' hands fast, without compromising security on consequential actions.
- **A:**
  1. **Scoped the beta to a safe core:**
     - Read- and draft-heavy tools.
     - OAuth with audience-bound tokens.
     - The same product permissions and AuthZ as REST.
     - State-changing tools gated.
  2. **Treated it as a two-way door:** beta flags, telemetry on tool selection and auth failures, and rate limits from the platform I designed (story 07).
  3. **Hardened between beta and GA:**
     - Evaluation gates.
     - Admin outside-in / inside-out toggles.
     - Denial contract.
     - Expanded API groups (including CLM).
  4. Ran it in parallel with the governance work rather than serially.
- **R:**
  - **Open Beta May 2026 → GA Sep 30 2026 (~4.5 months)** ✅.
  - **7 API groups**; **6 named AI clients** (Claude, ChatGPT, Microsoft Copilot, Perplexity, Gemini Enterprise, Slackbot) + any MCP client ✅.
  - **0** tool calls authorized from model text ✅.
- **Reflection:** "Beta taught us how real clients pick tools and shape payloads, which no design review could."
- **Backup (P6):** shipped the Global Directory API on-prem over existing stores first, so consumers moved before the data did; cutover became a DNS flip across **5 residency regions** ✅.


### Numbers to defend

| Claim | Basis |
|---|---|
| Open Beta at Momentum NYC, May 2026 | GA announcement ✅ |
| GA Sep 30 2026 (~4.5 months) | GA announcement ✅ |
| 7 API groups incl. CLM; 6 named AI clients | Developer docs; GA announcement ✅ |
| 0 tool calls authorized from model text | By construction ✅ |

### Likely follow-ups (with answers)
1. **What risks did you accept in beta?** Narrower tool coverage and beta-labelled reliability. Not security: same permissions as REST, audience-bound tokens, state changes gated.
2. **What did beta teach you?** How real clients pick tools and shape payloads, where auth flows broke, and which API groups customers wanted next (which is how CLM got into GA).
3. **How was it reversible?** Beta flags, the admin outside-in toggle, rate limits, and telemetry. We could narrow or switch it off per account.
4. **Would you do it again?** Yes. Waiting for full scope would have meant designing for imagined clients instead of real ones.
5. **Where did you NOT bias for action?** Consequential actions: no state-changing tool shipped without approval and idempotency.

### 2-minute spoken version
"At Momentum we committed publicly to agentic agreement workflows, and customers wanted to act on Docusign agreements from Claude, ChatGPT and Copilot. Full tool coverage and every governance feature would have taken many more months. So I shipped the Docusign MCP Server as an open beta in May with a deliberately safe core: read- and draft-heavy tools, OAuth with audience-bound tokens, the same permissions as our REST API, and state-changing tools gated. It was a two-way door: beta flags, per-account admin toggles, rate limits from our platform, and telemetry on how clients chose tools and where auth failed. Between beta and GA we hardened it with evaluation gates, the denial contract, and more API groups, including CLM, which customers asked for. We went to GA on September 30, about four and a half months later, with seven API groups on six named AI clients, and zero tool calls authorized from model text."

---

## 10. Frugality — "Migrating $59M MRR of sharing for less money and less scope than the legacy system" (P4)

> 🧠 **Recall:** Unified Sharing migration (P4). Moved legacy eSign sharing onto the AuthZ graph with no loss of access.
> **Key numbers:** ESDB $1.68M/yr → ~$1.14M/yr · $59M MRR · 40.5M configs · 13.5 FTE peak · 2.5 AuthZ engineers


**Likely questions:** accomplished more with less · cut cost without hurting customers · said no to scope

**Headline (10 s):** "On the Unified Sharing migration I cut cost three ways. **Infrastructure:** I moved list and index storage off a ~$1.7–2M-a-year store onto Postgres that is about 42% cheaper per vCore, projected to save **$0.5–0.9M a year**. **Scope:** we built parity only for what customers need in order to migrate, not every legacy behavior. **People:** I asked for 2.5 dedicated AuthZ engineers instead of a new team. That kept a $59M MRR migration inside a 13.5-FTE plan with no loss of access."

### S — Situation
- Legacy Shared Envelopes served about **21K accounts (≈ $59M MRR)**, with **40.5M sharing configurations** across **183.8K accounts**, plus Shared Access on **~89.6K (Manage) / ~24.2K (Send)** accounts ✅.
- The "shared with me" list and indexing depended on **ESDB, which cost $1.68M a year in prod and about $2M with demo and test** ✅.
- Stakeholders pushed for **full parity**: every legacy behavior, document visibility, template sharing, participant tokens, and admin "see everything" indexing. That would have multiplied scope across eSign, AuthZ, Data Replication and Search, on top of a program already peaking at **13.5 FTE** ✅.

### T — Task
Cut the program's run cost and build cost **without** violating the one non-negotiable: **no customer loses access**.

### A — Actions (each is a frugality decision with a trade-off)
1. **Right-size the storage, using numbers.**
   - Compared options per vCore. **Citus Postgres was about 42% cheaper than Hyperscale** ✅.
   - So the new Envelope Extended Data store (EED, the list and index store) runs on Postgres at about **$1.14M a year equivalent** to ESDB's prod footprint ✅.
   - Caveat I stated up front: the saving assumes our queries are as efficient as on the original store. I made **query-plan parity** part of the performance gate.
2. **Parity scoped to what migration needs, not everything legacy did.**
   - With Product and P&T, I classified every legacy behavior as **migration-blocking**, **workaround-able**, or **retire**.
   - **Built:** sender/recipient/signing-group access, Shared Access (Manage/Send), custody transfer as a permission, bulk-send permission check, cross-site access through the Global Directory instead of stub users.
   - **Deferred or excluded:** document-visibility replatforming, template sharing, participant-token migration, full participant migration, AuthZ platform scale work.
   - Niche workflows get **documented workarounds** instead of rebuilds.
3. **Don't index what we don't need.**
   - Indexing every account envelope to every admin would have blown up the AuthZ graph and the "shared with me" index.
   - Admins keep access through **permission profiles in Core**. Only explicit shares are written and indexed ✅.
4. **Keep commercial logic out of the new platform.**
   - Plan items and account settings stay at the eSign boundary. AuthZ never evaluates licensing ✅.
   - That avoids building and maintaining billing logic in a second system.
5. **Stop the bleeding early.**
   - At Gate 2 we **stop new enablements of legacy sharing** ✅, so the migration population stops growing while we build.
6. **Reuse, don't rebuild.**
   - Reused the existing AuthZ graph and REST client, the access-evaluator facade, DUCS flags for cohort ramps, and the resource-association pattern from Workspaces / Maestro.
   - No new services just for this migration, except the EED list store.
7. **People.**
   - Instead of asking for a new team, I asked for **2.5 dedicated AuthZ engineers** with a dotted line to the program DRI ✅.
   - Staffing scaled by phase rather than holding peak headcount throughout: **5 → 13.5 → 11 → 4–5 FTE** ✅.

### R — Result
- **Run cost:** ESDB **$1.68M/yr prod (~$2M all environments)** → EED on Postgres **≈ $1.14M/yr**, which is **$0.54M/yr saved in prod and up to ~$0.9M/yr across environments** 📈.
- **Build cost:** held to the **13.5-FTE peak plan**, with Q2 2027 cohort delivery and all-customer EOL in Q3 2027. That was possible because three multi-quarter workstreams (document visibility, template sharing, participant tokens) were scoped out of the critical path ✅ / 📈.
- **Customer outcome preserved:** no-loss-of-access gates (0 drift, rehearsed rollback) were unchanged, and the **33 partner integrations** need zero changes ✅.
- **Platform health:** no admin-indexing blow-up in the graph; AuthZ kept free of licensing logic ✅.

### Reflection
"Frugality isn't 'spend less', it's 'spend only where customers feel it.' I'd present the cost model and the parity matrix in the very first review. The parity debate cost us two review cycles that a single upfront table would have saved."

### Numbers you must be able to defend (the math)

| Claim | How you got it |
|---|---|
| $1.68M/yr | ESDB prod cost across all prod regions (P&T deep-dive cost thread) ✅ |
| ~$2M/yr | Prod plus demo and test environments ✅ |
| 42% cheaper | Citus Postgres vs. Hyperscale **per vCore** ✅ |
| ≈ $1.14M/yr | Equivalent-capacity EED on Postgres, assuming query efficiency parity ✅ |
| $0.54M/yr | $1.68M − $1.14M (prod only) |
| up to ~$0.9M/yr | Including demo/test, which are right-sized more aggressively than prod 📈 |

### Likely follow-ups (have these ready)
1. **"How did you decide what was migration-blocking?"** Usage data per legacy behavior (accounts using it, MRR attached), plus whether a workaround exists. Behaviors with real usage and no workaround were built. Niche ones got workarounds. Product signed off on the matrix.
2. **"Didn't deferring document visibility hurt customers?"** It stays on the legacy path and runs as a parallel track after GA. No customer loses a behavior they rely on during migration.
3. **"What if Postgres queries are slower?"** The saving assumes query parity, so the performance gate (list p99, AuthZ check p99 < 15 ms) must pass before cutover. If it fails, we scale vCores, and the cost model shows we still save money up to roughly a 1.5× vCore increase.
4. **"Why not keep ESDB and avoid migration risk?"** ESDB was the expensive piece, and keeping it meant two list stores forever. Retiring it is where both the savings and the simplification come from.
5. **"Where did you push back?"** On full parity, and on indexing every envelope for admins. Both would have added quarters and infra cost for little customer value.

### 2-minute spoken version
"Our legacy envelope-sharing system served about $59M of MRR and 40 million sharing configurations, and its list store alone cost about $1.7M a year in prod. The migration plan was drifting toward full parity with everything legacy did, on top of a 13.5-FTE program. I pushed for frugality on three axes. First, storage: I benchmarked options and moved the list store to Postgres, which was about 42% cheaper per vCore, projecting about half a million to nine hundred thousand dollars a year in savings, with query parity as a hard performance gate. Second, scope: with Product, I classified every legacy behavior as migration-blocking, workaround-able, or retire. We built only what customers need to move, and deferred document visibility, template sharing and participant tokens. We also refused to index every envelope to admins, which would have blown up the graph. Third, people: instead of a new team, I asked for 2.5 dedicated AuthZ engineers, and staffing ramped down by phase. The result: lower run cost, a plan that stayed at 13.5 FTE for the Q2 2027 cohort target, and zero change to our no-loss-of-access gates or to our 33 partner integrations."

- **Backup (P2):** NHI program: I cut the core build to **101–148 engineer-weeks** by deferring 3 items (SSF/CAEP, CIBA, ID-JAG), so Scenario 1 shipped in P1 instead of waiting on full zero-trust ✅.
- **Backup (P5):** about **500 MB of DSS heap per workload** removed, and Azure Key Vault reused instead of building a secret store 📈.

---

## 11. Earn Trust — "Exact revocation exposure, not 'instant'" (P2)

> 🧠 **Recall:** Agent-identity program (P2). Security asked how fast a rogue agent can be stopped.
> **Key numbers:** ≤ 1 access-token lifetime residual · kill switch immediate · < 5 s with SSF · revocation all apps → one agent


**Headline:** "Security asked how fast we could stop a rogue agent. Instead of implying 'instantly,' I gave them the exact residual exposure and a plan to shrink it. That precision earned their sign-off."

- **A:**
  1. Stated the truth. Access tokens are validated statelessly, so after refresh-token revocation **exposure = one access-token lifetime** ✅.
  2. Added mechanisms:
     - A **graph-wide kill switch**: one "disabled" fact on the AgentId short-circuits every check. Reversible, audit-preserving.
     - **RFC 7009 refresh-token revocation**.
     - SSF/CAEP to **one receiver per AuthZ shard**, targeting **< 5 s** ✅.
  3. Documented what P1 does **not** do (bearer tokens are audience-bound only, not DPoP-bound) ✅.
- **R:** Security and lead-engineer sign-off on the phased plan ✅. Per-agent revocation (RFC 7009) live in P1 📈. Exposure bounded to **≤ 1 access-token lifetime**, and **immediate** via the kill switch ✅.
- **Reflection:** "Trust comes from precise bounds, not optimistic adjectives."


### Numbers to defend

| Claim | Basis |
|---|---|
| Residual exposure after refresh-token revocation ≤ 1 access-token lifetime | Revocation design (RFC 7009 Phase 1) ✅ |
| Kill switch: immediate at the PDP | NHI spec ✅ |
| SSF/CAEP target < 5 s | NHI spec 📈 |
| Before: revocation was user-wide (`RefreshTokenNotBefore`) | Revocation design ✅ |

### Likely follow-ups (with answers)
1. **How did Security react?** They signed off because the bound was precise and the plan to shrink it was concrete, not because we promised zero.
2. **Why not opaque tokens with introspection for instant revocation?** That breaks existing APIs that validate tokens locally and puts heavy load on Account Server. It was rejected explicitly in the design.
3. **How did you document the limits?** Each phase has a 'stated honestly' line, for example: P1 tokens are audience-bound bearer, not DPoP-bound, and revocation is bounded by TTL.
4. **Tell me about a time you lost trust.** Use the bulk-send ramp (Ownership): I owned it publicly and fixed the class.
5. **How else do you build trust with partner teams?** Explicit ownership tables (who owns what) and 'what is NOT changing' sections in every cross-team design.

### 2-minute spoken version
"In the agent-identity review, Security asked the hardest question: if an agent goes rogue, how fast can we stop it? The easy answer was 'instantly'. It wasn't true. Access tokens are validated locally by every API, so even after we revoke the refresh token, an issued access token lives until it expires. I gave them the exact bound: at most one access-token lifetime. Then I gave them mechanisms to shrink it: a kill switch that denies every check for that agent immediately while preserving its history for forensics, standard refresh-token revocation so no new tokens are minted, and a push-based revocation channel targeting under five seconds. I also wrote down what phase one does not provide, such as proof-of-possession tokens. Security signed off because the risk was quantified and owned, not hand-waved. Revocation went from 'every app the user has' to 'one agent'."

---

## 12. Dive Deep — "The delete call that would silently revoke another team's access" (P3)

> 🧠 **Recall:** Maestro / Workspaces (P3). The 'delete envelope resource' call wiped every association on an envelope.
> **Key numbers:** 2 integrations at risk · guard shipped pre-launch · 0 silent access-loss incidents · plus the TOCTOU fix (P7)


**Headline:** "Reading the business logic rather than the API docs, I found that unassigning an envelope from one integration would silently strip another integration's access. I shipped a guard before it could reach production."

- **A:**
  1. "Delete envelope resource" deleted the **whole** AuthZ resource, not one association. A code comment said "until AuthZ has API to unlink."
  2. Once an envelope is both ticket- and workflow-associated, unassigning either **destroys both**, and the survivor silently starts returning 403. It was **latent only because `ResourceType.Workflow` was unused**, and Maestro would make it reachable.
  3. Requested association-level unlink from AuthZ, and **shipped a multi-association guard regardless**.
  4. Same instinct on P7: found a **time-of-check/time-of-use (TOCTOU) mode-drift bug** in the limit migration (the DSS mode was read twice per request, so a mid-request flip could double-count or skip) and fixed it by reading once per request.
- **R:** Zero silent access-loss incidents (caught pre-launch) ✅. TOCTOU fixed before ramp ✅.


### Numbers to defend

| Claim | Basis |
|---|---|
| Delete call removed the **whole** AuthZ resource, not one association | Business logic + code comment ✅ |
| Latent until `ResourceType.Workflow` was used (Maestro) | Design analysis ✅ |
| Guard shipped pre-launch → 0 silent access-loss incidents | ✅ |
| TOCTOU mode-drift fix (P7) | Rate-limit migration ✅ |

### Likely follow-ups (with answers)
1. **How did you find it?** By reading the unassign path end to end instead of trusting the API name. A comment said 'delete envelope resource until AuthZ has an unlink API', which made me ask what else lived on that resource.
2. **Why didn't tests catch it?** Only one association type existed, so no test had two associations. I added that scenario.
3. **Why not wait for the AuthZ unlink API?** It was on another team's roadmap. A guard that fails loudly was cheap and removed the risk immediately.
4. **How do you decide how deep to go?** When a change touches authorization or data ownership, I trace every write path. A silent failure there is the most expensive kind.
5. **Another example?** In the rate-limit migration, the mode flag was read twice per request. A mid-request flip could double-count. I fixed it by reading the mode once.

### 2-minute spoken version
"While designing Maestro's envelope access, I traced the unassign path in the business logic instead of trusting the API name. The call named 'delete envelope resource' deleted the entire authorization resource, every association on that envelope, not just one. A code comment said it was temporary until the platform had an unlink API. Today that was harmless, because only Agreement Desk tickets used associations. But Maestro was about to add workflow associations. Once an envelope had both, unassigning either would silently strip the other integration's access and it would start returning 403s with no alert. I raised it with the AuthZ team and asked for association-level unlink. But I didn't wait for their roadmap: I shipped a guard that refuses unassignment when an envelope has more than one association, so the failure is loud. Zero silent access-loss incidents reached production. Same instinct on the rate-limit migration, where I caught a check-then-use bug in how the mode flag was read."

---

## 13a. Have Backbone — "Parity before migration, even on my own platform" (P7)

> 🧠 **Recall:** Rate-limit platform I designed (P7). Insisted on parity with eSign's legacy limits engine before migrating.
> **Key numbers:** 7 gaps / 3 P0 · 10+ limits in 1 round trip · 4 domains · ~4,000 TPS platform


**Headline:** "I designed the Global Rate Limit platform. There was strong pressure to keep it minimal and move eSign's fraud limits onto it quickly. I insisted on a parity bar first, because a half-parity migration of fraud limits would either let fraud through or block good customers. I held firm on the true blockers, conceded the rest, and the platform became stronger for every service."

- **S:** eSign was the platform's largest, hardest consumer: **10+ protectors per send**, 30-day and 3-day windows, per-plan and per-age thresholds, and **thousands of per-account overrides**. Engineers on the platform pushed for "keep it simple, no client library, no new algorithms, ship the migration."
- **A:**
  1. Wrote a **7-gap parity analysis (3 P0)** ✅. The blockers:
     - N×RTT instead of 1 RTT.
     - **A single weight per call, but an envelope send is +1 envelope and +N recipients.**
     - Counters reset on `age_bucket` changes.
     - Token-bucket burst credit for dormant fraud accounts.
     - No standard decision logging.
     - YAML-only overrides.
  2. Proposed concrete designs, not just objections: one domain + `limit_type` + per-limit addends, multi-limit per descriptor, a stable account counter with client-side age thresholds, and an override store.
  3. **Conceded where the minimal view was right:** no new counting algorithm (shorter-window limits instead), and no heavyweight client library (decision data returned in the response).
  4. **Held firm** on per-limit weights and override management, and **deferred the migration ramp rather than ship without parity.**
- **R:**
  - **Batch / multi-limit evaluation shipped** ✅.
  - **Deploy-free config** and an **override store + UI** added to the platform ✅.
  - eSign's 4 domains (envelope, recipient, cumulative, email) now evaluate in **one round trip** ✅.
  - Platform now carries **~4,000 TPS average** with **2.5–12× Redis headroom** ✅, available to every MSF service.
- **Backup (P5):** argued that the unified config service **must not read secrets for callers**, because it would need read access to every Key Vault (which bypasses access control) and its cache would mix secrets with non-secrets. The door stayed closed ✅.


### Numbers to defend

| Claim | Basis |
|---|---|
| 7 gaps, 3 P0 | Gap-analysis doc ✅ |
| Legacy: 10+ protectors in 1 round trip | Gap analysis ✅ |
| eSign: 4 limit domains per send, mixed weights | Gap analysis ✅ |
| Thousands of per-account overrides | Gap analysis ✅ |
| Platform today: ~4,000 TPS average, 2.5–12× headroom | Service Protection thread ✅ |

### Likely follow-ups (with answers)
1. **Who disagreed, and why?** Platform engineers wanted a minimal surface: no client library, no new algorithms, ship the migration. That's a reasonable platform instinct.
2. **What did you concede?** No new counting algorithm (shorter-window limits instead), and no heavyweight client library (decision data in the response instead).
3. **What didn't you concede, and why?** Per-limit weights and override management. Without them a fraud-limit migration either lets fraud through or blocks good customers.
4. **Weren't you being a blocker?** I proposed concrete designs (one domain + limit_type + per-limit addends, multi-limit per descriptor, an override store), not just objections.
5. **What if they'd said no?** Keep eSign on the legacy path behind the mode flag and escalate with the data. Migrating without parity wasn't an option for fraud limits.

### 2-minute spoken version
"I designed the global rate-limit platform, and eSign was its biggest, hardest customer: more than ten limits per send, 30-day and 3-day windows, per-plan and per-account-age thresholds, and thousands of per-account overrides. There was real pressure to keep the platform minimal and just migrate. I didn't agree. I wrote a parity analysis with seven gaps, three of them blockers. The biggest: one weight per call, while an envelope send counts plus one envelope and plus N recipients. Also counters resetting when account age changed, burst credit for dormant fraud accounts, and no way to manage overrides without a deploy. I proposed concrete designs for each. I conceded where the minimal view was right: no new counting algorithm, no heavy client library. I held firm on per-limit weights and override management, and deferred the migration rather than ship without parity. Batch multi-limit evaluation, deploy-free config and an override store landed. eSign now evaluates all four domains in one round trip, and the platform runs about 4,000 TPS with plenty of headroom."

## 13b. Disagree and Commit — "Phase 1 skipped" (P1)

> 🧠 **Recall:** 1P Access Management (P1). Leadership skipped my fast GitOps Phase 1.
> **Key numbers:** Phase 1 ≈ 2 weeks to INT · Phase 2 ≈ 3–4 weeks · 4 parallel tracks · now 5 callers / 13 capabilities


**Headline:** "Leadership skipped my fast Phase 1 and went straight to the governed platform. I disagreed, said why, then committed fully and re-planned without losing a week."

- **A:**
  1. In design review, leadership decided Phase 1 (GitOps → Blob, about 2 weeks to INT) was "no different than DSS" and "adds no value to Phase 2" ✅.
  2. I argued for faster value and early de-risking of the SDK.
  3. Once decided, I **moved Phase 1 to an appendix**, re-sequenced into **4 parallel tracks**, and **kept the SDK interface stable**, so the policy source could change without consumers noticing.
- **R:** Phase 2 delivered in the planned **3–4 week** INT window 📈. Now live with **5 first-party callers** and **13 capabilities** ✅.
- **Reflection:** "They were right. Later we simplified even further by reusing DUCS."


### Numbers to defend

| Claim | Basis |
|---|---|
| Phase 1 plan: GitOps YAML → Blob, ~2 weeks to INT | 1P evaluation doc ✅ |
| Leadership: 'no different than DSS', 'adds no value to Phase 2' | Design review comment ✅ |
| Phase 2 plan ~3–4 weeks to INT governance | Eng design ✅ |
| Result: live with 5 callers, 13 capabilities | Tracker; design docs ✅ |

### Likely follow-ups (with answers)
1. **Did you escalate?** No. I made my case once, with data, in the review. It was a reversible scope call, not a safety issue.
2. **How did you show you'd committed?** I moved Phase 1 to an appendix the same day, re-sequenced into 4 parallel tracks, and never relitigated it.
3. **What did skipping Phase 1 cost?** Early SDK de-risking. I offset that by building the SDK track in parallel from day one against a stub policy endpoint.
4. **Were they right?** Largely yes. Phase 1 duplicated DSS, and we later simplified further by reusing DUCS segments.
5. **When wouldn't you commit?** If a decision created a security gap or customer harm. Then I'd escalate with data. This wasn't that.

### 2-minute spoken version
"For first-party access management I proposed two phases: a quick GitOps-to-Blob phase that could reach INT in about two weeks, then a governed platform with a database and an approval portal. In the design review, engineering leadership decided to skip Phase 1, reasoning that it was no different from our existing config allowlists and added no value to Phase 2. I disagreed once, with data: Phase 1 delivered value sooner and de-risked the SDK. When the decision went the other way, I committed fully that day. I moved Phase 1 to an appendix, re-planned into four parallel tracks (service, SDK, integration, UI), and kept the SDK interface stable so the policy source could change without consumers noticing. I recovered the de-risking by building the SDK in parallel against a stub. In hindsight they were largely right: we later simplified even further by reusing existing config segments. The platform is live with five first-party callers and 13 capabilities."

---

## 14. Deliver Results — "MCP GA on Sep 30, plus 5+ agents on one governed platform" (P8)

> 🧠 **Recall:** Agent Platform + MCP (P8). Hit the public MCP GA date while moving the platform to LangGraph.
> **Key numbers:** GA Sep 30 2026 · 7 API groups · 6 AI clients · 5+ agents / 4 teams · 50% faster NDA summarization


**Headline:** "I delivered the Docusign MCP Server to GA on schedule, **Sep 30 2026**, with 7 API groups on 6 AI clients. Over the same period I moved the agent platform to LangGraph and got 5+ production agents from 4 teams onto one governed platform."

- **S:** Momentum committed us publicly to agentic agreement workflows. Multiple teams depended on a platform that was mid-migration (Microsoft Agent Framework → LangGraph) and had no shared governance.
- **T:** Hit the public GA date and get teams shipping on one platform, safely.
- **A:**
  1. **One plan, three tracks:** the MCP Server (beta → GA), the platform (LangGraph migration, model gateway, knowledge search + citations), and governance (checklist, HITL, kill switch). Clear owners and weekly risk burn-down.
  2. **Cut scope, not safety.** I deferred lower-value tools rather than ship state changes without approval and idempotency.
  3. **Unblocked teams early:** migrated existing agents to LangGraph so new ones started on the target runtime, and onboarded Agreement Desk intake agents (Intake Selector, Form Filler, Ticket Summary) ✅.
- **R:**
  - **MCP GA Sep 30 2026** ✅; **7 API groups** ✅; **6 AI clients** ✅.
  - **5+ production agents across 4 teams** ✅; existing agents **migrated to LangGraph** ✅.
  - **50% faster** MNDA summarization with Iris (user research) ✅.
  - New agent tools approved in **≤ 1 week** against the checklist 📈.
- **Backup (P3):** Workspaces + Maestro shipped across 4 teams. **123K** Maestro workflow runs per 30 days now flow on the app-token pattern (📈 ~120K+ envelopes/month); a CAB patch delivered the model change ✅.


### Numbers to defend

| Claim | Basis |
|---|---|
| MCP GA Sep 30 2026, on the public date | GA announcement ✅ |
| 7 API groups; 6 AI clients | Developer docs; GA announcement ✅ |
| 5+ production agents across 4 teams; 3 Agreement Desk intake agents | Agent platform newsletter / FAQ ✅ |
| Existing agents migrated MAF → LangGraph | ✅ |
| 50% faster MNDA summarization | EAP research ✅ |

### Likely follow-ups (with answers)
1. **What was the biggest risk to the date?** The runtime migration happening at the same time as GA hardening. I ran them as separate tracks with clear owners and moved existing agents first, so new ones started on the target runtime.
2. **What did you cut?** Lower-value tools, never safety. No state-changing tool shipped without approval and idempotency.
3. **How did you coordinate four teams?** One plan with three tracks (MCP, platform, governance), weekly risk burn-down, and a shared readiness checklist as the definition of done.
4. **Did anything slip?** Admin controls (outside-in/inside-out toggles, agent-admin role) moved to the next quarter. GA didn't depend on them because the server-side safety was already there.
5. **What happened after GA?** Expanding tools and connectors, and the agent-identity P1 work (story 02).

### 2-minute spoken version
"At Momentum we publicly committed to agentic agreement workflows, which meant a public GA date for the Docusign MCP Server. Several product teams were also counting on an agent platform that was mid-migration between runtimes and had no shared governance. I ran it as one plan with three tracks: the MCP Server from beta to GA; the platform (moving to LangGraph, a multi-model gateway, knowledge search with citations); and governance (the readiness checklist, human approval, kill switch). We cut scope on lower-value tools but never on safety. We migrated existing agents first so new ones started on the target runtime. We hit GA on September 30 with seven API groups on six AI clients: Claude, ChatGPT, Copilot, Perplexity, Gemini Enterprise and Slackbot. Over the same period, five-plus production agents from four teams moved onto the governed platform, and Iris made complex NDA summarization 50% faster in user research."

---

## 15. Strive to be Earth's Best Employer — "Taking engineers and ops out of the config toil loop, safely" (P5) *(or use Microsoft)*

> 🧠 **Recall:** Runtime config & secrets (P5), or better, a Microsoft people story.
> **Key numbers:** 50+ PRs per ramp · 2,500+ ops requests/yr · 13% misconfig incidents · deploy 15 → 5 min


**Use Microsoft instead** if you have a people or team-health story (mentoring, burnout, inclusion, on-call culture). Amazon values those more for this principle. If you use DocuSign:

**Headline:** "Config changes at eSign meant 50+ PRs per flag ramp and 2,500+ manual ops requests a year. I moved our services onto unified config through shims and shadow mode, so engineers ship config in one step and ops stops being a human relay, with zero behavior change for anyone."

- **S:**
  - 5+ config systems. **50+ PRs plus manual ops work per flag ramp** ✅.
  - **2,500+ DOC requests a year** ✅.
  - **13% of incidents (12 in 6 months) were misconfigurations** ✅.
  - DSS cost **500 MB of heap per workload** and refreshed every 5 minutes on every workload ✅.
- **A:**
  1. **Zero-touch shims** for the legacy config APIs, so no team had to rewrite code.
  2. **Shadow mode** compares old and new values off the hot path. It uses a **bounded channel (capacity 200, drop-on-full)** so validation can never hurt production, and normalizes DateTime so false diffs don't erode trust ✅.
  3. Per-key move to DUCS-only once diffs were clean.
  4. Built new enforcement directly on DUCS segments instead of adding more one-off lists.
- **R:**
  - Program targets: DOC involvement **−100%**, deploy time **days → minutes**, ramp steps **50+ → 1**, misconfig incidents **−75%** ✅.
  - Achieved: **shadow mode 100% in Prod**, **DUCS 100% in Demo**, **time-to-deploy 15 → 5 min (−67%)** ✅.
  - On track: **50+ → 1** ramp steps and **2,500+ → ~0** DOC requests for migrated namespaces 📈.


### Numbers to defend

| Claim | Basis |
|---|---|
| 50+ PRs + manual ops per flag ramp | Runtime Config & Secrets deck ✅ |
| 2,500+ DOC config requests/year | Deck ✅ |
| 12 misconfig incidents in 6 months (13%) | Deck ✅ |
| Time-to-deploy 15 → 5 min | SRE notes Nov 2026 ✅ |
| Shadow 100% in Prod; DUCS 100% in Demo | SRE notes ✅ |

### Likely follow-ups (with answers)
1. **How is this about people, not systems?** The toil landed on humans: ops engineers handling 2,500+ manual requests a year, developers running 50+ PRs for one flag, and on-call responding to misconfiguration incidents.
2. **How did you make the change safe for the people adopting it?** Zero-touch shims so no team rewrote code, and shadow mode that can't hurt production.
3. **What feedback did you get?** Faster deploys (15 → 5 minutes) and fewer manual hand-offs. 📈 Survey or ticket data if you have it.
4. **(Better) Do you have a team-health or mentoring story?** Use your Microsoft people story here if you have one. Amazon weights that more for this principle.

### 2-minute spoken version
"Before, changing one config flag at eSign meant 50-plus pull requests and manual work by an ops team that handled over 2,500 config requests a year. Misconfiguration also caused 13% of our incidents, which meant pages for on-call. That's a people problem disguised as a systems problem. I moved our services onto unified configuration without asking any team to rewrite code: compatibility shims kept the old APIs working, and shadow mode compared old and new values off the hot path, built so it could never hurt production. We cut over namespace by namespace once the diffs were clean. Shadow mode is at 100% in production, the new system is 100% live in Demo, and time to deploy a config change dropped from 15 minutes to 5. We're on track to take a flag ramp from 50-plus steps to one, and the ops team out of the manual loop."

---

## 16. Success and Scale Bring Broad Responsibility — "Our product was a phishing vector" (P7)

> 🧠 **Recall:** Rate-limit platform + IDV (P7). Docusign envelopes were being used to phish non-customers.
> **Key numbers:** Fraud recipients 0.5% → 0.15% (−70%) · 758 verified / 97.3% · 100× trusted limits · 1.9M customers


**Headline:** "Docusign envelopes were being used to phish people who aren't even our customers. I closed specific evasion paths in our limits, while making sure legitimate senders weren't collateral damage."

- **S:** Org target: cut **fraud-envelope recipients from ~0.5% to < 0.2%** ✅.
- **A:**
  1. Built the **upfront atomic cumulative-recipient check for BulkSend**, closing the "split into batches" evasion. Ramped Demo 25 → 50 → 100% ✅.
  2. **Counted signing-group members** toward recipient limits, Learning mode first, so groups can't hide volume ✅.
  3. Balanced this with **100× limits for trusted accounts** ✅ and **direct-customer exclusions** ✅, so legitimate senders aren't punished.
- **R:** Fraud-envelope recipient rate **~0.5% → 0.15% (−70%)**, beating the **< 0.2%** target (Sep–Oct 2026) ✅. **758** verified accounts with a **97.3%** IDV success rate ✅.
- **Reflection:** "At our scale, every limit is a policy decision about strangers' inboxes *and* our customers' livelihoods."


### Numbers to defend

| Claim | Basis |
|---|---|
| Fraud-recipient rate 0.5% → 0.15% (−70%), target < 0.2% | Anti-Fraud tracker ✅ |
| BulkSend upfront atomic check, Demo 25 → 50 → 100% | EMT-7768 ✅ |
| Signing-group members counted (learning mode first) | DSS / design ✅ |
| 100× limits for trusted accounts; 758 verified at 97.3% | CSRM-884; Anti-Fraud tracker ✅ |
| Docusign: 1.9M+ customers, 1.1B+ signatures in FY26 | Corporate data sheet ✅ |

### Likely follow-ups (with answers)
1. **Who are the 'non-customers' you protected?** Phishing recipients: people who get a fraudulent Docusign envelope and trust it because of our brand.
2. **How do you balance protection with customer harm?** Progressive friction: verify, elevate and allowlist instead of blocking. Plus explicit segment exclusions, such as direct customers.
3. **What about privacy of ID verification?** Only a hashed ID key is stored, for reuse checks. Raw ID data stays with the IDV vendor flow. Verification is opt-in, with a manual alternative.
4. **How is the fraud rate measured?** Share of total recipients who receive envelopes later classified as fraud, from the Anti-Fraud tracker query, over a rolling window.
5. **What's next?** Verified accounts bypass the spam filter, IDV inside the spam-filter flow (V2), and a 'Verified Sender' signal for recipients.

### 2-minute spoken version
"Docusign serves over 1.9 million customers, and our brand is trusted. That makes our envelopes attractive to phishers. The people harmed are often not our customers at all: they're recipients. The org target was to cut fraud-envelope recipients from about half a percent to under 0.2%. I closed specific evasion paths in our limits. An upfront, atomic cumulative check stops attackers splitting bulk sends into small batches. We now count signing-group members so groups can't hide recipient volume. Then I balanced it so legitimate senders weren't collateral damage: trusted accounts get 100x limits, ID verification lets real customers unblock themselves (758 did, at 97.3% success), and direct customers are explicitly excluded. The fraud-envelope recipient rate dropped to 0.15%, about 70% lower, beating the target. At our scale every limit is a policy decision about strangers' inboxes and our customers' livelihoods, and I design for both."

---

## Verified numbers (✅ sources)

| Number | Source |
|---|---|
| ~21K accounts / $59M MRR; 183.8K accounts; 40.5M configs | P&T Deep Dive: Unified Envelope Sharing |
| 89.6K Manage / 24.2K Send; 33 partner integrations; SLOs (99.99%, 15 ms, 0.05%) | Eng Design PFPP-3291 |
| Gates 10/15, 1/15, 3/1, 4/15, 6/1; 13.5 FTE peak; 2.5 AuthZ FTE | Unified Sharing resourcing doc |
| ESDB $1.68M/yr (~$2M with demo/test); Citus ~42% cheaper per vCore; ~$1.14M/yr | P&T deep-dive comments |
| 51 DSS allowlists / 22 files; 13 auth attribute classes | Core repo scan |
| < 10 µs evaluation; < 2 min kill switch | 1P eng design + execution strategy |
| Redis ~1 ms vs. Cosmos ~10 ms; change feed vs. EventHub locality; scim 20–100 ms | Global Directory "Caching" design |
| ~8 sites; on-prem API first; DNS CNAME; Outbox + NDNS; GlobalIndex etag writes | Global Directory design |
| 5+ config systems; 50+ PRs per ramp; 2,500+ DOC requests/yr; 13% misconfig incidents; 500 MB heap | eSign Runtime Configuration & Secrets deck |
| Shadow channel capacity 200, DropWrite | `SysConfigOrDUCSSetting.cs` |
| 100× trusted limits (Prod Apr 17 2026); ~150 accounts self-verified; 0.5% → < 0.2% OKR | Support release note CSRM-884; Slack launch post; IDV PRD |
| BulkSend ramp 25/50/100%; EMT-8388 direct-customer exclusion | Jira EMT-7768; Slack support thread |
| 7 gaps / 3 P0; batch API shipped; override DB/UI committed | Your gap-analysis doc + Slack |
| 5 of 7 AuthZ calls removed; PRs #117683/#118061/#118142; EMT-8869 patch Oct 21 | Maestro design; git; release channel |
| MCP outbound/inbound; DAF agents; LangGraph migration | MCP & IAM guide; DAF newsletter; Agentic Admin PRD |
| **758 IDV successes / 779 = 97.3%**; fraud-recipient rate **0.15%** (Sep 26 – Oct 26 2026) vs. 0.5% baseline | Anti-Fraud tracker (INFSEC Confluence) |
| Rate-limit service **~4,000 TPS avg**; Redis P2 **10K–50K TPS** | Service Protection Slack thread |
| Trusted-path limit evaluations **~48M / 7 days** | Jira SPR-3010 |
| DUCS time-to-deploy **15 → 5 min**; shadow **100% Prod**; DUCS **100% Demo** | 2026-11-04 SRE notes |
| GDS in **5 residency regions** (NA, CA, AU, EU, IL4); caller-side cache **~96.5%** hit | Global Directory design; incident thread |
| Maestro **123K workflow runs / 30 days** (Q2) | IAM planning review |
| **1.9M+ customers; 1.1B+ signatures in FY26; IAM 40K customers, $350M+ ARR; 300M+ agreements in Agreement Manager; 47% of transactions via API** | Docusign corporate data sheet; FY26 earnings |
| 5 first-party apps on the app-token pattern | "All App Token Integrations" tracker |
| MCP Server **Open Beta May 2026 → GA Sep 30 2026**; **7 API groups**; **6 named AI clients** (Claude, ChatGPT, Copilot, Perplexity, Gemini Enterprise, Slackbot) | MCP GA announcement (SCMSKB); developer docs |
| NHI program: **25 work items, 4 scenarios, P0–P4, 101–148 engineer-weeks, 4–5 engineers, 2–3 quarters**; 3 items deferred | Agentic and NHI Requirements spec |
| Iris: complex MNDA summarization **50% faster** | Agentic Experiences EAP research report (Aug 2026) |
| Agreement Desk intake agents (Intake Selector, Form Filler, Ticket Summary); 5+ agents across 4 teams; MAF → LangGraph | DAF FAQ; DAF newsletter |

## Company-scale opener (use it in the first 15 seconds of any answer)
"Docusign serves **1.9M+ customers** and processed **1.1B+ signatures in FY26**, with **47% of transactions via API**. The platforms I built sit on that hot path."
