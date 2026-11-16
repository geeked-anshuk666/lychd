# Discrepancy and gap audit — LychD issues, PRs and my-reference docs

- **Snapshot:** 2026-11-16
- **Compared:** `ISSUES_REPORT.md`, `open_issues.json`, `closed_issues.json`, `open_pull_requests.json`, live GitHub search (`repo:Hexanomicon/Lychd is:issue is:open assignee:geeked-anshuk666`), `my-reference/*.md`, and the repo's own `docs/` and `CONTRIBUTING.md`.

---

## A. What matches (verified)

| Claim | Source | Result |
|---|---|---|
| 40 open issues | ISSUES_REPORT, README | `open_issues.json` has 40. Match. |
| Open-issue table rows and labels | ISSUES_REPORT vs JSON | All 40 numbers present in both; no extra rows. Match. |
| 70 closed issues | ISSUES_REPORT, README | `closed_issues.json` has 70, and the report table has 70 matching numbers. Match. |
| 5 open PRs | ISSUES_REPORT, README | `open_pull_requests.json` has 5 (#150, #167, #168, #169, #170). Match. |
| `creation.py` and its vertical test exist | Issue #152 body | Both present in the checkout. |

---

## B. Discrepancies (things that are wrong or stale)

| # | Where | What it says | What is true | Severity |
|---|---|---|---|---|
| B1 | `my-reference/04_PERSISTENCE_AND_MEMORY.md` §1 | Migrations `0001` through `0023` | The repo has 24 numbered files (`0001`–`0027`, with gaps at 0014, 0019, 0020) plus 3 dated migrations. | Medium — misleads anyone reading the schema history. |
| B2 | `my-reference/README.md` header | Location `d:/computer_science/Snayu_AI/my-reference/` | The folder is at `D:\computer_science\Snayu_AI\lychd\my-reference\`. | Low — stale path. |
| B3 | `my-reference/README.md` table-of-contents descriptions of 01 ("What is possible") and 10 ("100 Practical, Mind-Blowing & Revenue-Generating Applications") | Broad capability and revenue framing | State of Work marks remote transport, federation, executable evolution, durable recall, vision/audio bytes, native Oculus, and resource-aware scheduling as **Partial or Designed**. The README has no caveat. *Note: I read the README descriptions, not the full text of docs 01 and 10; verify the body text before editing.* | High for an onboarding reader if the bodies match the descriptions. |
| B4 | `my-reference/` (general) | Describes Run statuses, the Phylactery and the Litestar/PostgreSQL path as working | Repo docs say PostgreSQL and SAQ are **not one transaction**, Step events have no outbox, memory/PostgreSQL parity is incomplete, and no durable watchdog exists for same-boot containment failure. | High — the reference should carry these limits. |
| B5 | `my-reference/` vs `docs/summoning.md` | (Implicit) a full install path exists | `docs/summoning.md` says **summoning does not work end to end yet**, and Windows is unsupported for the deployment path. The my-reference docs I read do not say this. | High for anyone trying to summon from Windows. |
| B6 | `ISSUES_REPORT.md` | Tabular catalogue of open issues | No **assignee** column, no **sub-issue progress**, no **blocked-by/blocking** info, no **bodies**. You cannot tell from the report that #131, #152, #156 are yours. | Medium — the report can't answer "what is assigned to me". |
| B7 | `ISSUES_REPORT.md` | Open PRs list | No link between a PR and the issue it serves. For example, PR #150 (Nav's Reach model binding) is referenced in #131's body but not in the report. | Low–medium. |
| B8 | `my-reference/README.md` (Upstream section) | "Snapshot of all current and historical issues" | The snapshot is point-in-time (2026-11-16) and will go stale. It says nothing about how to refresh it. | Low. |

---

## C. Left out (gaps in the reference set)

| # | Gap | Why it matters |
|---|---|---|
| C1 | No doc maps **issue → area → owner** for the 40 open issues | Needed to answer "who owns what" and "what do I do first". The new per-issue docs for #131, #152, #156 cover your three; the other 37 are not mapped. |
| C2 | No doc explains the **three development lanes** (Agentic, Fullstack, Backend) from CONTRIBUTING | These lanes tell you which focused `make test` target to run for each change. |
| C3 | No doc lists the **focused test commands** | Without them, a newcomer runs the whole suite inside OCI and gets lost. |
| C4 | No doc records the **"evidence envelope"** (what counts as proven vs. fixture vs. dated observation) | This is the rule every issue in the tracker uses. Without it, claims in issues look stronger than they are. |
| C5 | No doc covers **Windows as a development host** | You are on Windows 11. The repo's supported host policy says Linux for hosting; development needs the Podman/Docker OCI path. That split is not written anywhere in my-reference. |
| C6 | No doc maps **PR #150 and PR #151** (merged Tether) to their issues | Both are referenced by open issues but not explained. |

---

## D. Internal inconsistencies to resolve (not errors yet)

| # | Item | Note |
|---|---|---|
| D1 | `0004_run_delivery_outbox.py` exists, but State of Work says "Step events lack an outbox" | These may refer to different tables (Run delivery vs Step events). Read both before claiming either. Flagged in the #152 doc. |
| D2 | Issue #131 and issue #152 both name Nav and Anshuk as owners of overlapping restart/Nexus work | Intended split per the bodies: Anshuk = PostgreSQL restart; Nav = execution trace. Confirm with the team that this is the agreed split. |
| D3 | The Makefile defines `test-ci` twice (around lines 332 and 362) | The later definition probably wins in GNU make. Worth a cleanup PR, but not a blocker. |

---

## E. Summoning — should you summon a Lich for your issues?

| Issue | Answer | Why |
|---|---|---|
| #156 Demo | **No** | You need a reproducible local scene and captures, not a full host install. |
| #152 Runtime / PostgreSQL | **No, not for the first pass** | Use the focused PostgreSQL restart tests and a local DB. Summon only when a supported profile needs real hardware. |
| #131 Core backend | **No, not for the first pass** | Same as #152. Nexus and the Discord path can be exercised with synthetic requests. |

Reasons (from the repo, not guesses): [docs/summoning.md](../../docs/summoning.md) says the full sequence is not end to end; it needs Linux, rootless Podman ≥ 5.4, Quadlet, NVIDIA CDI, a GGUF model and `localhost/lychd:dev`; and Windows is unsupported for the deployment path. You are on Windows 11.

---

## F. Recommended fixes (for the team, not done here)

1. Correct B1 (migration range) and B2 (path) in `my-reference`.
2. Add a "State of Work limits" banner to `my-reference/README.md` and to the 01 and 10 pitch docs (B3, B4, B5).
3. Add assignee and sub-issue columns to `ISSUES_REPORT.md`, or a generator script (B6).
4. Link each open PR to its issue (B7).
5. Add a short doc for C2–C5 (lanes, focused tests, evidence envelope, Windows development).
6. Resolve D1 by reading `0004_run_delivery_outbox.py` and the outbox sections of State of Work.

---

## G. Method and limits

- The GitHub MCP `list_issues` call returned about 130 KB, which exceeds the response limit. I did not read that output in full. Instead I used the local `open_issues.json` snapshot (40 items, verified complete) and a targeted search for your assignments.
- The three assigned issue bodies come from the live search results (`geeked-anshuk666` assignee filter), which returned the full body text.
- I did not read every line of `closed_issues.json` or the full text of all 40 open issue bodies. Checks on the other 37 open issues were limited to titles, labels, and dependency counts.
- Test commands in the per-issue docs were **not executed** in this session. They are taken from `CONTRIBUTING.md` and the Makefile.
