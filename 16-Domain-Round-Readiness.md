# 16 — Domain Round Readiness (Reverse System Design)

Recruiter's brief:
> *"Discuss a system architecture you have designed or made contributions towards. Be prepared to **defend your choices** and talk about your **learnings**. **CodeSignal whiteboarding** may be used. They will also ask how you **leveraged AI tools in system architecture, design decisions, and technical problem-solving**."*

## 1. Coverage check

| Recruiter ask | 14 Trust Platform | 15 Agent Platform | Where |
|---|---|---|---|
| A system you designed | ✅ | ✅ | 14 §1–7, 15 §1–7 |
| Defend choices | ✅ decision tables + probes | ✅ | 14 §8, §15 · 15 §8, §15 · **§4 below (pushback drills)** |
| Learnings | ✅ | ✅ | 14 §14 · 15 §14 |
| Whiteboard | ⚠️ diagrams exist, but no drawing plan | ⚠️ | **§3 below** |
| AI in **architecture** | ✅ | ✅ | 14 §13 · 15 §13 |
| AI in **design decisions** | ✅ | ✅ | 14 §13 · 15 §13 |
| AI in **technical problem-solving** | ⚠️ thin | ⚠️ thin | **§2 below** |
| Staff-level signals (scope, influence, mechanisms) | partly | partly | **§5 below** |

---

## 2. AI across all three areas: a story for each

Say it as a **loop**: *AI explores and drafts → I decide → verification gates prove it.* Always include **one place AI was wrong**; it shows you weren't using it blindly.

### A. Architecture (exploring options)
- **14:**
  - An agent traced the legacy ServiceProtection code to draft the protector inventory. I verified every gap, which gave the **7-gap parity list (3 P0)** in days, not weeks 📈.
  - I used the model to compare limiter algorithms against our fraud requirements, then built the per-limit choice (token bucket vs. fixed window + multi-limit).
- **15:**
  - Ran 50 vague prompts against draft tool schemas on several models. **The tools the models confused showed me which to merge**, giving the 7 task-shaped groups.

### B. Design decisions (stress-testing)
- **Red-team prompts:**
  - "You are a fraudster with 1,000 stolen IDs" led to the **5-accounts-per-ID cap**, attempt lockout and conditional writes (14).
  - "You are a malicious contract" led to **tool filtering before the prompt** and attachment isolation (15).
- **Back-of-envelope:** the model drafted the Redis memory and ops model; I corrected it with real TPS (4,000 ✅).
- **Option memos:** asked for the strongest case *against* my preferred option (per-region counters, fail open), then answered each point in the design doc.

### C. Technical problem-solving (debugging and root cause) ← the gap, now filled
| Problem | How AI helped | What I verified |
|---|---|---|
| **Double-counting during migration (TOCTOU)** (14) | Gave the agent the send path and the symptom (counts slightly above legacy in shadow). It pointed out the **migration mode was read twice per request**, so a flag flip mid-request could take both paths. | Reproduced it with a test that flips the flag between reads. Fix: read the mode once per request ✅. |
| **Counters reset when account age changed** (14) | AI-generated descriptor test matrix (order × prefix × empty values × `age_bucket` transitions) | The failing case showed `age_bucket` in the counter key. Fix: stable counter on `account_id`; threshold chosen client-side ✅ |
| **Ramp hit a direct enterprise customer** (14) | Used AI to write telemetry queries joining blocked decisions with account segment, to find **every** affected account in minutes | Checked results against the account list; excluded the segment the same day ✅ (EMT-8388) |
| **Silent config fallback in NA1** (P5) | AI drafted queries listing every config change during the window, per namespace | Each change confirmed with its owner |
| **Agent tool-selection failures** (15) | Clustered failed traces by error pattern: wrong tool, bad arguments, missing context | Top clusters became golden-set cases plus tool-description fixes |

### Where AI was wrong (use one)
1. **Redis Cluster cross-slot Lua.** The AI-written script updated several counter keys in one Lua call, which **fails in Redis Cluster** when the keys hash to different slots.
   - Fix: **hash tags** (`rl:{account_id}:...`) so one account's counters share a slot. That also keeps the atomic multi-limit check.
   - Caught by an integration test against a real cluster, not by review.
2. **It proposed globally replicated (CRDT) counters** for exact global quotas. Technically elegant, but it adds cross-region latency and complexity to a 1–10 ms hot path. I rejected it with the numbers.
3. **A query joined decisions on the wrong time window,** which undercounted affected accounts. Caught by checking against a known account.

**Lines to say:**
- "AI made exploring and drafting cheap, so I put the saved time into verification: shadow parity, integration tests, red-team review."
- "I never let AI output reach production without a gate that would catch it being wrong."
- 🔭 "On your team I'd apply the same loop to investigator tooling: AI drafts, a human approves, everything is audited."

---

## 3. CodeSignal whiteboard plan

**Before the interview:**
- Open a CodeSignal or Excalidraw-style canvas and **draw the design once in under 8 minutes**.
- Practice: rectangles, arrows, text, moving groups.

**Rules:**
- **Draw at most 6–8 boxes** at first. Zoom into one area per deep dive and draw it separately to the right.
- **Number the arrows** for the main flow (1, 2, 3…) and narrate them as you go.
- **Write numbers on the board:** 4K TPS, 1–10 ms, 12K peak, 100K Redis ops/s, ~0.8 GB.
- Keep a **"Decisions" box** in a corner and add each trade-off as you defend it.

**Drawing order for 14 (~8 min):**
1. Left: `Callers` (Edge/Envoy, eSign send).
2. Center: `Rate-limit service (stateless)` → `Redis (sharded, {account} hash tag)`.
3. Top: `Control plane: Git YAML + override store → config distributor (<30 s)`.
4. Right: `Trust layer: risk engine + IDV orchestrator → tier store → events`.
5. Bottom: `Decision log → analytics → support + risk signals`.
6. Numbered hot path: 1 send → 2 tier lookup (cached) → 3 batch check → 4 Lua → 5 decision.

**Drawing order for 15 (~8 min):**
1. Left: `Front doors` (external AI clients, Iris, Studio).
2. Center: `MCP server` + `Runtime (LangGraph)`.
3. Center-right: `Policy layer: PEP → PDP, HITL, rate limits, kill switch` → `Docusign APIs`.
4. Below the runtime: `Model gateway`, `Knowledge search`, `Checkpoint store`.
5. Bottom: `Audit → Eval gate`.
6. Numbered tool call: 1 token exchange → 2 filter tools → 3 model proposes → 4 PDP → 5 approval → 6 execute + audit.

---

## 4. Pushback drills ("defend your choices")

Answer format: **acknowledge → criterion → numbers → when you'd change your mind.**

| They push | Your answer |
|---|---|
| "Fail open is a fraud hole." | "It's bounded by outage time, alerted, and layered under risk tiers and IDV. Failing closed would turn a cache blip into a revenue outage on 100–500 ms sends. I'd fail closed for a money-movement flow." |
| "Per-region counters aren't accurate." | "Accounts are region-homed, so overshoot is ≤ limit × regions and only for multi-region senders. Exact global counting adds a cross-region hop to a 1–10 ms path. If exactness mattered, I'd route the counted operation to the home region." |
| "Why Redis, not a database?" | "Sub-ms atomic increments, Lua for an all-or-nothing multi-limit check, TTL matching the window. Counters are reconstructible, so losing them is a bounded risk." |
| "Unmatched → OK hides misconfiguration." | "It hides it from customers, not from us. A zero-match alert fires on every new limit, and everything ships in shadow first." |
| "Fixed windows allow a 2× burst at the edge." | "Yes. The shorter windows (daily, 3-day) cap it. For exact rolling windows I'd sum sub-buckets in Lua for one MGET." |
| "Why not let the LLM decide permissions with a good prompt?" | "Prompts are guidance, not enforcement. Injected text shares the context window. The PDP is deterministic, and the model can't widen its own access." |
| "Human approval kills agent value." | "Only state changes need it. Reads and drafts don't. The value is in drafting; approval is one click on a pre-filled card." |
| "Why LangGraph and not build your own?" | "Checkpointed interrupts for HITL, explicit state, retries per node. Teams ship in weeks. I'd revisit if we needed cross-language runtimes." |
| "Why multiple model providers? That's complexity." | "Failover, cost routing and version pinning. One gateway hides it from agent teams. The eval gate makes switching safe." |
| "What would you do differently?" | 14: segment-aware ramps and block-to-unblock metrics as platform features from day one. 15: the eval gate before beta, not after. |

---

## 5. Staff-level signals to weave in (don't just describe boxes)

| Signal | Say |
|---|---|
| **Scope** | "A platform for every MSF service, not one feature." / "One agent platform for 4 teams and 6 AI clients." |
| **Business link** | Fraud rate −70% ✅, support load, revenue protected by failing open; 50% faster NDA summarization ✅ |
| **Mechanisms over heroics** | Shadow-first config, parity gates, eval gates, readiness checklist, segment ramps |
| **Influence** | "The hardest consumer's parity requirements drove the roadmap; I aligned eSign, Security and Support on the 7-gap list." |
| **Safe change at scale** | Legacy → Shadow → MsfOnly with one-flag rollback; Beta → GA |
| **Owning failure** | The ramp hit a direct enterprise customer; fixed the segment the same day and made segment exclusion standard |
| **Knowing when not to** | Deferred real-time revocation and out-of-band approval; rejected CRDT counters; no fine-tuning |

---

## 6. Night-before checklist
- [ ] Say the 60-second pitch for 14 and 15 out loud, timed.
- [ ] Draw 14 on a blank canvas in under 8 min, with numbers on the board.
- [ ] Do the back-of-envelope from memory: 4K → 12K TPS, × 8 counters ≈ 100K ops/s, ~0.8 GB, ~100 GB/day → sampled.
- [ ] One AI story each for architecture, decisions and problem-solving, plus one "AI was wrong" (Redis cross-slot).
- [ ] Practice 3 pushbacks from §4 out loud.
- [ ] Have 3 learnings ready to close with.
- [ ] Prepare 2 questions to ask (e.g., "What's the hardest reliability problem on the investigator platforms right now?").
