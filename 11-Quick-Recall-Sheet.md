# 11 — Quick-Recall Sheet (read this 10 minutes before the interview)

✅ = actual · 📈 = projected or target (say "projected" or "on track to")

## 0. Company numbers (your 15-second opener)
- **1.9M+ customers** · **1.1B+ signatures in FY26** · **47% of transactions via API** ✅
- **IAM: 40K customers, $350M+ ARR** · **300M+ agreements** in Agreement Manager ✅
- *"The platforms I built sit on that hot path."*

---

## 1. Project recall cards

### P1 — First-Party (1P) Access Management
- **What it is (one line):** a system that decides **which internal Docusign app** (Maestro, Workspaces, Webforms…) **may call which eSign action**, replacing hard-coded allowlists.
- **How it works:**
  - Teams request a capability (e.g., `Envelopes:EnvelopeRead`) in a portal; the target owner approves; the decision is stored in Cosmos.
  - An **in-process SDK** in eSign and in the Participant Service polls policy every 2 h and checks requests in **< 10 µs** with no network call.
  - Rollout went AuditOnly → Shadow → Enforce, with a kill switch.
- **My role:** designed it, wrote the build-vs-buy POC (Entra rejected), ran 4 delivery tracks.
- **Key decisions:**
  - In-process, not a network call.
  - Failover per capability: reads fail open, system writes fail closed.
  - Requester ≠ approver.
- **Metrics:** **51 allowlists / 13 auth classes → 1 platform** · **13 capabilities, 5 callers** ✅ · **< 10 µs, 0 RPCs** ✅ · **kill switch < 2 min** ✅ · **onboarding 3–5 days → < 1 day** 📈 · **100% of grants auditable** ✅
- **Story hooks:** Invent & Simplify · Disagree & Commit (leadership skipped Phase 1) · Are Right (Entra POC).

### P2 — AuthZ and Non-Human Identities (agent identity program)
- **What it is:** making **AI agents a first-class identity** so they can act for users safely, with an attributable actor chain, least privilege, and per-agent revocation.
- **How it works:**
  - The agent token carries `sub` = user, `act` = agent.
  - Each call is checked by AuthZService with a **4-way AND**: user consented ∧ token scope covers the call ∧ the user can do it ∧ the agent's owner allows it.
  - A kill switch and refresh-token revocation stop an agent.
  - An API accepts agent traffic only after it is migrated.
- **My role:** program owner and architect.
- **Key decisions:**
  - One canonical permission across agent tools, REST and MCP.
  - Fail closed.
  - Adopt RAR, token exchange and the Actor Profile; partial AuthZEN; defer SSF/CAEP and CIBA.
- **Metrics:** **25 work items, 4 scenarios, P0–P4** ✅ · **101–148 engineer-weeks, 4–5 engineers, 2–3 quarters** ✅ · **3 items deferred** ✅ · **permission models 3 → 1** ✅ · revocation **all apps → one agent**, exposure **≤ 1 access-token lifetime**, **< 5 s** with SSF 📈 · **ES256 saves ~190 B per token** ✅
- **Story hooks:** Think Big · Earn Trust (honest revocation bound) · Have Backbone (no agent traffic to unmigrated APIs).

### P3 — Maestro and Workspaces Delegated Access
- **What it is:** letting **Maestro (workflows)** and **Workspaces** access envelopes through the shared AuthZ graph, with **eSign as the only enforcer and the only writer of AuthZ records**.
- **How it works:**
  - **Workspaces:** user token; access is derived Envelope → Workspace → member.
  - **Maestro:** app token plus a one-time user consent. eSign checks consent, creates the envelope, then writes the association; if that write fails, the envelope is voided.
- **My role:** designed it and owned the eSign side across 4 teams.
- **Key decisions:**
  - Fail closed with no opt-out.
  - Client ID taken only from the token.
  - A guard against the whole-resource delete bug.
  - Later switched to a user-based model.
- **Metrics:** **4 teams** · **AuthZ calls 7 → 2** ✅ · **0 orphaned envelopes** ✅ · **123K Maestro workflow runs / 30 days** ✅ → **~120K+ envelopes/month** on the app-token path 📈 · **5 apps** on the pattern ✅ · next app onboards in **6–8 → ~2 weeks** 📈
- **Story hooks:** Highest Standards · Dive Deep (delete bug) · Deliver Results (backup) · Are Right (model switch).

### P4 — Unified eSign Sharing Migration to AuthZ
- **What it is:** moving eSign's legacy sharing (SQL tables) onto the AuthZ graph **without any customer losing access**.
- **How it works:**
  - Backfill → dual-write (SQL is primary) → reconciliation.
  - Shadow reads compare SQL with AuthZ.
  - Reads cut over per cohort only at **0 drift**, with rehearsed rollback.
- **My role:** technical owner across 5 teams.
- **Key decisions:**
  - Never fall back on a deny.
  - Plan items stay out of AuthZ.
  - Don't index every envelope to admins.
  - Parity only for what migration needs.
- **Metrics:** **40.5M sharing configs, 183.8K accounts** ✅ · **~21K accounts ≈ $59M MRR** ✅ · **89.6K / 24.2K** Shared Access ✅ · **33 partners, zero change** ✅ · **13.5 FTE peak** ✅ · SLOs **99.99% / < 15 ms / < 0.05%** ✅ · storage **$1.68M → $1.14M/yr** (save **$0.54–0.9M**) 📈 · cohorts by **Q2 2027** 📈
- **Story hooks:** Frugality · Customer Obsession (backup) · Highest Standards (0-drift gate).

### P5 — Runtime Configuration and Secrets Modernization
- **What it is:** replacing **5+ config systems** and scattered secret stores with **one GitOps-governed config service (DUCS)** and **Azure Key Vault**.
- **How it works:**
  - Compatibility shims keep the old APIs working.
  - Shadow mode compares old vs. new values asynchronously (bounded channel, drop when full).
  - Then flip to DUCS-only per namespace.
- **Key decisions:**
  - Shims instead of rewrites.
  - The config service never reads secrets for callers.
  - Last-known-good cache.
- **Metrics:** **50+ PRs per flag ramp** · **2,500+ ops requests/yr** · **12 misconfig incidents in 6 months (13%)** ✅ · **time-to-deploy 15 → 5 min** ✅ · **shadow 100% in Prod**, **DUCS 100% in Demo** ✅ · **500 MB heap per workload** reclaimed 📈
- **Story hooks:** Earth's Best Employer (fallback) · Have Backbone (secrets) · Invent & Simplify (backup).

### P6 — Global Directory and Multi-Region Identity Caching
- **What it is:** one **global source of truth for users, memberships and orgs**, replacing ~8 site-partitioned DBs, with fast regional reads.
- **How it works:**
  - Writes go to the authoritative region.
  - Each region reads from Redis.
  - The Cosmos change feed invalidates caches everywhere.
  - Etag-conditional writes enforce "one principal per email".
- **Key decisions:**
  - Redis, not Cosmos-native caching.
  - Change feed, not Event Hubs.
  - Invalidate in `finally`.
  - Explicit consistency contract.
- **Metrics:** **5 residency regions** ✅ · **~1 ms vs. ~10 ms** reads (−90%) ✅ · **≥ 95% hit-rate target** (a comparable cache ran ~96.5% ✅) · **< 5 s** propagation 📈 · "pick which user" login challenge eliminated ✅
- **Story hooks:** Are Right · Bias for Action (backup: API first, data later).

### P7 — Global Rate Limit Platform + Trusted Account (IDV)
- **What it is:**
  - A **rate-limit service for every MSF service** (Redis-backed, Envoy-compatible).
  - Moved eSign's fraud limits onto it.
  - Built **ID verification** so legitimate customers unlock higher limits themselves.
- **How it works:**
  - One gRPC call per send evaluates all limits (envelopes +1, recipients +N).
  - Shadow first, config-only deploys.
  - Risk ladder: Low → Medium → High → Verified.
- **Key decisions:**
  - Counters per data center.
  - Algorithm chosen per limit.
  - An unmatched limit returns OK.
  - Verification elevates limits, never removes them.
  - Abuse caps on IDV.
- **Metrics:** **~4,000 TPS**, **2.5–12× headroom** ✅ · **1–10 ms per check** ✅ · **7 gaps / 3 P0** ✅ · **758 verified, 97.3% success** ✅ · **fraud-recipient rate 0.5% → 0.15% (−70%)** ✅ · **100× trusted limits** ✅ · **48M limit evaluations/week** ✅ · tickets **~80/mo → ≥ 50% fewer** 📈
- **Story hooks:** Customer Obsession · Ownership (ramp incident) · Have Backbone (parity bar) · Success & Scale (phishing).

### P8 — Agent Platform and MCP Governance
- **What it is:** a shared **agent platform** (catalog, LangGraph runtime, model gateway, evaluation) plus the **Docusign MCP Server**, so customers can use Docusign from any AI assistant safely.
- **How it works:**
  - Tools are filtered before the model sees them.
  - The model only proposes; the PDP authorizes.
  - State changes need approval plus idempotency.
  - Kill switch and audit trust record.
- **Key decisions:**
  - Standardize on LangGraph.
  - Multi-model gateway.
  - MCP is not authorization.
  - Open Beta first.
- **Metrics:** **Open Beta May 2026 → GA Sep 30 2026 (~4.5 months)** ✅ · **7 API groups**, **6 AI clients** (Claude, ChatGPT, Copilot, Perplexity, Gemini Enterprise, Slackbot) ✅ · **5+ agents across 4 teams** ✅ · **50% faster** NDA summarization ✅ · **95% vs. 80%** eval bar 📈 · **2 s target / 60 s alert** ✅
- **Story hooks:** Bias for Action · Deliver Results · Learn & Be Curious · Think Big (backup).

---

## 2. LP → one-line hook + 3 numbers

| LP | Hook (say this first) | 3 numbers |
|---|---|---|
| Customer Obsession | "Fraud limits were blocking paying customers. I built self-serve ID verification." | 758 verified / 97.3% · 100× limits · fraud −70% |
| Ownership | "My ramp blocked an enterprise customer. I fixed it for every direct customer." | 25 → 50 → 100% ramp · same-day fix · 48M evals/week |
| Invent & Simplify | "51 allowlists became one platform." | 51 → 1 · < 10 µs · < 2 min kill switch |
| Are Right | "Redis plus change feed, decided with latency numbers." | 1 vs. 10 ms · 5 regions · < 5 s |
| Learn & Be Curious | "Learned MCP, agent runtimes and eval; turned it into the platform's rules." | GA Sep 30 · 6 clients · 95% vs. 80% |
| Hire & Develop | ➡️ Microsoft story | — |
| Highest Standards | "Fail closed, void on failure, guard before the platform fix." | 7 → 2 calls · 0 orphans · 4 teams |
| Think Big | "Agents became a new identity type: a 25-item program." | 25 items · 4 scenarios · 101–148 eng-weeks |
| Bias for Action | "MCP Open Beta at Momentum, GA in 4.5 months." | May → Sep 30 · 7 API groups · 6 clients |
| Frugality | "Cheaper storage, scoped parity, no new team." | $1.68M → $1.14M · 13.5 FTE · 2.5 AuthZ eng |
| Earn Trust | "Told Security the exact revocation exposure." | ≤ 1 token lifetime · kill switch immediate · < 5 s |
| Dive Deep | "A delete call would have silently revoked another team's access." | 2 integrations · 0 incidents · TOCTOU fix |
| Have Backbone | "Parity before migrating onto my own platform." | 7 gaps / 3 P0 · 10+ limits in 1 RTT · 4,000 TPS |
| Disagree & Commit | "Leadership skipped Phase 1; I committed the same day." | 2 weeks saved vs. 3–4 weeks plan · 4 tracks · 5 callers |
| Deliver Results | "MCP GA on the public date, plus 5+ agents on one platform." | Sep 30 · 7 groups · 4 teams |
| Earth's Best Employer | ➡️ Microsoft preferred, else: "Took ops out of the config toil loop." | 2,500 requests/yr · 50+ → 1 · 15 → 5 min |
| Success & Scale | "Our envelopes were a phishing vector." | 0.5% → 0.15% · 1.9M customers · 97.3% |

---

## 3. The 20 numbers to memorize
1. **1.9M** customers · **1.1B** signatures FY26 · **47%** via API
2. **40K** IAM customers · **$350M+** IAM ARR · **300M** agreements
3. **51** allowlists → 1 · **13** capabilities · **< 10 µs**
4. **25** work items · **4** scenarios · **101–148** engineer-weeks
5. **123K** Maestro runs per 30 days · **7 → 2** AuthZ calls
6. **40.5M** sharing configs · **$59M** MRR · **13.5** FTE
7. **$1.68M → $1.14M** storage · **0** drift gate
8. **15 → 5 min** config deploy · **2,500** ops requests/yr · **13%** misconfig incidents
9. **5** regions · **1 ms vs. 10 ms**
10. **4,000 TPS** · **758 / 97.3%** · **0.5% → 0.15%** fraud · **100×** limits
11. **May → Sep 30** MCP GA · **7** API groups · **6** AI clients · **50%** faster

**Mnemonic:** *"51-25-123-40-15-5-4000-758-Sep30"* is one key number per project, P1 → P8 in order (P4's is 40.5M configs, P7's has two).
