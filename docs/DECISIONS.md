# ai-flow — Architecture Decision Records (D0, revision r7)

Format per entry: **Context → Decision → Alternatives → Consequences → Status / Confidence**.

- Status values: `Proposed` (awaiting the targeted D0 r7 review), `Accepted`, `Superseded`.
  - D0 approval: the seventh independent review (Codex, D0 r7) returned **APPROVE** with zero required changes, so every ADR previously `Proposed` is now `Accepted` as of D0 r7. Decision text, alternatives, consequences and confidence are unchanged; `Medium`/`Low` confidence entries and items marked conditional on D0.5 remain subject to D0.5 evidence.
  - CHECKPOINT: approval commit `b9e2b451f84419d1e9ff95443d3f86f770f14744` (`docs: approve D0 r7 architecture`).
- Confidence values:
  - `High`: well understood.
  - `Medium`: reasonable; revisit with data or D0.5 evidence.
  - `Low`: an explicit bet that the reviewer should challenge.
- `r2` marks entries revised after the first independent review (Codex, verdict CHANGES_REQUESTED).
- `r3` marks entries corrected after the second independent review (Codex, verdict CHANGES_REQUESTED, no blockers; architecture otherwise confirmed sound).
- `r4` marks entries corrected after the third independent review (Codex, verdict CHANGES_REQUESTED, two remaining recovery-safety gaps).
- `r5` marks entries corrected after the fourth independent review (Codex, verdict CHANGES_REQUESTED, two remaining material defects).
- `r6` marks entries corrected after the fifth independent review (Codex, verdict CHANGES_REQUESTED, one remaining MAJOR finding; the r5 journal-integrity and cleanup-safety corrections were confirmed resolved).
- `r7` marks entries corrected after the sixth independent review (Codex, verdict CHANGES_REQUESTED, two material defects in the r6 recovery procedure).
- `[OWNER]` marks decisions made explicitly by the project owner.
- Section references (§) point to [`ARCHITECTURE.md`](./ARCHITECTURE.md).

| ADR | Title | Status | Confidence |
|---|---|---|---|
| 001 | Local-first CLI orchestration, not a server | Accepted | High |
| 002 | The orchestrator owns the workflow; agents execute steps | Accepted | High |
| 003 | Capability-oriented provider adapters; Codex SDK-first, conditional on D0.5 (r2) | Accepted | Medium |
| 004 | Journal authoritative, state derived, Git reconciled (r2, r3, r4, r5) | Accepted | Medium |
| 005 | Global vs project config; runtime state outside the project (r2) | Accepted | High |
| 006 | Structured review with evidence, structural lineage and test-decided ACs (r2) | Accepted | High |
| 007 | Implementer/reviewer independence, explicit fallback only (r2) | Accepted | High |
| 008 | Context bundles: minimal, ordered, hashed, incremental re-review (r2) | Accepted | Medium |
| 009 | Git isolation: sibling worktree, orchestrator-only checkpoints, hooks bypassed (r2) | Accepted | Medium |
| 010 | Spec Kit as an optional integration, not a core dependency | Accepted | High |
| 011 | Recovery invariant: at most one accepted result per attempt; reconciliation (r2, r3) | Accepted | Medium |
| 012 | ~~Declarative YAML workflows~~ | **Superseded by ADR-024** | — |
| 013 | Staged deterministic validation gates AI review (r2) | Accepted | High |
| 014 | TypeScript on Node.js 22, no orchestration framework (r2) | Accepted | Medium |
| 015 | Routing frozen per epoch; human-approved reroute creates a new epoch (r2, r3) | Accepted | High |
| 016 | Billing protection fails closed with non-secret attestation (r2) [OWNER] | Accepted | High |
| 017 | Versioning: typed workflow version + prompt id@semver + hashes, snapshots (r2) | Accepted | Medium |
| 018 | Mandatory D0.5 feasibility spikes before D1 (r3) | Accepted | High |
| 019 | Suspension overlays; only DONE/FAILED/CANCELLED are terminal (r3, r6) | Accepted | High |
| 020 | Fenced lease lock; takeover only on verified owner death; damaged journal tail fails closed; staged successor publication (r3, r4, r5, r6, r7) | Accepted | Medium |
| 021 | Runtime-state and worktree locations; repo-instance identity (r6: recovery staging area) | Accepted | Medium |
| 022 | Side-effect classes, effect locations and retry safety; no destructive cleanup without safe primitives (r3, r4, r5, r6) | Accepted | High |
| 023 | Security enforcement levels | Accepted | High |
| 024 | One typed code workflow + pure reducer in D1; DSL deferred | Accepted | High |
| 025 | Model & Effort Router: separate decisions, abstract profiles, per-provider mapping (r3) | Accepted | Medium |
| 026 | Risk floors and bounded, per-lineage escalation (r3) | Accepted | Medium |
| 027 | Small-task fast path: opt-in, no self-certification (r3) [OWNER] | Accepted | Medium |
| 028 | D1 scope: provider-neutral local orchestrator for Git-based projects | Accepted | High |

---

## D0 review resolution log

Traceability for the reviews. "Accepted" means implemented as proposed. "Refined" means accepted with a stated precision or limit. "Disagreed" means an alternative was proposed (none remain unresolved: the one disagreement was settled by the owner).

### Seventh review (r7): APPROVE

The seventh independent review (Codex, targeted re-review of D0 r7) returned verdict **APPROVE**.

| Check | Result |
|---|---|
| Two-phase acquisition (R6-M1) | PASS |
| `B = 0` recovery refusal (R6-M2) | PASS |
| Initial-run crash safety | PASS |
| Material findings | None |
| Required changes | None |
| Checkpoint readiness | READY |

Scope: specification-level review; no executable implementation exists or was tested. D0 r7 is **APPROVED**; the next phase is **D0.5**. CHECKPOINT: approval commit `b9e2b451f84419d1e9ff95443d3f86f770f14744` (`docs: approve D0 r7 architecture`).

### Sixth review findings (r7)

The sixth independent review (verdict CHANGES_REQUESTED) found exactly **two** material defects in the r6 recovery procedure. r7 is a **minimal correction** that follows explicit owner decisions [OWNER]: fix the ordering without a transaction manager; separate physical lock ownership from the journal append; refuse successor recovery when `B = 0`; add no special empty-prefix successor format; keep every previously approved D0 safety decision. Only ADR-020 changes. Staging and atomic publication, idempotent recovery, bound human continuation, fail-closed damaged-tail handling, the Windows destructive-cleanup prohibition, quarantine-first `cancel --cleanup`, and the provider, billing, router and fast-path decisions are unchanged. IDs: `R6-M` = material defect.

| Item | Finding | Resolution | Where |
|---|---|---|---|
| R6-M1 | Recovery P3 ran the ordinary §13.5 acquisition, which appends `LEASE_ACQUIRED` as part of acquiring. The lease record was therefore appended before P4 had installed the verified predecessor prefix (into a journal that did not yet exist). | **Accepted** [OWNER]. Acquisition is split into two explicit operations. **`acquire_lock_only`** (acquire steps 1–2 / takeover steps 1–5): exclusive ownership and fence, prior-owner death or explicit break-lock validated; writes only lock files; no `events.jsonl` append, no `state.json`, no Git. **`append_lease_acquired`** (acquire step 4 / takeover step 7 / recovery P5): only after every required check has passed under the held lock; re-checks ownership and the expected EOF; appends and fsyncs exactly one `LEASE_ACQUIRED` with the held fence. Ordinary commands: read-only readiness check → lock only → tail classification + readiness re-check under the lock → `LEASE_ACQUIRED` → anything else; no Git mutation before every required check passed and every required event is durable. Recovery: P3 lock only; P4 installs or verifies the prefix and verifies descriptor, predecessor hash, seal state and successor initialization state under the lock; P5 appends `LEASE_ACQUIRED`. A failure under the lock appends nothing (lock expired and kept, marker deleted). A crash between the phases leaves an unjournaled lock fence; the retry is a new acquisition with fence max(journal, lock) + 1 and exactly one record, so no fence appears in two lease records. | §13.5, §12.1.1, §12.1.2, §13.3, §13.4, §13.6, ADR-020, D1_PLAN A2 |
| R6-M2 | With `B = 0` (no complete valid predecessor record) the successor journal would start with its own `LEASE_ACQUIRED { mode: recovered, run_id: successor }`. Its first record names its own id while `recovery.json` exists, which the r6 classification rule treats as `RECOVERY_INCONSISTENT`, so the successor could never pass the readiness gate. | **Accepted** [OWNER]. `recover --journal` **refuses** `B = 0` with `RECOVERY_NO_VALID_PREFIX`, decided read-only at P0 before any side effect: no staging directory, descriptor, successor journal, predecessor seal, publication marker or lock; the predecessor journal stays byte-identical. The diagnostic explains that no complete valid record exists, so there is no prefix to carry over. Supported options: restore from an independently verified intact copy, or abandon (`recover --abandon`). No special empty-prefix successor format. The structural distinction is unchanged and now total: a successor always has a non-empty verified prefix, so its first record is the predecessor's; a descriptor with `prefix_end: 0` is schema-invalid. | §12.1.1, §12.1.2, §13.6, ADR-020, D1_PLAN A2 |

### Fifth review findings (r6)

The fifth independent review (verdict CHANGES_REQUESTED) confirmed that the r5 journal-integrity and cleanup-safety defects are resolved. It found exactly **one** remaining MAJOR issue: an incomplete recovery successor could become available to ordinary resume before `JOURNAL_RECOVERED` and the mandatory human-continuation suspension were durable. r6 is a **strictly scoped correction**. Only ADR-019 (one bullet), ADR-020, ADR-021 (one bullet) and ADR-022 (one bullet) change. Provider architecture, billing protection, the Model & Effort Router, escalation, the fast path, independent review and the D0.5/D1A/D1B boundaries are unchanged. No transaction manager and no workflow state are introduced. IDs: `R5-M` = MAJOR, `R5-m` = MINOR/stale reference.

| Item | Finding | Resolution | Where |
|---|---|---|---|
| R5-M1 | r5 created the successor in the run namespace and appended `LEASE_ACQUIRED` and `JOURNAL_RECOVERED` one by one. A crash in between left a successor with a clean prefix plus `LEASE_ACQUIRED`: indistinguishable from an ordinary run, resumable and Git-reconcilable without human continuation. Only `JOURNAL_RECOVERED` could identify it for a rerun. | **Accepted.** **Staged initialization:** the successor is built only under `runs/<repo>/.recovering/<successor-id>/`. The id is derived deterministically from the predecessor run id, the predecessor journal sha256 and the recovery generation. The immutable **recovery descriptor** (`recovery.json`: predecessor id, journal size + sha256, `B`, prefix hash, tail, successor id, generation) is published exclusively first. The steps then run in fixed order: staging lease (r7: lock ownership only, see R6-M1) → verified prefix → `LEASE_ACQUIRED` → `JOURNAL_RECOVERED` → suspension `WAITING_HUMAN: JOURNAL_RECOVERED` (bound to the predecessor hash + successor id) → predecessor `run.sealed` (exclusive) → publication marker. **Atomic visibility boundary:** one same-volume directory rename into the run namespace. **Structural classification:** a normal run's first record names its own `run_id` (generation 0); a successor's first record is the predecessor's, so losing the descriptor or marker cannot make a successor look ordinary. **Readiness gate:** every mutating entry point checks it read-only before any side effect and again under the lease; failures → `RECOVERY_PREPARING` / `RECOVERY_INCONSISTENT`, nothing written. **Idempotent reruns** verify and finish the same successor, never duplicate it, never re-append recovery records, never write the predecessor journal and never apply `continue`. **Human continuation:** `decide continue` must quote the predecessor journal hash and successor id. Inconsistencies are never repaired automatically; an unpublished, unsealed staging successor can be set aside by rename. `status`/`doctor` stay read-only. | §12.1.1, §12.1.2, §12.2, §6.1, §13.1, §13.3, §13.4, §13.5, §13.6, ADR-019, ADR-020, ADR-021, D1_PLAN A2, SP-W9 |
| R5-m1 | D1_PLAN still named "D0 r3 accepted" as the D0.5 start gate | **Accepted.** The gate is "D0 accepted at its latest revision (currently r6) after the targeted re-review". | D1_PLAN §1 |
| R5-m2 | The §6.1 state diagram described `ACTIVE → FAILED` for a "corrupt journal beyond repair", which contradicts r5 (no repair; journal damage is `RECOVERY_REQUIRED`) | **Accepted.** `FAILED` is reached only by a broken workflow invariant. A non-`clean` journal produces no transition and is displayed as `RECOVERY_REQUIRED`. | §6.1, ADR-019 |
| R5-m3 | `cancel --cleanup` was described as "removes the worktree", which is a recursive delete and conflicts with the r5 cleanup restriction | **Accepted.** `--cleanup` quarantines the worktree with the identity-verified rename of §13.2.1 (no replacement). It never deletes, on Windows in D1 or on any other platform. An unverifiable worktree is left in place (`CLEANUP_SKIPPED_UNVERIFIED`). | §13.6, ADR-022, D1_PLAN A9 |

### Fourth review findings (r5)

The fourth independent review (verdict CHANGES_REQUESTED) found exactly two remaining material defects, both in r4 mechanisms. r5 is a **minimal safety correction** that follows explicit owner decisions [OWNER]:

1. no new authoritative durable acknowledgement watermark or commit protocol in D1;
2. an ambiguous damaged journal tail fails closed rather than risk losing an acknowledged event;
3. path-based automatic destructive cleanup of ignored files is disabled on Windows in D1;
4. fail-closed suspension, or explicitly configured whole-worktree quarantine/recreation, is used instead;
5. the same cleanup restriction applies to any platform without proven safe deletion primitives.

Only ADR-004 (two bullets), ADR-020 and ADR-022 change. Provider architecture, billing protection, the Model & Effort Router, escalation policies, the fast path, independent review and the D0.5/D1A/D1B boundaries are unchanged. IDs: `R4-M` = MAJOR from the fourth review.

| Item | Finding | Resolution | Where |
|---|---|---|---|
| R4-M1 | r4 treated `state.json.last_event_seq` as the evidence that a damaged final record was acknowledged. Because `state.json` is a disposable cache, a missing or stale `state.json` let r4 truncate an acknowledged record. | **Accepted.** Automatic damaged-tail truncation is **withdrawn**; the journal is append-only without exception. A damaged final record with no positive acknowledgement evidence → `JOURNAL_TAIL_AMBIGUOUS`; with positive evidence (valid `state.json` beyond it, or no lock) → `STATE_CORRUPT`; non-final corruption stays `STATE_CORRUPT`. All three: no truncation, no append, no seq reuse, journal byte-identical. `state.json` can only make the outcome stricter, never authorise a mutation. Interrupted append vs a later-damaged acknowledged record is stated as indistinguishable in D1 → suspend. Read-only diagnostic (`status`, `doctor --journal`). Human recovery: restore from an independent copy, a successor run built from the hash-verified valid prefix (old run byte-identical and sealed; seq `last_valid_seq + 1` reserved; `WAITING_HUMAN: JOURNAL_RECOVERED` before any Git mutation), or abandon. Git reconciliation recovers workspace effects only; lost decisions, routing, suspensions and attestations are not reconstructed. `JOURNAL_TAIL_UNREPAIRABLE` and the forensic tail files are removed. | §12.1, §12.1.1, §12.2, §13.1, §13.4, §13.5, §13.6, §6.1, ADR-004, ADR-020, D1_PLAN A2, SP-W9 |
| R4-M2 | r4's cleanup checked paths with no-follow `lstat`/identity/realpath and then deleted by path. On Windows (and with Node 22 path APIs anywhere) an ancestor could be swapped for a junction/symlink between the check and the delete. The post-deletion ancestor re-check only detected an escape after the external data was gone. | **Accepted.** On Windows in D1, `cleanup: delete` **never** executes path-based deletion; the same holds on any platform unless SP-W8 empirically proves handle-relative, no-follow, race-resistant deletion usable without a native helper (Node 22 path APIs do not qualify; no native helper in D1). Otherwise: `IGNORED_STATE_UNSAFE` (`NO_SAFE_DELETE_PRIMITIVES`), or whole-worktree quarantine/recreation if explicitly configured (`on_cleanup_unavailable: recreate_worktree`). No fallback to deletion. Post-deletion detection is not a safety argument anywhere. Quarantine: identity verified before the rename, after it and before the replacement is created; any failure → suspend, no replacement, nothing deleted; old content retained. Config validation, overlap rejection and the protected/stateful/secret safeguards are kept on every platform. | §13.2, §13.2.1, §13.1, §13.4, ADR-022, D1_PLAN A3/A9, SP-W8 |

### Third review findings (r4)

(R3-M1's truncation and R3-M2's path-based deletion are superseded by R4-M1 / R4-M2 above. The rest of both resolutions stands.)

The third independent review (verdict CHANGES_REQUESTED) found exactly two remaining material recovery-safety gaps. r4 is a **narrowly scoped correction**: no redesign, and no previously resolved finding is reopened. Only ADR-004 (one bullet), ADR-020 and ADR-022 change. IDs: `R3-M` = MAJOR from the third review.

| Item | Finding | Resolution | Where |
|---|---|---|---|
| R3-M1 | A torn journal tail was ignored logically but not physically repaired before the next append | **Accepted.** Every record carries a checksum (`chk`). Tail classification: `clean` / `torn_tail` (at most one candidate record after the last valid boundary) / `corrupt_middle` and `acknowledged_tail_damaged` (both fail closed; nothing truncated). Under verified exclusive ownership and before `observed_lines` / `LEASE_ACQUIRED`: forensic copy → ownership re-check → truncate to the last valid boundary → fsync → verify size. The repair is recorded in `LEASE_ACQUIRED.tail_repair`. It is the single documented exception to append-only. Crash during repair is idempotent. A filesystem without durable truncate → `JOURNAL_TAIL_UNREPAIRABLE`, with no append. Writers check the expected EOF before every append. | §12.1, §12.1.1, §13.5, §13.1, §13.4, ADR-004, ADR-020, D1_PLAN A2, SP-W9 |
| R3-M2 | Ignored-path cleanup lacked symlink/junction/reparse handling and policy-overlap protection | **Accepted.** Policy precedence (secrets/`.git`/protected > stateful > provision/preserve > bootstrap-derived > cleanup-derived > undeclared = never). Config load rejects non-normalised, absolute or escaping patterns, non-literal cleanup roots and any overlap with a non-destructive policy. Runtime: platform capability gate → worktree identity → ancestor checks (no links, same volume, realpath match) → no-follow inventory (any link stops) → per-candidate revalidation against all non-destructive policies → bottom-up, non-recursive deletion with a per-entry identity re-check. Any failure deletes nothing further and suspends `IGNORED_STATE_UNSAFE`. Recreation quarantines by rename and never deletes. The residual ancestor-swap TOCTOU window and the minimum isolation assumptions are documented. | §13.2, §13.2.1, §13.1, §13.4, ADR-022, D1_PLAN A3/A9, SP-W8 |

### Second review findings (r3)

The second independent review (verdict CHANGES_REQUESTED) found **no blockers**. It confirmed provider integration and billing, the Model & Effort Router, token/quota efficiency and the D1A scope as sound. r3 is a **focused correction, not a redesign**. ADR-003, 007, 013, 016, 023, 024 and 028 are unchanged. ADR-018 keeps the same spikes and only aligns the gates. IDs: `M` = MAJOR, `m` = MINOR, numbered in the order of the owner's correction brief.

| Item | Finding | Resolution | Where |
|---|---|---|---|
| R2-M1 | Lease/fencing: a stale event could be accepted between lock takeover and the new `LEASE_ACQUIRED`; expiry alone permitted takeover | **Accepted.** Automatic takeover requires lease expiry **and** `dead_verified` (PID + start time, same host); `alive`/`unverifiable` → no takeover, human recovery (`resume --break-lock`). Fence = max(journal, lock) + 1. Replay rule: an older-fence line counts only if at or before the newer owner's recorded `observed_lines`. Abandoned-marker recovery, clean release (`LEASE_RELEASED`) and a crash-window table added. The distributed-grade wording is removed. Residual risk is confined to the human-forced path. | §13.5, §13.4, §12.1, ADR-020, ADR-004 |
| R2-M2 | Retry assumed Git restores ignored files | **Accepted.** Effect locations `tracked_worktree` / `untracked_nonignored` / `ignored_local` / `external_side_effecting`; `reset --hard` + `clean -fd` stated not to touch ignored files (and `-x`/`-X` never used). Ignored-state inventory per writing attempt; retry only via a declared `restore: bootstrap` (idempotent) or `cleanup: delete` (scoped) policy, worktree recreation with quarantine (no deletion), or human (`IGNORED_STATE_UNSAFE`). Secrets (`protected`) are never touched. FakeAgent contamination tests added. | §13.2, §13.3, §13.1, §11.2, ADR-022, ADR-011, D1_PLAN A3/A9 |
| R2-M3 | Fast-path self-certification | **Accepted.** Oracle set = AC-mapped tests + declared `oracle_paths`, resolved at `base_sha`; all must pre-exist; any oracle change or new test file promotes; blob hashes verified against `base_sha`; collected test ids ⊇ base, none skipped. `fast_path.enabled` defaults to `false` and the `init` template ships it disabled. The 3 files / 60 lines thresholds are configurable starting values. Post-diff checks and one-way promotion are kept. | §16.7, ADR-027, D1_PLAN A0/A8, §5 |
| R2-m1 | D0.5 phase gates misaligned | **Accepted.** Platform group SP-W1..W9 gates D1A; provider group SP-C*, SP-A*, SP-W10 gates D1B. Amendments flow to the phase they gate. The cross-cutting prerequisites that must complete before D1A are SP-W4 (process identity, now load-bearing for takeover), SP-W6/W9 (lock, append, crash) and SP-W8 (worktrees, `git status --ignored`). | ADR-018, D1_PLAN §1, §2.4, §20 |
| R2-m2 | `status` must stay read-only | **Accepted.** `RECOVERY_REQUIRED` is a derived display condition; `status` never takes the lease or writes anything. | §6.1, §13.6, ADR-019 |
| R2-m3 | Checkpoint adoption trusted any stored result | **Accepted.** `result.json` must be atomic, schema-valid, attempt-bound (run/seq/attempt/fence) and hash-verified against the `ai-flow-result` commit trailer. | §13.3, §12.2, §14, ADR-011 |
| R2-m4 | Abandoned `run.lock.takeover`; normal release undefined | **Accepted** (with R2-M1). | §13.5, ADR-020 |
| R2-m5 | `PAUSING` looked like a durable status | **Accepted.** `PAUSE_REQUESTED` event + transient display condition only; a pending pause after a crash is superseded by `resume`. | §6.1, §13.6, §13.4, ADR-019 |
| R2-m6 | Tag-based document discovery in D1A | **Accepted.** Deferred to [SOON]; D1 uses explicit references only. | §10.5, §5.1, §18.1, D1_PLAN §6 |
| R2-m7 | Reroute phase labels inconsistent | **Accepted.** D1A: CLI stub, epoch field fixed at 1, event schema only, never offered as a decision; D1B: implementation. | §16.10, §13.1, §6.3, ADR-015 |
| R2-m8 | Unverifiable provider-reported model | **Accepted.** An undeclared reported model fails closed (`MODEL_UNVERIFIED`; result not accepted). A surface that does not report the model is labelled "request-verified only". | §16.4, §13.1, ADR-025 |
| R2-m9 | Effort-first default must stay a heuristic | **Accepted (no change in substance).** Restated explicitly as an empirical, configurable heuristic, not an invariant; the invariants are listed separately. | §16.6, ADR-026 |
| R2-m10 | Persistent failures ≠ lack of reasoning | **Accepted.** Triggers renamed to observed evidence (`same_failure_persists`, `output_or_tool_failure`, `persists_at_max_effort`), with `trigger_evidence` journaled. | §16.6, ADR-026 |

### Codex review findings (A1–A18)

| Item | Topic | My severity | Resolution | Where |
|---|---|---|---|---|
| A1 | Capability-oriented provider integration; Codex SDK-first; exec as reference; app-server only on evidence | MAJOR | **Refined**: accepted; additionally noted that the Codex SDK is expected to wrap the CLI binary, so process supervision still applies | §3, §8, ADR-003 |
| A2 | Mandatory D0.5 spikes | BLOCKER | Accepted | §20, ADR-018, D1_PLAN §2 |
| A3 | Correct idempotency guarantee; effect classes | BLOCKER | Accepted | §13.2, ADR-011, ADR-022 |
| A4 | Suspensions are not terminal; continuation record | BLOCKER | Accepted | §6.1, §6.3, ADR-019 |
| A5 | Billing protection fails closed; attestation | BLOCKER | **Refined/disagreed in part**: for Claude, strict blocking could make subscription use impossible if no machine-verifiable surface exists. The proposed `human_attested` opt-in was **approved by the owner** with conditions. Codex stays strictly machine-verified. | §17.3, ADR-016 |
| A6 | Honest D1 scope (Git-based projects) | MINOR | Accepted | §1, §2, ADR-028 |
| A7 | Journal authoritative, state derived, Git reconciled; 3 cases; no agent commits; hooks bypassed | MAJOR | Accepted; case 2 also requires the stored structured result, otherwise the result is treated as lost (never accept a checkpoint without its result) | §12.1, §13.3, §14, ADR-004, ADR-009, ADR-011 |
| A8 | Atomic lock + lease + fencing | MAJOR | **Refined**: accepted; added fencing on replay (stale-fence events ignored) and documented the residual file-system race, which is covered by Git reconciliation. *Superseded in r3 by R2-M1* | §13.5, ADR-020 |
| A9 | State in `~/.ai-flow/runs/<repo-instance-id>/<run-id>/`; sibling worktrees; repo-instance identity | MAJOR | **Refined**: accepted; identity stored in the git common dir; moved/copied clones detected; the loss of co-location is mitigated by `report --export` [SOON] | §5.1, §14, ADR-005, ADR-021 |
| A10 | Human-approved reroute with epochs | MAJOR | Accepted | §16.10, ADR-015 |
| A11 | Evidence rules; structural lineage; `repeat_of` advisory; test-decided ACs | MAJOR | **Refined**: accepted; a reviewer may still challenge test *adequacy* with evidence but cannot override pass/fail; ambiguous lineage matching prefers merging (fails towards earlier human escalation) | §9, ADR-006 |
| A12 | Context Builder honesty; bundle hashes; cached tokens; incremental re-review; stable ordering | MINOR | Accepted | §10, ADR-008 |
| A13 | Staged validation; collect cheap failures; no AI repair for environment failures | MINOR | **Refined**: collect within a stage, fail fast between stages | §11, ADR-013 |
| A14 | Enforcement levels native / os_isolated / posthoc / instruction_only | MAJOR | Accepted; minimum levels enforced by refusing routing | §15.1, ADR-023 |
| A15 | Expanded audit metadata; "reconstructable/attributable" | MINOR | **Refined**: invocation stored as a sanitized hash + structured fields, never the raw command line or env | §12.3, §12.4 |
| A16 | Windows coverage | MAJOR | Accepted; split between D0.5 spikes (feasibility) and D1A/D1B acceptance tests | §3, §13.5, D1_PLAN SP-W*, D1A/D1B ACs |
| A17 | D0.5 / D1A / D1B restructuring | MAJOR | Accepted | §20, D1_PLAN |
| A18 | Dependency-free Node 22 ESM fixture with node:test | MINOR | Accepted | D1_PLAN §5 |

"My severity" is the author's assessment. The first review returned an overall verdict without per-item severities.

### Owner requirements (B, C, D)

| Item | Topic | Resolution | Where |
|---|---|---|---|
| B1 | Abstract profiles FAST/STANDARD/STRONG/CRITICAL (renamed from r1's FAST/STANDARD/HIGH to avoid confusion with effort "high") | Added | §16.4, ADR-025 |
| B2 | Model and effort as separate axes; policy for effort-vs-model | Added (configurable ladders; default heuristic marked empirical) | §16.4, §16.6, ADR-025, ADR-026 |
| B3 | Bounded escalation, persisted with reasons | Added | §16.6, ADR-026 |
| B4 | De-escalation; per-step resolution | Added (escalation keyed by lineage) | §16.5 |
| B5 | Deterministic before AI | Strengthened | §4, §11, ADR-013 |
| B6 | Quota awareness: measured/estimated/unknown/known exhaustion; no invented quota | Added | §17.1, §13.1 |
| B7 | Risk classes LOW/NORMAL/HIGH/CRITICAL; minimum profile floors | Added | §7.2, §16.5, ADR-026 |
| B8 | Independence not sacrificed for cheaper models | Added | §16.3, ADR-007 |
| B9 | Historical routing-evaluation metadata; no ML router in D1 | Added | §12.3, §16.11 |
| B10 | Provider-specific mapping owned by the adapter; no silent substitution | Added | §16.4, ADR-025 |
| B11 | Small-task fast path with guardrails [OWNER: project opt-in] | Added | §16.7, ADR-027 |
| B12 | Context and routing together; stable prefixes; hashes; no transcripts | Added | §10, §16.8 |
| C | Spec Kit / MCP / Skills optional | Updated | §19, ADR-010 |
| D | Simplification: one typed workflow, reducer, FakeAgent, no DSL/registry | Applied | §6.2, §18, ADR-024 |

### Owner decisions recorded in r2

1. **Claude billing fallback** [OWNER, approved]: default fail closed. If D0.5 proves that no reliable machine-verifiable auth/billing mode exists for Claude Code, a project may opt in to `billing_verification: human_attested`. This requires explicit human approval, a non-secret attestation, verified absence of known billing-changing variables/overrides, no fall-through to paid usage, and a distinguishable audit marker. An unknown billing mode without machine verification or explicit attestation **blocks** execution. (ADR-016)
2. **Fast path** [OWNER, approved]: project opt-in with conservative deterministic guardrails. (ADR-027) *r3 per owner correction brief: the opt-in is now explicit (default `enabled: false`, `init` ships it disabled), and oracle pre-existence and unchanged oracles are required.*

---

## ADR-001 — Local-first CLI orchestration, not a server

**Context.** The user runs agent tools locally under personal subscriptions, against local Git repositories, one task at a time. The requirements stress resumability, not multi-user access.

**Decision.** ai-flow is a local CLI process. Each invocation drives a run until it reaches a terminal state or a suspension, then exits. There is no daemon in D1. All state is on disk.

**Alternatives.** A local daemon (background queueing, but lifecycle management and a second failure surface). Server/cloud orchestration (remote credentials and checkouts; conflicts with subscription tooling).

**Consequences.**
- \+ Simple and inspectable.
- \+ A crash is a process exit, and recovery means re-running the CLI.
- − No background execution unless a terminal stays open.
- − No multi-machine operation (a non-goal).

**Status / Confidence.** Accepted / High.

---

## ADR-002 — The orchestrator owns the workflow; agents execute steps

**Context.** Chaining agents with control flow in prompts makes loops unbounded and runs non-resumable.

**Decision.** A deterministic reducer (ADR-024) decides every transition from schema-validated outcomes and enforces counters. Agents receive a role, a contract and a context bundle, and return structured results. Agents never choose the next step, never invoke other agents, and never commit (ADR-009).

**Alternatives.** Agent-led orchestration (a "manager" agent) is flexible but non-deterministic, hard to bound and hard to resume.

**Consequences.**
- \+ Testable with FakeAgent.
- \+ Limits are real.
- \+ Provider-agnostic.
- − Adaptivity is limited to outcome mapping and routing policy.

**Status / Confidence.** Accepted / High.

---

## ADR-003 — Capability-oriented provider adapters; Codex SDK-first, conditional on D0.5 (r2)

**Context.** r1 assumed every provider is a CLI subprocess behind one generic interface. Codex offers an official SDK, `codex exec` and an app-server. Claude Code offers a print-mode CLI and an Agent SDK. They differ in lifecycle, structured output, permissions, auth inspection and error signalling. Forcing a lowest common denominator would discard capabilities that the design needs (thread resume, interrupt, auth inspection).

**Decision.**
- The core is written against **capabilities**, not mechanisms (§8.1). Each adapter declares its capabilities and the enforcement levels it can deliver. The core applies documented per-capability fallbacks or refuses routing. There is no silent degradation.
- **Codex:** the official SDK is the preferred D1B surface, **subject to D0.5**. `codex exec` is the bounded reference/scripting surface and fallback. The **app-server** is used only if D0.5 shows that the SDK cannot cleanly provide a D1 requirement:
  - interrupt with a resumable thread;
  - positive auth/account-mode inspection;
  - distinguishable rate-limit vs quota signals;
  - approval/sandbox control for a native read-only reviewer;
  - reliable Windows lifecycle.

  Moving to app-server requires amending this ADR.
- **Claude Code:** the print-mode CLI with JSON/stream-JSON output and session resume is the expected surface. The Agent SDK is evaluated in D0.5 and chosen only if superior **and** compatible with subscription billing under fail-closed rules.
- Adapters own invocation, classification, extraction, permission/profile translation, usage and auth attestation. They never retry or contain workflow logic.

**Alternatives.**
- One generic CLI interface (r1): simplest, but it loses capabilities and hides differences.
- Direct model APIs: API billing, and ai-flow would turn into an agent framework.

**Consequences.**
- \+ No lowest common denominator; the core stays provider-neutral.
- − Two different adapter shapes to maintain.
- − The surface choice is unresolved until D0.5.
- − If the SDK wraps the CLI binary [ASSUMPTION], Windows process supervision is still needed.

**Status / Confidence.** Accepted / Medium (the shape is confident; the surfaces are unverified).

---

## ADR-004 — Journal authoritative, state derived, Git reconciled (r2)

**Context.** Runs must survive crashes and reboots at any instruction. r1 was ambiguous about which store wins.

**Decision.**
- `events.jsonl` is the **authoritative history**. It is append-only and fsync'd, and each event carries a seq, schema version, fence, wall-clock time and monotonic time.
- `state.json` is a **derived cache** (a fold of the reducer), written atomically and rebuildable at any time.
- **Git is an external fact.** It is never assumed to match the journal and is reconciled on resume (ADR-011).
- Provider thread state is an external **hint** only.
- Replay applies the fence replay rule of ADR-020 (r3): an older-fence event counts only if the newer owner had observed it at takeover. Events are ordered by line position. Non-final corruption → `STATE_CORRUPT`.
- (r4) Every record carries a checksum.
- (r5) The journal is append-only **without exception**. A damaged final record is never truncated, rewritten or appended after. It fails closed (`JOURNAL_TAIL_AMBIGUOUS`, or `STATE_CORRUPT` with positive acknowledgement evidence) and needs explicit human recovery (ADR-020, §12.1.1). `state.json` is a non-authoritative cache: it may give positive evidence that a record was acknowledged, but its absence or staleness is never evidence that a record was not.
- Read-only commands (`status`) fold the journal in memory and never write (r3).
- Run state lives under `~/.ai-flow/runs/<repo-instance-id>/<run-id>/` (ADR-021).

**Alternatives.**
- SQLite: transactional, but a native dependency, less inspectable, and not needed for one run at a time.
- State-only persistence: crash windows become hard to reason about.

**Consequences.**
- \+ Inspectable.
- \+ The journal doubles as audit trail and eval dataset.
- − Atomic writes, replay and fencing must be hand-rolled, with Windows retry.

**Status / Confidence.** Accepted / Medium.

---

## ADR-005 — Global vs project config; runtime state outside the project (r2)

**Context.** The engine must be reusable across projects. r1 kept runtime state in `.ai/runs/` inside the repo.

**Decision.**
- Configuration layers, in precedence order: task > project (`.ai/`) > global (`~/.ai-flow/`) > built-in.
- Task and project may only **raise floors and lower limits** relative to the layer below (monotonic config).
- `.ai/` contains only committed, reviewable configuration and task contracts.
- **Runtime state lives globally** (ADR-021).
- Provider mappings (model names), billing settings and policies are **global**, so model names never appear in project workflows.
- The resolved config is frozen per run.

**Alternatives.** All-project (duplicates agent config in every repo; runtime clutter); all-global (projects become implicit).

**Consequences.**
- \+ Project behaviour is versioned with the project.
- \+ Machine concerns stay global.
- − Users need `status` to locate run data.

**Status / Confidence.** Accepted / High.

---

## ADR-006 — Structured review with evidence, structural lineage and test-decided ACs (r2)

**Context.** Free-form reviews cause scope creep and loops. r1 fingerprinted findings with text, and trusted the model's `repeat_of`.

**Decision.**
- The reviewer returns `review@2` (§9.1). A finding is **actionable** only if all of the following hold:
  - severity is blocker/major;
  - it cites a valid ref;
  - it carries concrete evidence that exists at the reviewed checkpoint;
  - it states `required_behavior`;
  - it is not contradicted by deterministic results;
  - its lineage is not suppressed.
- **Lineage** = (ref, category, normalised file, location anchor). Text similarity is only a tie-breaker. Model `repeat_of` is advisory. Ambiguity merges lineages.
- ACs with `verify: test` are **decided by mapped tests**. The reviewer may challenge test adequacy with evidence, but cannot override pass/fail.
- Verdict normalisation turns "changes requested with nothing actionable" into approval.

**Alternatives.** Trusting the reviewer's verdict; numeric scores.

**Consequences.**
- \+ Bounded, contract-anchored review.
- \+ Comparable data across reviewers.
- − Poorly evidenced real issues become non-actionable (still reported).

**Status / Confidence.** Accepted / High.

---

## ADR-007 — Implementer/reviewer independence, explicit fallback only (r2)

**Context.** Independence is a core quality invariant. Quota pressure and cheaper-model routing create a temptation to weaken it.

**Decision.**
- `independence: required` by default (scope: provider).
- `fallback` is explicit and is evaluated only at run creation for an unavailable primary. Mid-run unavailability or quota suspends the run.
- Self-review requires `fallback: <same>` **and** `allow_self_review: true`, and is flagged `degraded_independence`.
- **Model/effort optimisation (ADR-025) never affects independence.**
- Reroutes (ADR-015) re-check independence.

**Consequences.**
- \+ The invariant holds unless consciously and visibly disabled.
- − Runs wait when the reviewer is unavailable.

**Status / Confidence.** Accepted / High.

---

## ADR-008 — Context bundles: minimal, ordered, hashed, incremental re-review (r2)

**Context.** Token efficiency is a must. r1 implied that the Context Builder controls consumption. It only controls the initial prompt.

**Decision.**
- Each step gets a **context bundle**: ordered segments (static instructions → contract → stable references → semi-stable state → volatile evidence) for prefix caching.
- The bundle records a hash of the exact rendered bytes, per-item hashes, the checkpoint SHA, the template version and estimated sizes.
- Re-review receives the contract + unresolved lineages + validation delta + diff since the last reviewed checkpoint. It falls back to the full bounded diff after an epoch change or a large scope shift.
- No transcripts or reasoning are forwarded.
- Cached-token usage is recorded when reported.
- Bundle size is a routing input (ADR-025).
- The docs state plainly that agents with read tools consume beyond the bundle.

**Alternatives.** Session continuity as the default (non-resumable, unbounded growth; kept only as an optional optimisation). Retrieval/embeddings from day one.

**Consequences.**
- \+ Auditable, cache-friendly, resumable.
- − Some re-orientation tokens.
- − Token estimates are approximate.

**Status / Confidence.** Accepted / Medium.

---

## ADR-009 — Git isolation: sibling worktree, orchestrator-only checkpoints, hooks bypassed (r2)

**Context.** Agents must never touch `main` or the user's tree. r1 put worktrees inside the repo, squashed agent commits and ran repository hooks on checkpoints.

**Decision.**
- Each run gets branch `ai-flow/<task-id>-<run8>` in a **sibling worktree** (`<repo-parent>/.ai-flow-worktrees/<repo8>/<run8>/`, configurable, same volume preferred).
- **Only the orchestrator commits**, with trailers (run, attempt, step, epoch, fence, result hash — r3).
- **Agent-created commits are prohibited** and trigger `GIT_DIVERGED`. They are not squashed.
- Checkpoints **bypass repository hooks**. Quality checks run as explicit validation stages.
- Merging and pushing are human actions in D1.
- A repo-level lock serialises ref mutations.

**Alternatives.**
- In-repo worktrees: tool duplication, nested-repo confusion.
- Squashing agent commits (r1): hides divergence.
- Running hooks: arbitrary code with arbitrary effects, non-reconcilable.

**Consequences.**
- \+ Clean isolation, unambiguous reconciliation.
- − Projects relying on hooks must declare them as validation checks.
- − The worktree is less discoverable.

**Status / Confidence.** Accepted / Medium.

---

## ADR-010 — Spec Kit as an optional integration, not a core dependency

**Context.** Spec Kit structures large features. Most tasks are small.

**Decision.** Spec Kit is integrated [FUTURE] as a separate typed workflow or an external pre-step that **emits ai-flow task contracts**. It is never used for tiny tasks or on the fast path. The core never imports it. MCP and Skills are likewise optional (§19).

**Consequences.**
- \+ Optional value with zero coupling.
- − Multi-task execution awaits the future queue.

**Status / Confidence.** Accepted / High.

---

## ADR-011 — Recovery invariant: at most one accepted result per attempt; reconciliation (r2)

**Context.** r1 claimed "exactly-once step effects", which is not achievable for agent execution or external effects.

**Decision.**
- **Invariant:** at most one accepted result and at most one accepted checkpoint per attempt identity `(run, seq, attempt)`. Execution may repeat. Only `tracked_worktree` and `untracked_nonignored` effects are reconciled automatically by Git. `ignored_local` effects are handled only by the ignored-state check and a declared policy (r3). External side effects are prohibited by default and never reconciled (ADR-022).
- **Resume order:** lease (ADR-020) → fence-aware replay → snapshot verification → billing re-attestation → Git reconciliation → ignored-state check → provider-thread evaluation.
- **Git reconciliation** for an interrupted writing attempt from `S`:
  1. `HEAD == S` → save the partial patch (tracked + untracked-non-ignored), `reset --hard` + `clean -fd` (never `-x`), then the ignored-state check (ADR-022), then retry or suspend.
  2. `HEAD` is a single orchestrator checkpoint with parent `S` and this attempt's trailers → adopt it **only if `result.json` is atomic-published, schema-valid, attempt-bound (run/seq/attempt/fence equal the trailers) and hash-verified against the `ai-flow-result` trailer** (r3). Otherwise journal `CHECKPOINT_ADOPTED_RESULT_LOST`, reset, and retry.
  3. Anything else → `WAITING_HUMAN: GIT_DIVERGED`.
- Read-only attempts: any change means `POLICY_VIOLATION` or `GIT_DIVERGED`.
- Provider threads are a validated optimisation, never the truth.
- Classification uncertainty always resolves towards suspension.

**Alternatives.** Resuming the agent's half-finished session (unreliable); a durable-execution framework (ADR-014).

**Consequences.**
- \+ An honest, testable guarantee.
- − Quota is re-spent on interrupted steps.

**Status / Confidence.** Accepted / Medium (Windows edge cases are pending D0.5).

---

## ADR-012 — Declarative YAML workflows (SUPERSEDED)

**Superseded by ADR-024 in r2.** The reviewer warned against building a miniature workflow platform in D1. A YAML DSL may return [FUTURE] once three or more real workflows exist.

---

## ADR-013 — Staged deterministic validation gates AI review (r2)

**Context.** AI review is expensive. r1 ran checks sequentially with simple fail-fast.

**Decision.**
- Stages, in order: preflight (tools/environment) → bootstrap (only if inputs changed) → static → focused/AC-mapped tests → broad tests/build → policy (scope, protected paths, gates, secrets) → AI review.
- Within a stage, all failures are collected. Between stages, the run fails fast.
- **Environment failures** (missing tool, broken baseline, unavailable service, network) suspend and are **never** sent to AI repair.
- The baseline on `base_sha` is cached. A failing baseline needs a human decision.
- D1 accepts only `pure`/`workspace_local` checks (ADR-022).

**Consequences.**
- \+ Cheaper, more reliable reviews.
- \+ The reviewer can stay read-only.
- − Projects must configure stages (the `init` template helps).

**Status / Confidence.** Accepted / High.

---

## ADR-014 — TypeScript on Node.js 22, no orchestration framework (r2)

**Decision.**
- TypeScript on **Node.js 22 LTS**.
- Minimal dependencies: a schema library, a YAML parser (config only), a CLI parser, the provider SDK(s) chosen by D0.5, and the built-in `node:test`.
- No Temporal/LangGraph/Airflow/Prefect.
- Windows process-tree handling (Job Objects or `taskkill /T`, chosen in D0.5) is implemented explicitly.

**Alternatives.** Python; Go/Rust; durable-execution frameworks.

**Consequences.**
- \+ Fast iteration and typed schemas, in the same ecosystem as the provider SDKs.
- − Windows process handling needs deliberate work.

**Status / Confidence.** Accepted / Medium.

---

## ADR-015 — Routing frozen per epoch; human-approved reroute creates a new epoch (r2)

**Context.** Re-evaluating routing on resume could silently change reviewers. But a quota exhaustion or outage may justify a deliberate change.

**Decision.**
- Routing is resolved at run creation as **epoch 1** and persisted. Resume never re-routes.
- `ai-flow reroute` (allowed only while suspended) is itself the human approval. It must:
  - re-check independence;
  - re-attest billing;
  - require a **separate** explicit approval for any subscription → paid/unknown change (`--approve-paid-billing` + `allow_paid_api` + a spend ceiling);
  - journal `ROUTING_EPOCH_STARTED` with the reason and time.
- Prior results keep their epoch attribution.
- Re-review after a new epoch uses the full bounded diff.
- Automatic mid-run fallback **never** happens. Quota exhaustion may only *offer* a reroute decision.
- **Phases (r3):** D1A = `reroute` CLI stub, `epoch` fixed at 1, `ROUTING_EPOCH_STARTED` schema only; `reroute` is never a permitted decision in D1A. D1B = the full implementation (D1_PLAN B4).

**Consequences.**
- \+ Predictable runs, clean attribution, and a controlled escape hatch.

**Status / Confidence.** Accepted / High.

---

## ADR-016 — Billing protection fails closed with non-secret attestation (r2) [OWNER]

**Context.** Agents may run under subscriptions or pay-per-use API keys. Env vars, config files (including project-level provider settings in the repo) and key helpers can switch billing silently. Stripping env vars is not sufficient proof. Model availability proves nothing about entitlement.

**Decision.**
- Each provider declares `billing` (`subscription | api`) and `billing_verification` (`machine_verified | human_attested`).
- **Attestation** happens at run start, on resume, on every epoch change and, where cheap, before each turn. It requires:
  - positive verification of the active auth mode, using a mechanism selected in D0.5;
  - verified absence of known billing-changing env vars **and** config overrides.

  The result is persisted as a **non-secret attestation** referenced by every attempt.
- **Codex** `billing: subscription` must be `machine_verified`. Without a D0.5-proven mechanism, Codex subscription use is blocked.
- **Claude Code** [OWNER, approved]: default `machine_verified`. Only if D0.5 proves no reliable machine-verifiable mechanism exists, a project may opt in to `human_attested`, which:
  - requires an explicit human approval per run/epoch;
  - records a non-secret attestation;
  - requires the billing-changing variables/overrides to be verified absent;
  - never falls through to paid usage;
  - is always distinguishable from `machine_verified` in audit metadata.
- **Unknown billing mode** without machine verification or explicit attestation **blocks execution**.
- A detected mode change suspends the run.
- `billing: api` requires explicit routing + `allow_paid_api` + an approved spend ceiling.
- Usage figures are labelled measured/estimated/unknown/known_exhaustion. No remaining-quota percentages are invented.

**Alternatives.**
- Env stripping only (r1): insufficient.
- Strict blocking for Claude without an attestation option: it may make subscription use impossible, so it was rejected by the owner in favour of a controlled opt-in.

**Consequences.**
- \+ No surprise paid usage.
- − A provider may be unusable for subscription work until verification is possible.
- − Human attestation is a weaker, explicitly labelled guarantee.

**Status / Confidence.** Accepted / High.

---

## ADR-017 — Versioning: typed workflow version + prompt id@semver + hashes, snapshots (r2)

**Decision.**
- The typed workflow carries a `code-workflow@semver` constant + a hash of its serialised transition table.
- Prompts carry `id@semver` + a content hash. Output schemas are versioned (`review@2`, `fix_response@2`).
- Provider mappings and ladders are part of the config snapshot, and each routing decision records `mapping_hash`.
- Runs store copies of all of these.
- Templates modified without a version bump are tagged `+dirty` in D1A. Strict rejection is [SOON].
- Skills are hashed where practical.

**Consequences.**
- \+ The meaning of a historical run is immutable and attributable.
- − Small duplication per run.

**Status / Confidence.** Accepted / Medium.

---

## ADR-018 — Mandatory D0.5 feasibility spikes before D1

**Context.** Several load-bearing decisions depend on unverified provider and platform behaviour:

- structured output;
- read-only enforcement;
- auth-mode inspection;
- interrupt/resume;
- error signatures;
- Windows process trees, locks and paths.

**Decision.** A D0.5 milestone runs **time-boxed spikes** (throwaway code outside the product tree) for Codex (SDK, exec, app-server where relevant), Claude Code (CLI, Agent SDK evaluation) and the Windows platform (D1_PLAN §2). Outputs:

- a filled capability matrix and enforcement-level matrix;
- the selected surface per provider;
- the billing verification method per provider, or a documented "blocked";
- an amendment list for this architecture.

**Phase gates (r3, aligned with D1_PLAN §1/§2.4):**

- **Platform group** (SP-W1..W9: process trees, shims, Ctrl+C, process identity, rename/lock contention, paths, Git worktrees and `git status --ignored`, crash recovery) **gates D1A**. Its amendments must be merged before D1A.
- **Provider group** (SP-C*, SP-A*, and SP-W10, which is provider sandbox behaviour on Windows) **gates D1B**. Its amendments must be merged before D1B, and may amend the `ProviderAdapter` seam that D1A built against FakeAgent. That rework risk is accepted.
- **Cross-cutting prerequisites** that must be complete before D1A because D1A safety depends on them: SP-W4 (PID + start-time liveness, load-bearing for lease takeover), SP-W6 + SP-W9 (exclusive publish, append atomicity, crash windows), SP-W8 (worktrees, ignored-state inventory). SP-W1..W3 are needed for the validation supervisor and Ctrl+C in D1A; their provider-CLI variants (shims of the provider binaries) are re-checked in the provider group.
- No provider spike gates D1A.

**Consequences.**
- \+ Evidence before code.
- − Calendar time before D1.
- D0.5 may change the adapter/recovery design. That is the point of the milestone.

**Status / Confidence.** Accepted / High.

---

## ADR-019 — Suspension overlays; only DONE/FAILED/CANCELLED are terminal (r3, r6)

**Context.** r1 modelled `HUMAN` as a terminal workflow step, which conflicts with resume.

**Decision.**
- Run status is `QUEUED | ACTIVE | DONE | FAILED | CANCELLED`. Only the last three are terminal.
- `PAUSED`/`WAITING_HUMAN` are **suspension overlays** on an ACTIVE run. Each carries a continuation record:
  - suspended step;
  - pending action id;
  - permitted decisions, each with its transition;
  - checkpoint SHA;
  - provider thread identity;
  - interrupted-attempt validity;
  - routing epoch;
  - optional `auto_resume_after`.
- Resume applies only a permitted decision. Every decision is journaled.
- **Display conditions (r3):** `PAUSING` (a journaled `PAUSE_REQUESTED` not yet followed by a suspension) and `RECOVERY_REQUIRED` (lease/marker/journal/reconciliation needs `resume` or a human) are derived by the read-only `status` command. They are **not** run statuses or suspension kinds and are never persisted as such. `status` never writes.
- (r6) `RECOVERY_PREPARING` and `RECOVERY_INCONSISTENT` are **reasons** shown under `RECOVERY_REQUIRED` for an unpublished or inconsistent recovery successor (§12.1.2). They are not statuses, suspension kinds or workflow steps, and the reducer never sees them. Journal damage never moves a run to `FAILED`. `FAILED` is reached only by a broken workflow invariant (§6.1).
- (r6) A recovery successor is published already suspended `WAITING_HUMAN: JOURNAL_RECOVERED`. `resume` never implies `continue`. `continue` must quote the predecessor journal hash and the successor id that the suspension is bound to.

**Consequences.**
- \+ Deterministic, safe resume from any suspension.
- \+ Displayed states stay familiar (derived).

**Status / Confidence.** Accepted / High.

---

## ADR-020 — Fenced lease lock; takeover only on verified owner death; damaged journal tail fails closed; staged successor publication (r3, r4, r5, r6, r7)

**Context.** PID/heartbeat stale-lock detection alone allows two writers after PID reuse, clock issues or a paused process. The second review found two defects in r2:

- expiry alone allowed takeover from a live (e.g. suspended) owner;
- the replay rule ("ignore events whose fence is lower than a *preceding* `LEASE_ACQUIRED`") accepted a stale event appended between the taker's lock write and its `LEASE_ACQUIRED`.

**Decision** (single machine, local filesystem; full protocol in §13.5).

- **Liveness verdict** from PID + process start time + hostname: `dead_verified` / `alive` / `unverifiable`.
- **Automatic takeover requires both** lease expiry **and** `dead_verified`. Lease expiry alone never permits takeover from a live process. `alive` → no takeover. `unverifiable` → no takeover; the run is suspended for human recovery (`RECOVERY_REQUIRED` display; `resume --break-lock`, which refuses while any owner is `alive` and requires explicit confirmation otherwise).
- **Fence** = `max(highest journal fence, current lock fence) + 1`. No fence is reused after a crash between the lock write and `LEASE_ACQUIRED`.
- **Acquisition:** the complete lock is published exclusively (hard link, or `wx` fallback per SP-W6). **Takeover:** exclusive `run.lock.takeover` marker → re-read the lock → atomic replace → tail check → record `observed_lines` → `LEASE_ACQUIRED { fence, observed_lines, … }` → delete the marker.
- **Two-phase acquisition (r7) [OWNER].** Taking the lock never appends to the journal by itself. **`acquire_lock_only`** (acquire steps 1–2, takeover steps 1–5, also on the `--break-lock` path) establishes exclusive ownership and the fence, validates prior-owner death or the explicit break-lock conditions, and writes only lock files: no `events.jsonl` append, no `state.json`, no Git, no process. Then, **under the held lock**, the caller re-checks ownership, classifies the tail, and repeats its readiness checks (ordinary commands) or installs/verifies the prefix and the successor initialization state (recovery P4), and records `observed_lines`. Only if all of that passes does **`append_lease_acquired`** re-check ownership and the expected EOF and append and fsync exactly one `LEASE_ACQUIRED` with the held fence. A failure under the lock appends nothing: the lock is expired and kept (its fence stays counted), the marker is deleted, and the process exits. No `state.json` write, other event, process spawn or Git command precedes the acknowledged `LEASE_ACQUIRED`, and no Git mutation precedes any other event the command requires. A crash between the two phases leaves a lock fence with no lease record. The retry is a new acquisition (normal takeover rules, fence = max(journal, lock) + 1, all checks repeated) and appends one record, so each acquisition contributes at most one `LEASE_ACQUIRED` and no fence appears twice.
- **Abandoned marker:** cleared (rename + delete) only if its creator is `dead_verified`; `alive` → bounded back-off; `unverifiable` → human.
- **Clean release:** expire the lease → `LEASE_RELEASED` → delete the lock.
- **Fencing on write** (defence in depth): re-check the lock before each append and Git mutation; stop with `FENCED`.
- **Replay rule:** a line at position `i` with fence `f` is accepted iff `i ≤ L.observed_lines` for every `LEASE_ACQUIRED` L with `L.fence > f`.
- An explicit crash-window table covers acquire, takeover, marker and release (§13.5).
- **Damaged journal tail (r4 checksums; r5 fail-closed rule, §12.1.1).** The third review found that a torn final record was only skipped logically, so the next owner could append after it. r4 truncated it automatically. The fourth review found that r4 relied on `state.json`, a disposable cache, as the only evidence that the record had been acknowledged. r5 decision [OWNER]:
  - Records carry a checksum. A record is acknowledged when its fsync returned. No dependent action (next append, `state.json`, Git mutation, provider turn, human confirmation, lock release) precedes acknowledgement. Downstream effects are therefore **positive** evidence of acknowledgement, but their absence proves nothing.
  - Tail classes: `clean`; `tail_ambiguous` (at most one candidate record after the last valid boundary `B`, no positive evidence) → `JOURNAL_TAIL_AMBIGUOUS`; `acknowledged_tail_damaged` (same shape, plus positive evidence: a valid `state.json` beyond `B`, or no lock) → `STATE_CORRUPT`; `corrupt_middle` → `STATE_CORRUPT`.
  - **Fail-closed invariant:** for every non-`clean` tail there is no truncation, no rewrite, no append (including `LEASE_ACQUIRED`), no seq reuse, no `state.json` write and no Git mutation. The journal stays byte-identical. The taker expires its own lock (kept, so its fence is counted), deletes its marker and exits; `status` shows `RECOVERY_REQUIRED`. `--break-lock` cannot bypass this.
  - An interrupted append and a later-damaged acknowledged record are indistinguishable in D1 → suspend. No acknowledgement watermark or commit protocol is introduced to tell them apart.
  - **Read-only diagnostic** (`status`, `doctor --journal`): offsets, hashes, a bounded preview, an untrusted parse, `state.json` evidence labelled non-authoritative, lock state, Git facts.
  - **Human recovery:** restore from an independent copy; or (r7: only when `B > 0`) `ai-flow recover --journal --tail-sha256 <h>`, which leaves the old run byte-identical (plus an exclusive `run.sealed`), builds a successor run from the hash-verified prefix `[0, B)`, reserves seq `last_valid_seq + 1`, journals `JOURNAL_RECOVERED`, and suspends `JOURNAL_RECOVERED` before any Git mutation; or abandon. Git reconciliation then recovers workspace effects only (case-2 adoption under its full conditions). Lost human decisions, routing choices, suspensions and attestations are **not** reconstructed; a lost decision must be re-made by the human.
  - The append guard is unchanged: writers verify the expected end-of-file offset before every append and never append after a non-`clean` tail.
- **Successor initialization and publication (r6, §12.1.2).** The fifth review found that a crash between the successor's `LEASE_ACQUIRED` and `JOURNAL_RECOVERED` left a resumable, ordinary-looking run that a rerun could not find. Decision:
  - **Staging:** a successor is built only in `runs/<repo>/.recovering/<successor-id>/`, which ordinary commands never resume, continue or reconcile.
  - **Deterministic identity:** `successor_run_id` = f(sha256(predecessor run id ‖ predecessor journal sha256 ‖ recovery generation)); `recovery_generation` = predecessor's + 1 (normal run: 0).
  - **Provenance first:** the immutable recovery descriptor `recovery.json` is published exclusively before any journal is written.
  - **Fixed order:** (r7) staging lock ownership only (`acquire_lock_only`, §13.5 rules; P3) → under that lock, hash-verified prefix (published atomically) and verification of descriptor, predecessor hash, seal state and initialization state (P4) → `LEASE_ACQUIRED { mode: recovered }` (`append_lease_acquired`; P5) → `JOURNAL_RECOVERED` → suspension `WAITING_HUMAN: JOURNAL_RECOVERED` with `binding { predecessor_journal_sha256, successor_run_id }` → predecessor `run.sealed` (exclusive) → `recovery.published` marker → **one same-volume directory rename** into the run namespace (the atomic visibility boundary) → lease release.
  - **Structural distinction:** a normal run's first record is `LEASE_ACQUIRED { mode: fresh, run_id: own }`; a successor's first record is the predecessor's. A foreign first `run_id` without a valid descriptor and marker, or the reverse, → `RECOVERY_INCONSISTENT`.
  - **No valid prefix (r7) [OWNER]:** `B = 0` means no complete valid predecessor record exists. `recover --journal` refuses it (`RECOVERY_NO_VALID_PREFIX`) read-only at P0: no staging directory, descriptor, successor journal, predecessor seal, publication marker or lock is created, and the predecessor journal stays byte-identical. The diagnostic explains why successor recovery is unavailable. The human restores an independently verified intact copy or abandons. There is no empty-prefix successor format, so every successor's first record is the predecessor's and the structural distinction is never weakened; a descriptor with `prefix_end: 0` is schema-invalid.
  - **Readiness gate:** run read-only by every mutating entry point before any side effect, and repeated under the held lock before `LEASE_ACQUIRED` is appended (r7). It verifies the descriptor, marker, seal, predecessor journal hash, successor prefix hash, the hash through the suspension, and the recovery records. Unpublished → `RECOVERY_PREPARING`; anything missing, inconsistent or unverifiable → `RECOVERY_INCONSISTENT`; no side effect in either case.
  - **Idempotent reruns:** a rerun finds the same successor by id, verifies, and finishes only the missing steps. It never creates a second successor, never re-appends `JOURNAL_RECOVERED` or the suspension, never overwrites a descriptor, seal or marker, never writes the predecessor journal and never applies `continue`.
  - **Inconsistency:** never repaired automatically. The human restores evidence, sets aside an unpublished and unsealed staging successor (rename, no deletion), or abandons.
  - **Human continuation:** `decide continue` must quote the bound predecessor hash and successor id. Only then do Git reconciliation and the workflow continue. Lost decisions, routing choices and attestations are still never reconstructed.
- **Non-claim:** this is not a distributed lock and gives no distributed-system guarantee. No distributed locking infrastructure is introduced. Runtime state on shared/synced storage is unsupported.

**Alternatives.**
- OS file locks: inconsistent across platforms, not fenced.
- A daemon as lock owner (ADR-001).
- Expiry-only takeover (r2): unsafe for paused processes.
- Distributed lock services: out of scope for a local tool.
- (r4) Logical skipping of a torn tail without truncation (r3): the next append makes it middle corruption.
- (r4) Rewriting the journal to a new file and renaming it over the old one: a larger mutation and a larger crash surface than truncating one suffix, and it would rewrite acknowledged records.
- (r4) Appending a "skip marker" after the damaged bytes: it leaves an invalid non-final line, which is indistinguishable from real middle corruption.
- (r5) Automatic truncation when `state.json` does not prove acknowledgement (r4): it can discard an acknowledged event, because `state.json` may be missing or stale.
- (r5) A durable acknowledgement watermark or commit protocol (e.g. a separately fsync'd ack file, or a two-phase record): it could make interrupted appends safely repairable, but it adds a second authoritative store and new crash windows. Declined by the owner for D1.
- (r5) Successor run vs in-place human truncation: the successor keeps the original journal as untouched evidence, and the human decision is journaled in the new run.
- (r6) Discovering an incomplete successor only by `JOURNAL_RECOVERED` (r5): a crash before that record leaves an undiscoverable, resumable run.
- (r6) Building the whole successor journal in one temp file and renaming it into place: it removes partial journals, but the successor directory would still need its own visibility boundary and provenance. The staged directory and descriptor give both, and they reuse the existing lease and append primitives.
- (r6) A general recovery transaction log or a new `RECOVERING` run status: out of proportion for one command. The staging area plus a read-only gate is enough, and the reducer is unaffected.
- (r6) A publication flag inside the successor's journal: the journal could then be resumed before the flag was written, which is exactly the r5 defect.
- (r7) A transaction manager or a recovery-specific lock variant to fix the P3 ordering: declined by the owner. Splitting the existing acquisition into its two natural halves fixes the ordering for every caller with the existing primitives.
- (r7) Appending `LEASE_ACQUIRED` at lock time and installing the prefix afterwards in front of it: it would rewrite the journal, which the append-only rule forbids.
- (r7) A special empty-prefix successor format for `B = 0` (e.g. a first record marked as a recovery root): declined by the owner. It would weaken the structural first-record classification, and with no valid record there is nothing for a successor to carry over.

**Consequences.**
- \+ On the automatic path a second writer cannot exist, because the previous owner is verified dead. Fencing and the replay rule are defence in depth.
- \+ On the human-forced path, a stale writer's later events are excluded on replay and its Git mutations are detected by reconciliation (detection, not prevention).
- − A hung but alive orchestrator, or an unverifiable owner, needs a human.
- − Depends on reliable process start-time queries and exclusive publish on Windows (D0.5 SP-W4/W6/W9).
- \+ (r4/r5) A crash during an append can no longer poison later appends.
- \+ (r5) No journal byte is ever removed or rewritten, so no acknowledged event can be lost through ai-flow's own recovery. No truncate primitive is needed.
- − (r5) **Every** damaged final record needs a human, including a harmless interrupted append after power loss. SP-W9 measures how often a process kill alone produces one.
- − (r5) After a successor-run recovery, the effects of the possibly lost record are recovered only where Git can show them (verified case-2 adoption). A lost human decision, routing choice, suspension or attestation must be re-made, and its usage/escalation effects are reported as unknown.
- \+ (r6) No ordinary command can resume or reconcile a successor before its recovery records, suspension, predecessor seal and marker are durable. A crash anywhere in `recover` is finished by a rerun.
- − (r6) Every mutating command hashes the predecessor journal while a recovery lineage exists. Journals are small (payloads live in attempt files), so this is acceptable. Later damage to the sealed predecessor's journal blocks its successor (`RECOVERY_INCONSISTENT`) until the evidence is restored. That is fail-closed by design.
- − (r6) The visibility boundary depends on an atomic, durable same-volume directory rename (SP-W5/W8/W9). On Windows a rename blocked by AV/indexer handles leaves the successor unpublished until a rerun succeeds.
- \+ (r7) No `LEASE_ACQUIRED` is ever journaled for an acquisition whose checks failed, and in recovery it always follows the verified prefix. Every acquisition path has one ordering.
- − (r7) A run whose journal has no complete valid record cannot be recovered by ai-flow; it needs an independently verified copy, or it is abandoned.

**Status / Confidence.** Accepted / Medium.

---

## ADR-021 — Runtime-state and worktree locations; repo-instance identity

**Context.** In-repo runtime state and worktrees confuse tools, pollute repos and complicate multiple clones.

**Decision.**
- Authoritative runtime state: `~/.ai-flow/runs/<repo-instance-id>/<run-id>/`.
- Worktrees: `<repo-parent>/.ai-flow-worktrees/<repo8>/<run8>/` (configurable root, same volume preferred, short segments for Windows path limits).
- **Repo-instance id:** a random UUID in `<git-common-dir>/ai-flow/instance-id`. It is not committed, it is shared by all worktrees of a clone, and it is registered with path and remote in `~/.ai-flow/repos/`. A path mismatch (copied/moved clone) → `WAITING_HUMAN: REPO_IDENTITY`.
- (r6) Recovery successors are staged in `~/.ai-flow/runs/<repo-instance-id>/.recovering/`, on the same volume as the run namespace, so publication is one directory rename (ADR-020, §12.1.2). Ordinary run discovery never treats a staging entry as a run.

**Alternatives.**
- Identity from the remote URL: multiple clones collide.
- Identity from the path: breaks on move.
- In-repo state (r1): rejected.

**Consequences.**
- \+ Clean repos; clones are distinguished.
- − Run data is not co-located with the project (`report --export` [SOON]).

**Status / Confidence.** Accepted / Medium.

---

## ADR-022 — Side-effect classes, effect locations and retry safety; no destructive cleanup without safe primitives (r3, r4, r5, r6)

**Context (r3).** r2 described `workspace_local` effects as "reconcilable by Git reset", including gitignored caches. That is false: `git reset --hard` and `git clean -fd` do not remove ignored files. A retried attempt could therefore silently inherit an interrupted attempt's caches, local databases or generated files, or ai-flow could be tempted to delete ignored files that include secrets.

**Decision.** Validators, agent steps and future actions are classified as:

- `pure`: automatically retryable.
- `workspace_local`: retryable after Git reconciliation **and** the ignored-state check.
- `external_side_effecting`: never retried automatically without an idempotency key, a declared-safe operation, or human recovery.

**Effect locations** (§13.2):

| Location | Restored by Git? | Retry |
|---|---|---|
| `tracked_worktree` | yes (`reset --hard`) | automatic |
| `untracked_nonignored` | yes (`clean -fd`) | automatic |
| `ignored_local` | **no** | via policy only (below) |
| `external_side_effecting` | no | per the effect-class rule |

**Ignored local effects** require one of:

- a declared-safe / idempotent regeneration (`restore: bootstrap`);
- an explicit, scoped cleanup (`cleanup: delete`, only declared paths inside the worktree; config rejects globs that hit secret patterns, `.git/**` or outside paths). r5: it executes **only** on a platform with proven race-resistant deletion, and **never on Windows in D1** (below);
- worktree recreation with safe preservation (the old worktree is quarantined with verified identity, never deleted; only declared `provision`/`preserve` paths are carried);
- human recovery (`WAITING_HUMAN: IGNORED_STATE_UNSAFE`) when safety cannot be established. This is the default for undeclared, `stateful` or `unverifiable` changes.

`protected` ignored paths (secrets) are never modified or deleted by ai-flow. `git clean -x/-X` is never used. Detection uses an ignored-state inventory taken at the start of each writing attempt (best effort; uncertainty ⇒ suspend).

**Cleanup safety (r4 validation; r5 platform restriction, §13.2.1).** The third review found that "inside the worktree" was checked only lexically and that overlapping policies were not rejected. r4 added no-follow checks but still deleted by path. The fourth review found that path-based checks followed by a path-based delete cannot prevent an ancestor being swapped for a junction/symlink in between, and that the r4 post-deletion re-check only detected an escape after external data was gone. Decision [OWNER]:

- **Precedence (kept):** `.git` / secret patterns / `protected` > `stateful` > `provision`/`preserve` targets > `derived` + `restore: bootstrap` > `derived` + `cleanup: delete`. Undeclared ignored paths are never deleted. Only the last class can ever be deleted by ai-flow.
- **Config load (kept, every platform):** rejects a cleanup rule that is absolute, escaping, non-normalised (`..`, `.`, empty segments, backslash, ADS `:`, trailing dot/space), has no literal root directory, explicitly targets secret patterns, or **overlaps** any rank 1–4 policy (ancestor/descendant/equal roots or a possible glob intersection; undecidable = overlap). Overlaps are rejected, not resolved.
- **Windows (D1):** **no path-based destructive cleanup** of ignored files or directories, whatever D0.5 finds. No native helper is required or built.
- **Other platforms (D1):** the same, unless SP-W8 empirically proves a no-follow, **race-resistant** mechanism usable without a native helper: deletion relative to a held, verified directory handle (`openat`/`unlinkat` semantics). Node 22 path-based `fs` APIs do not qualify. Expected D1 outcome: disabled everywhere.
- **Where cleanup is disabled:** declared cleanup paths are fingerprinted read-only like undeclared paths. Unchanged → retry. Changed or unverifiable → `IGNORED_STATE_UNSAFE` (`NO_SAFE_DELETE_PRIMITIVES`) with zero destructive operations, or whole-worktree quarantine/recreation if the project explicitly configured `on_cleanup_unavailable: recreate_worktree`. **No fallback to deletion** of any kind (recursive delete, `git clean -x/-X`, `git worktree remove`, copy-then-delete).
- **On a proven platform:** worktree identity → handle-relative, no-follow descent and inventory (any link stops) → per-candidate revalidation → bottom-up, non-recursive, handle-relative deletion. Validation of all rules completes before the first deletion. Any failure deletes nothing further and suspends.
- **Post-deletion detection is not a safety guarantee** and is not relied on anywhere.
- **Quarantine/recreation:**
  - verify the original worktree (real directory, `WORKTREE_CREATED` identity, realpath) **before** the rename;
  - one same-volume rename into `.quarantine/`, with no copy fallback, no delete and no recursion;
  - verify the quarantined identity and the absence of the original path **after** the rename;
  - update only Git's administrative entry (never `git worktree remove`);
  - re-verify **before** creating the replacement.

  Any failure → `IGNORED_STATE_UNSAFE` (`QUARANTINE_UNVERIFIED`), **no replacement worktree**, nothing deleted. The quarantined tree keeps all of its content until a human removes it. A path-based rename that is redirected by a race destroys nothing and is caught by the after-rename identity check. That detection is acceptable only because a rename is non-destructive.
- **`cancel --cleanup` (r6):** quarantine-first, never deletion. The worktree is quarantined with the same identity-verified rename (steps 1–4, no replacement), on Windows in D1 and on every other platform alike. There is no recursive delete, no `git worktree remove` and no `git clean`. An unverified worktree (identity mismatch, link/reparse point, missing, `unverifiable`, rename failure) is left in place and reported as `CLEANUP_SKIPPED_UNVERIFIED`. The run is still `CANCELLED`, and the branch is never deleted.

D1 rules:

- D1 agent permission profiles **prohibit external writes**.
- Agent steps are auto-retried only if that prohibition is enforced at `native`/`os_isolated` level **and** the ignored-state check passes; otherwise the project must explicitly set `retry_with_unverified_isolation: true` (recorded), and the ignored-state check still applies.
- D1 validation accepts only `pure`/`workspace_local` checks.

**Consequences.**
- \+ Retries never duplicate external effects unknowingly.
- \+ Ignored local state cannot silently contaminate a retried attempt, and ai-flow never deletes ignored files or secrets indiscriminately.
- − External checks (e.g. integration tests against shared services) are deferred.
- − Projects must declare `ignored_local` to avoid suspensions on interrupted attempts; inventory cost on large ignored trees is a D0.5 question (SP-W8).
- \+ (r4) A misconfigured rule cannot delete secret, stateful, provisioned or preserved state because of a broad glob, since overlaps are rejected at load.
- \+ (r5) In D1 no ignored-file deletion can be redirected outside the worktree by a link swap: on Windows and on unproven platforms none is attempted, and on a proven platform deletion is handle-relative.
- − (r5) `cleanup: delete` is effectively inert on Windows in D1. An interrupted attempt that touched declared cleanup paths suspends, or uses the configured quarantine/recreation, which costs disk until a human removes the quarantined tree.

**Alternatives (r4).**
- Recursive delete after a lexical containment check (r3): escapes through links.
- Resolving overlaps by precedence silently: hides misconfiguration that could delete user state.
- (r5) Path-based no-follow deletion with post-deletion detection (r4): it cannot prevent an ancestor swap, and detection comes after the external data is gone.
- A native helper with `openat`/`unlinkat` or Windows handle-relative deletion: closes the TOCTOU window but adds a native build dependency. Not required and not built in D1 [OWNER]; a candidate for after D1.
- A general filesystem-sandbox layer: out of proportion for a single cleanup operation.

**Status / Confidence.** Accepted / High.

---

## ADR-023 — Security enforcement levels

**Decision.**
- Every security property (fs write scope, reviewer read-only, network deny, shell policy, external writes, secret env, Git mutation) is reported with its effective level: `native`, `os_isolated`, `posthoc` or `instruction_only`.
- The levels are persisted per attempt and shown in reports.
- Minimum levels are enforced by refusing routing: a reviewer needs native fs read-only plus posthoc verification, unless the project explicitly accepts `posthoc`.
- ai-flow never claims to guarantee a property that is only enforced by a provider, detected posthoc, or merely instructed.

**Consequences.**
- \+ Honest security posture.
- \+ Auditors can see exactly what was enforced.
- − Some provider/role combinations will be refused.

**Status / Confidence.** Accepted / High.

---

## ADR-024 — One typed code workflow + pure reducer in D1; DSL deferred

**Context.** The reviewer warned against building a mini workflow platform. D1 needs exactly one workflow.

**Decision.**
- D1 implements **one** code workflow as a TypeScript transition table plus a pure reducer `reduce(state, event) → { state, effects }`, with full-path and fast-path variants.
- The table is serialised and hashed for snapshots.
- Seams (provider adapter, check runner, gate detector, context source) are plain interfaces with in-tree implementations. There is **no registry, no dynamic loading, no DSL**.
- A declarative format may be extracted later, once three or more real workflows exist (rule of three).

**Alternatives.** The YAML DSL of r1 (ADR-012, superseded); workflows as arbitrary code without a reducer (less testable).

**Consequences.**
- \+ Small, exhaustively testable core.
- \+ Replay = fold.
- − New workflows need code changes (acceptable now).

**Status / Confidence.** Accepted / High.

---

## ADR-025 — Model & Effort Router: separate decisions, abstract profiles, per-provider mapping

**Context.** Token/quota efficiency is a first-class owner requirement. Always using the strongest model or highest effort wastes quota. Provider, model and effort are different decisions.

**Decision.**
- Routing makes three decisions:
  1. **provider** per role (routing epoch, independence);
  2. **abstract profile** per step (FAST/STANDARD/STRONG/CRITICAL), from the step class, floors and per-lineage escalation;
  3. **model + effort + options** from the provider's profile mapping in `~/.ai-flow/providers/<id>.yaml`.
- Model names appear **only** in provider mappings.
- The adapter validates mappings. Unsupported profile/model/effort **fails clearly**, with no silent substitution.
- **Model-floor check (r3):** a provider-reported model that is neither the requested model nor a declared `accepted_resolutions` entry cannot be verified against the floor. This **fails closed**: the result is not accepted and the run suspends with `MODEL_UNVERIFIED`. A surface that does not report the served model at all is recorded as "request-verified only" (a D0.5 capability fact).
- Bundle size gates mapping eligibility by context window.
- The router is a **pure function** over the frozen config. Every decision is journaled with its inputs hash, floor sources, rung and reason.
- **No ML/learned routing in D1.** Historical metadata supports later human policy changes.

**Alternatives.**
- A single "level" axis (r1 FAST/STANDARD/HIGH) conflates model and effort.
- Cost-optimising routing from day one: no reliable data.

**Consequences.**
- \+ Cheapest configuration above a floor; auditable.
- − Static "cheapness" ordering until evals exist.

**Status / Confidence.** Accepted / Medium.

---

## ADR-026 — Risk floors and bounded, per-lineage escalation

**Decision.**
- **Effective floor** = max(project minimum, task minimum, risk-class floor, step-class floor, detector floors). Effective risk can only be **raised** by deterministic detectors.
- **Escalation** follows configured ladders of `(profile, effort_step)` rungs, keyed by **lineage** (no leakage to unrelated steps). It is:
  - monotonic and acyclic;
  - bounded per lineage and per run, with a ceiling;
  - CRITICAL only if permitted or decided by a human;
  - triggered only by deterministic events (never by rate limits or quota);
  - journaled.
- Ladder exhaustion → `WAITING_HUMAN: ESCALATION_EXHAUSTED`.
- Default policy: effort-first when the same failure persists, model-first for output/tool failures or persistence at maximum effort. This is an **empirical, configurable heuristic, not an invariant**. The invariants are the bullets above (deterministic triggers, monotonic, bounded, CRITICAL gated, journaled).
- **Observed evidence, not diagnosis (r3):** triggers are named for what was observed (`same_failure_persists`, `output_or_tool_failure`, `persists_at_max_effort`) and journaled with `trigger_evidence`. A persistent test failure is recorded as evidence. It is not proof that the model lacked reasoning ability (the contract, context, test or environment may be at fault).

**Consequences.**
- \+ No unbounded "cheap → strong → critical → repeat" loops.
- \+ Expensive profiles do not leak.
- − The default heuristic may be suboptimal before evals.

**Status / Confidence.** Accepted / Medium.

---

## ADR-027 — Small-task fast path: opt-in, no self-certification (r3) [OWNER]

**Context.** Trivial, test-verifiable changes should not pay for planning plus a strong review. The second review found that r2 let the implementer add or edit the very tests that decide its ACs ("tests to be added in declared scope") and then skip independent review. That is self-certification.

**Decision.**
- **Explicit project opt-in:** `fast_path.enabled` defaults to `false`, and the `init` template ships it disabled (r3; r2 enabled it in the template).
- Flow: `IMPLEMENT (FAST) → staged VALIDATE → FAST_POSTCHECK → DONE`, or a single light review if the project requires one.
- **Trusted oracles (r3):**
  - The oracle set is resolved at `base_sha`: AC-mapped test files plus the declared `fast_path.oracle_paths` (helpers, fixtures/snapshots, runner config, manifests).
  - Every AC must be `verify: test` with mapped tests that **already exist at `base_sha`**. New ACs and new tests ⇒ ineligible pre-run (full review).
  - Any add/modify/delete/rename of an oracle file, or any new file under the test globs ⇒ `FAST_POSTCHECK` ineligible ⇒ promotion to full review.
  - `FAST_POSTCHECK` verifies oracle blob hashes against `base_sha`, an unchanged selector resolution, and collected test ids ⊇ base with none skipped. So ACs are decided by the trusted base versions.
- Eligibility is deterministic:
  - **pre-run:** LOW effective risk; all ACs test-verified by pre-existing oracles; small declared scope; no protected/gate paths; floor ≤ FAST;
  - **post-diff:** no oracle change; file/line limits; no gates; no dependency/protected changes; all ACs MET.
- **Thresholds** (3 files / 60 lines / 10 scope files) are configurable starting values, not architectural guarantees.
- A failing post-check **promotes** the run to the full review path. There is never a demotion.
- No PLAN, Spec Kit or STRONG/CRITICAL on the fast path.

**Consequences.**
- \+ Large savings on trivial work, without the implementer certifying itself.
- − Tasks that need new tests never use the fast path.
- − Residual risk: behaviour that the trusted tests do not exercise (including test-environment special-casing). This is mitigated by small-diff limits and the optional light review, not eliminated.

**Status / Confidence.** Accepted / Medium.

---

## ADR-028 — D1 scope: provider-neutral local orchestrator for Git-based projects

**Context.** r1 implied a universal project/workspace abstraction.

**Decision.** D1 is described and built as **a provider-neutral local orchestrator for Git-based projects**. Content, research and non-Git workflows are future use cases. They inform the seams (§18), but no abstraction for them is built in D1.

**Consequences.**
- \+ Honest scope and less speculative code.
- − Non-Git use cases will need an extension later.

**Status / Confidence.** Accepted / High.
