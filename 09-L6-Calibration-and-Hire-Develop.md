# 09 — L6+ Calibration (what's in, what's out, and why)

## Level reference
- **Amazon L6 = SDE III / Senior SDE.** L7 = Principal. Amazon has no "Staff" title.
- **Coinbase Staff (IC6) ≈ strong Amazon L6 to low L7.** Aim every answer at **strong L6**.
- **L6 behavioral bar:**
  - Owns a large, ambiguous problem across your team **and** 2+ partner teams.
  - Explicit options and trade-offs, backed by data.
  - Influences decisions without authority.
  - Leaves a **mechanism** behind (platform, guard, checklist, gate), not heroics.
  - Shows a **customer or business outcome with a number**.
  - Is self-critical ("what I'd do differently").

## Verdict per project (all 8 owned by you)

| Project | L6+? | Why it clears the bar | Have ready |
|---|---|---|---|
| **P1 1P Access Mgmt** | ✅ | Horizontal platform across monolith and MSF; build-vs-buy POC; governance; disagree-and-commit | Adoption numbers (allowlists retired, clients onboarded) |
| **P2 AuthZ / NHI** | ✅ L6/L7 | Program owner: 25 work items, 4 scenarios, P0–P4, 101–148 engineer-weeks; new identity type; standards calls; fail-closed enforcement | eSign enforcement live; GDS Agent identity on track |
| **P3 Maestro / Workspaces** | ✅ | 4 teams; new impersonation primitive; latent cross-integration bug; mid-flight model change | Volume on the path; next app onboarded |
| **P4 Unified Sharing** | ✅ (L6/L7 scope) | $59M MRR, 40.5M configs, 13.5-FTE program, cohort migration with parity gates | Gate status reached; mismatch rate |
| **P5 Config / Secrets** | ✅ | Org-wide toil (2,500+ requests/yr), safe shadow migration, the secrets boundary | Keys migrated; ramp time before/after |
| **P6 Global Directory** | ✅ | Multi-region consistency design; data-driven cache choice; phased migration | Read p99; propagation lag |
| **P7 Rate Limit Platform + IDV** | ✅ (L6/L7 scope) | Designed a multi-tenant platform for all MSF services; parity-driven roadmap; IDV shipped 100% in Prod with real outcomes | Services onboarded; fraud-rate and ticket numbers |
| **P8 Agent Platform + MCP** | ✅ L6/L7 | MCP Server beta → GA in ~4.5 months on 6 AI clients and 7 API groups; shared agent platform (LangGraph, gateway, eval); governance bar | 5+ agents across 4 teams; 95% vs. 80% eval gate |

## Principles where DocuSign doesn't clear L6 → use Microsoft

| Principle | Decision |
|---|---|
| **Hire and Develop the Best** | ➡️ **Microsoft.** None of the 8 projects has a story about growing a named person, giving hard feedback, or a hiring decision. Partner enablement (guides, coaching another team) reads as L5. |
| **Strive to be Earth's Best Employer** | ➡️ **Prefer Microsoft** if you have a people or team-health story. The DocuSign fallback (P5 toil removal) is L6 on scope but is a weaker fit for this principle. |

## Removed entirely (never use)
- **DSS flag cleanup.** Routine expired-flag removal, not a story.
- Anything outside the 8 projects (alert tuning, CLM task delegation, KTLO plan, AI-skill tweaks, handoff notes). These came from browsing your history, not from your stories.
- **Weak LP bullets:** "asked the config team for SLOs," "coached Signing on Slack," "wrote an onboarding guide" (as a people story). None are L6.

## Five upgrades that apply to every answer
1. **Open with scope:** teams, engineers, customers, $ at stake.
2. **"I" for decisions, "we" for execution.**
3. **Name the tension:** who disagreed, what was ambiguous, what could break.
4. **End with a mechanism plus a number.**
5. **One "what I'd do differently."**

## Microsoft story slots you need to prepare
1. **Hire and Develop the Best:** a named engineer you grew (scope handed off, weekly coaching on trade-offs, hard feedback, outcome such as a promotion or leading a design), plus one hiring or debrief beat.
2. **Earth's Best Employer (optional but stronger):** team health, on-call culture, inclusion, or burnout prevention, with a measurable change.
3. **Optional spare:** a second Ownership or Deliver Results story, so you don't reuse P3 or P7 three times within one loop.
