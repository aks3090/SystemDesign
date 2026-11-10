# 50-Minute Lead / Staff System Design Playbook: Sensor Fleet Management

Companion to [sensor-fleet-management-system-design.md](sensor-fleet-management-system-design.md). That doc is everything you *know*. This doc is what you *say* in 50 minutes and in what order.

Target level: **Lead (e.g. Salesforce LMTS) to Staff**. Lead is judged mostly on a solid, well-driven design with clear trade-offs. Staff adds: naming the real problem early, designing for failure and blast radius, and thinking about how the system is operated and evolved.

---

## The One Thing to Remember

Interviewers expect you to start with scale. **Start by saying scale is not the hard part; safety is.** Everything you draw after that supports this point.

> "1.2M devices is about 4K check-ins per second and a few GB of state. That's easy. The hard part is that we're shipping kernel-level code to 1.2M machines. One bad update can brick the fleet (the CrowdStrike July 2024 incident is the example), and whoever controls this system can run code everywhere. So I'll optimize for safety first, then security, then speed."

---

## Time Plan (50 min)

| Min | Phase | Goal | Output on the board |
|---|---|---|---|
| 0–5 | Clarify | Scope + key assumptions | 5–6 bullet requirements |
| 5–8 | Estimate + framing | Show scale is easy, safety is hard | 4 numbers + priority order |
| 8–11 | Principles | 4 rules the whole design follows | 4 short lines |
| 11–20 | High-level design | Boxes + one end-to-end flow | One diagram |
| 20–38 | Deep dives (3–4) | Depth where it matters | Rollout state machine, cohorts/gates, on-device safety, check-in |
| 38–45 | Failures + security | Walk the scary cases | Failure table (spoken), trust boundaries |
| 45–50 | Trade-offs, operations, wrap | Judgment and ownership | 3 trade-offs + "how we run it" |

**Clock checkpoints: 11, 20, 38, 45.**
- Behind at 20: drop deep dive C (check-in) and keep A and B.
- Behind at 38: give failures as 4 one-liners and go straight to trade-offs.
- Ahead at 38: ask the interviewer, "Is there an area you'd like more depth on?"

Leave the last 5 minutes for wrap-up no matter what. Ending mid-sentence on a deep dive looks worse than covering one fewer topic.

---

## Phase 1 (0–5 min): Clarify

Ask 5–6 questions, then **state assumptions and move on**.

1. "Is this a vendor platform with many customer tenants, or one company's fleet?" (Assume multi-tenant.)
2. "What changes do we ship: agent binaries only, or also detection content and policy?" (Assume all three. Content ships often and is the riskiest.)
3. "Do sensors have kernel components?" (Assume yes on Windows, eBPF/system extensions on Linux/macOS.)
4. "Any restricted or air-gapped networks?" (Assume some.)
5. "How fast must an urgent detection update reach the fleet?" (Assume hours, not seconds.)
6. "Do customers control their own update timing (maintenance windows, pinning)?" (Assume yes.)

Write on the board:

```
Functional: enroll/inventory · upgrade agent · distribute policy + content
            · progressive rollout + gates + halt/rollback · health · rollout monitoring
Non-func:   devices stay protected if control plane is down
            · kill switch < 60s · signed everything · offline devices converge later
Out:        detection engine itself, EDR event data lake
```

## Phase 2 (5–8 min): Numbers and Framing

| Number | Value |
|---|---|
| Check-ins | 1.2M / 5 min ≈ **4K rps**, design 40K for reconnect storms |
| Concurrent connections (push hint) | ~900K, ~40 gateway nodes |
| Device state | ~4 KB × 1.2M ≈ **5 GB** |
| Artifact egress | 80 MB × 900K in a day ≈ **~7 Gbps**, so use a CDN |

Then say the priority order out loud: **safety > security > protection availability > rollout speed > cost.**

## Phase 3 (8–11 min): Four Principles

1. **Pull-based desired state.** The server says *what* a device should run. The device pulls, reconciles and reports. Push is only a "check in now" hint.
2. **Every change is a rollout.** Binary, content and policy all go through rings and gates. Urgent means a shorter schedule, never skipping the canary.
3. **The device protects itself.** It verifies signatures, keeps the last good version, and rolls back on its own.
4. **Never break the recovery path.** The updater is separate from the sensor and stays alive when the sensor crashes.

## Phase 4 (11–20 min): High-Level Design

Draw this. Keep it to about 12 boxes.

```
 [Device: Updater | Sensor | Watchdog | Validator | A/B slots]
        |  mTLS check-in (pull)       ^ "check in now" hint
        v                             |
 [Gateway] --> [Desired State Svc] --> [Device State DB]      (per cell, ~200K devices)
        |
        v
 [Kafka] --> [Flink] --> [ClickHouse] --> [Rollout Orchestrator] --> [Config DB: rollouts]
                              |                   ^
                              v                   |
                        [Dashboards]        [Admin API]
                                                  |
                       [Policy Svc / Release Svc] --> [Signing Svc (HSM, M-of-N)] --> [CDN / Relay]
                                                                                          ^
 Device downloads signed artifacts from CDN ----------------------------------------------+
```

Walk through **one flow end to end** (about 3 min):
"An operator creates a rollout. The orchestrator opens 0.1% of each OS group. The device checks in, the desired state service sees its hash bucket is in range and returns the new version. The device downloads from the CDN, verifies the signature, installs to slot B, and runs a probation check. It reports health. Flink compares upgraded devices to non-upgraded ones in the same group. The orchestrator moves to the next ring or halts."

Then 1 minute on **policy flow** to show it's covered: "Policy goes through the same engine: author → lint → compile per agent schema → sign → per-tenant rings."

## Phase 5 (20–38 min): Deep Dives

Offer a choice: **"I'd like to go deep on rollout safety, since that's where the risk is. Would you rather I cover something else?"** Then do these in order.

### A. Rollout engine (must do, ~6 min)

- Rings: lab → 0.1% → 1% → 10% → 50% → 100%.
- Durable state machine (Temporal-style): Draft → Approved → StageRunning → Baking → GateEval → next / Paused / RollingBack → Completed. Draw it.
- Idempotent steps, single writer per rollout with fencing.
- Global guardrails enforced in code: velocity budget (max % of fleet changing per hour), one rollout per component per group, freeze windows, kill switch.
- Halt vs rollback vs fix-forward: halt stops new devices; rollback is a new rollout to the previous version; fix-forward is most common for content.

### B. Cohorts and health gates (must do, ~5 min)

- Cohorts: `hash(device_id, rollout_salt) mod 10000`, **stratified per OS/kernel group**. Widening only adds buckets, so devices never flip. Rare kernels get a minimum count so they appear early.
- Gates: **treatment vs control** in the same group and time window. This cancels noise like Patch Tuesday.
- Advance needs **bake time AND minimum sample size**. At 0.1% you only catch catastrophic bugs. At 1% (~12K) you can detect about a 2x crash regression in a day.
- **Fail closed:** stale telemetry means pause, never pass.
- **Silence after an update is a failure signal**, because a crashed device can't report.
- Monitoring: funnel (notified → downloaded → installed → healthy), per-group gate status, ETA.

### C. On-device safety (must do, ~4 min)

- Split agent: updater / sensor engine / watchdog / content validator.
- A/B slots, probation window, local auto-rollback, local quarantine of a bad version.
- **Boot-loop counter**: 2 failed boots means load in safe mode with the last good version, updater still alive. "A bricked device becomes a degraded but reachable one."
- **Validate content in a user-mode sandbox before kernel code reads it.** This is the direct lesson from CrowdStrike.

### D. Check-in and propagation (if time, ~3 min)

- Generation numbers, `304` when nothing changed.
- Desired state computed on read from a small in-memory snapshot. This avoids 1.2M writes per stage.
- Jitter, `429 Retry-After`, exponential backoff for reconnect storms.
- Kill switch: push hint for connected devices (< 60 s), poll interval for the rest.

## Phase 6 (38–45 min): Failures and Security

**Failures (~4 min), one line each:**

| Scenario | Answer |
|---|---|
| Device offline 3 weeks | Converges to *current* desired state on return. No replay. Keeps old policy meanwhile. |
| Control plane down | Devices keep enforcing last good policy. Nothing new rolls out. |
| Telemetry pipeline down | Gates can't pass, rollouts pause. |
| Bad content crashes the kernel | Sandbox validation, rings, boot-loop safe mode, "devices went silent" tripwire halts in minutes. |
| Bad policy blocks business apps | Linting, impact preview, audit-mode dry run, per-tenant rings, block-rate gate. |
| Region/cell down | Cells isolate the blast radius. Fail over to paired region. |
| Reconnect storm | Jitter, admission control, 304 cache. |
| Orchestrator crash | Durable workflow, idempotent steps, fencing tokens. |

**Security (~3 min):**
- **Signing is separate from distribution.** HSM, M-of-N approvals, TUF metadata, pinned root on device. "If the control plane is hacked, it still can't sign code."
- mTLS, short-lived TPM-bound device certs, anti-rollback floor, 2-person rule for platform-wide actions, append-only audit.
- Telemetry integrity: only authenticated devices report, so fake devices can't make a bad rollout look healthy.

## Phase 7 (45–50 min): Trade-offs, Operations, Wrap

**Three trade-offs (~2 min):**
1. **Pull + hint vs pure push:** pull is correct through NAT and offline. Push alone isn't idempotent.
2. **Ringed content vs instant content:** costs 1–3 hours on urgent detections, but prevents the CrowdStrike failure.
3. **Cells vs one global cluster:** more ops work, but limits damage from our own bad deploys.

**How we run it (~2 min, strong Staff signal):**
- The control plane itself deploys cell by cell, canary cell first.
- The updater gets the most conservative rollout of all.
- Game days: kill a cell, inject bad content in the lab, cut the CDN.
- Every incident produces a new automated gate or lint rule, not just a process note.
- Phased delivery: foundation (signed updates + local rollback) → automated gates → policy/content → cells/multi-region.

**Close (~1 min):** "If I had more time I'd go into the exact gate statistics, the policy compiler for mixed agent versions, and moving more sensor logic out of the kernel (eBPF, system extensions), which is the best long-term safety lever."

---

## Lead vs Staff vs Senior

| Senior | Lead (LMTS) | Staff |
|---|---|---|
| Starts with Kafka, DBs, sharding | Clear requirements, clean design, explains each component's job | Says scale is easy and names safety as the real problem |
| "Canary then full rollout" | Rings with health checks and rollback | Stratified cohorts, control groups, sample-size-aware gates, fail closed |
| Server-side rollback | Device keeps old version, can roll back | Device rolls back by itself, boot-loop safe mode, recovery path protected |
| "Sign the binaries" | Signed artifacts, mTLS, RBAC | Signing separate from distribution, M-of-N, survives control-plane compromise |
| Rollouts for binaries | Rollouts for binaries and policy | Rollouts for **every** change, including content |
| Answers questions | Drives the session, manages time | Also covers how the platform is operated, evolved and staffed |

For a Lead interview, the Lead column is the bar, and a few Staff points (safety framing, silence-as-failure, sandboxed content validation) set you apart.

---

## Likely Follow-Ups With Short Answers

**"A 0-day is spreading. You need new detection content everywhere in 30 minutes. What do you do?"**
Compressed ring plan: lab 5 min, 1% 10 min, 10% 10 min, then the rest, with tripwire-only gates (crashes, silence). Content was already validated in the sandbox. I'd push back on skipping rings entirely, and say the trade-off out loud: 30 minutes of slightly slower coverage vs the risk of bricking the fleet. It's a business decision, and I'd want it pre-agreed, not made during the incident.

**"How do you pick the canary devices?"**
Hash buckets stratified by OS/version/kernel/arch, with a minimum count per group, opted-in customers first, and a new salt per rollout so the same customers aren't always first.

**"How do you know an upgrade is bad if the device crashed and can't report?"**
Silence after an update. Compare the "went offline after change" rate to the control group. A spike halts the rollout.

**"Device is fully bricked by a kernel bug. Now what?"**
Boot-loop counter makes the next boot load safe mode with the last good version. The updater is still running, so we can deliver a fix. Without that, it's manual recovery, which is exactly what we're designing against.

**"What if someone hacks your admin console?"**
They can create a rollout but can't sign code: signing needs HSM + M-of-N + provenance. Rings and the velocity budget still apply, and they're enforced in code. Policy changes that disable protection need extra approval. The audit log is tamper-evident.

**"How do you handle 10 versions in the field at once?"**
Compatibility matrix in the release metadata. Support direct upgrade from N-3. The policy compiler emits a bundle per schema version, and the server serves the matching one.

**"Why not store desired state per device in a DB?"**
That works, but opening a stage would mean 1.2M writes. Computing on read from a small snapshot is deterministic and cheap. The cost is debuggability, so we build an "explain desired state for device X" tool.

**"How would you test the platform itself?"**
Synthetic devices per OS group in every cell running full rollouts continuously, a lab farm, game days, and the control plane rolled out cell by cell.

**"What's the data model?"**
Device, DeviceState (reported vs desired, generation), Release, Artifact, PolicyVersion (immutable), Rollout, RolloutStage, DeviceEvent (idempotent). Rollouts in a strongly consistent DB, device state sharded per cell.

**"How do you scale to 5M devices?"**
Add cells. Each cell is ~200K devices with its own gateways, DB and Kafka. The global plane is low-QPS and doesn't grow much.

---

## Pitfalls to Avoid

- Spending more than 5 minutes on clarifying or 3 on estimates.
- Drawing 25 boxes before explaining one flow.
- Going deep on database choice early. Mention Postgres/Kafka/ClickHouse in one line and move on.
- Saying "canary" without saying how you pick it and how you decide it passed.
- Forgetting content and policy rollouts. Many candidates only cover binaries.
- Forgetting that the device must recover **without** the server.
- Not stating trade-offs. Every choice should come with "the cost is…".
- Running out of time before failures and security. Use the checkpoints.
- Not checking in with the interviewer. Ask "does this match what you want to explore?" at 11 and 20 minutes.

---

## If the Interviewer Steers You

| They push toward | Pivot |
|---|---|
| Scale / storage | Cells of 200K, stateless gateways, Postgres per cell, compute-on-read. Then bring it back: "the bottleneck that matters is risk, not throughput." |
| Security | Phase 6 security, plus the threat list: control-plane compromise, supply chain, rollback attack, insider, local tampering. |
| Monitoring | Funnel, per-group gates, ETA, alerts on stuck rollouts, fleet drift view. |
| Policy | Immutable versions, per-schema compile, linting for dangerous rules, impact preview, audit-mode dry run, per-tenant rings. |
| Product / org | Tenant update channels (early/default/late/pinned), maintenance windows, critical groups go last, freeze windows, who approves what. |
