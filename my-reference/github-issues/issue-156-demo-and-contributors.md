# Issue #156 — Demo and contributors: prepare one repeatable scene and public invitation

- **Issue:** https://github.com/hexanomicon/lychd/issues/156
- **State:** open (snapshot 2026-11-16) · **Labels:** `area:devex`
- **Assignee:** `geeked-anshuk666` (you) — **primary**, coordinates the first scene and reviewable packet
- **Partner:** `NavSrijan` — capture/visual material, audience clarity, media experiments
- **Acceptance/reproduction support:** `morichika33` (Debayan), linked to #133
- **Product view (when used):** Chirayu, Kiran
- **Editorial/release decision:** founder `zenithruneblade`
- **Linked:** #52 (Start here, public contributor entry), #133 (Altar Core Bridge), #62 (future capability register), #162 (Atelier side quest, Nav + you)

---

## 1. What the issue asks for (plain English)

Prepare **one** short, honest demo and a **public invitation** for new contributors, built from **real, working evidence**. The demo is a packet:

- a scene with a clear user purpose, starting action, and observable result;
- a reproduction record (source revision, steps, expected observation, limits);
- a short script or storyboard plus one reviewable capture;
- a contributor invitation that links to #52 and to the bounded opportunity from #133;
- a check that another person can follow the scene and the newcomer entry steps.

This is **not** a product launch and **not** a claim that native Composition delivery exists. Mockups and external tools are labelled as such.

**Done means:** the packet and an accepted entry route are complete, and the founder has reviewed it. *Publishing* the video is a separate, deliberate action with its own record.

---

## 2. Prerequisites — what to know first

### Tech stack (light, mostly communication)
| Area | What to know | Used for |
|---|---|---|
| Git + GitHub | Branches, PRs, issue links | The packet and the invitation live in issues/PRs |
| Basic screen capture | Recording a short clip, cropping, captions | The reviewable capture |
| Markdown | Headings, checklists, links | All docs here |
| Browser + Altar | Opening the local web UI (`clients/web`, SvelteKit) | Likely the scene's surface |
| Local LychD run | `make install`, `make frontend-install`, `make frontend-build` | Reproducing the scene |
| Accessibility basics | Captions, text alternatives, readable contrast | Required by the issue |
| Privacy hygiene | Scrubbing credentials and private chat from captures | Required by the issue |

You do **not** need deep backend knowledge to finish #156. You do need to understand what the scene actually does, so you don't overclaim it.

### Architecture you need to understand (at the scene level)
1. [CONTRIBUTING.md — Find the part you want to change](../../CONTRIBUTING.md#find-the-part-you-want-to-change) and the [Fullstack lane](../../CONTRIBUTING.md#start-with-three-development-lanes): *generated API contract → retained evidence → accessible instrument*.
2. [docs/state-of-the-work.md — Current evidence envelope](../../docs/state-of-the-work.md#current-evidence-envelope) — **this decides what you're allowed to claim.** Read the "Outcome matrix" too.
3. [docs/state-of-the-work.md — Altar workbench and Bridge](../../docs/state-of-the-work.md#altar-workbench) and [Bridge conversation and consent surface](../../docs/state-of-the-work.md#bridge-surface) — the likely scene.
4. [docs/state-of-the-work.md — Local text chat](../../docs/state-of-the-work.md#vllm-integration) — the Outcome matrix says general answer quality and wider browser acceptance remain pending. **Don't promise answer quality in the demo.**
5. [docs/adr/15-frontend.md](../../docs/adr/15-frontend.md) and [clients/web/src/README.md](../../clients/web/src/README.md) — the Altar source map.
6. Your reference: `my-reference/07_ALTAR_AND_CLIENTS.md` (Bridge, Loom, Nexus, Orb) and `my-reference/08_COMPOSITIONS_AND_USE_CASES.md`.
7. [docs/summoning.md](../../docs/summoning.md) — read the warning. It explains why the full stack should not be presented as a turnkey install.

### Concepts
- **Scene** — one repeatable sequence: start point → action → visible result.
- **Fixture / prototype / dated observation / unfinished** — the four labels the issue asks you to use. Every claim gets one.
- **Synthetic input** — fake prompts and fake data. Never a real person's conversation or key.
- **Newcomer entry** — the path from #52 to a first contribution, which must work without private-board access or a Discord account.

---

## 3. Do you need to summon a Lich for this issue?

**No.** You need a *reproducible local scene*, not a full installation.

- The issue says: "Reuse current repeatable engineering work and existing authoring, capture and editing tools." and "a mockup or external tool" does not count as native delivery.
- Summoning (see [docs/summoning.md](../../docs/summoning.md)) requires Linux, NVIDIA, a GGUF model and rootless Podman. It is not end to end yet, and Windows is unsupported for it.
- Do the scene with the **Altar in dev mode** and a **synthetic** request, or with a recorded **dated observation** — and label which one it is.

If the scene needs a live model, use a fixture or say clearly that the model response is not evidence of answer quality.

---

## 4. How to proceed — step by step

### Step 0 — Set up the frontend (once)
```bash
cd D:\computer_science\Snayu_AI\lychd
make install
make frontend-install      # npm ci from the web-client lock
make frontend-check        # type-check and test the Altar
make frontend-build
```
Look at the target list with `make help`. If a command fails, record it; don't skip it silently.

### Step 1 — Choose one scene
Criteria (from the issue):
- one clear **user purpose** (e.g., "ask a question and see the Run trace");
- one **starting action** (e.g., open Bridge, type a prompt);
- one **observable result** (e.g., a settled reply, or a visible consent prompt);
- reuses **existing** tools — no new surface.

Good candidates: the Bridge conversation with a synthetic prompt, or the Run inspection path (Orb/Loom) on an existing Run. Agree the choice with Nav and Debayan.

### Step 2 — Write the reproduction record
Record in `my-reference/github-issues/` or the issue (keep private material **outside** the repo):
- source revision (`git rev-parse HEAD`) and profile;
- exact steps, with the starting state;
- the expected observation;
- the limits (what the scene does **not** prove);
- label: fixture / prototype / branch experiment / dated evidence.

### Step 3 — Script and capture
1. Write a short script or storyboard (3–5 beats). Each beat = one thing on screen.
2. Capture one reviewable clip or image sequence. Use **synthetic** inputs.
3. **Inspect every frame for credentials, tokens, usernames, private conversations, and local paths.** Blur or re-record if any appear.
4. Add captions and a text alternative for every beat.
5. State what the viewer can learn **and** what is unfinished. Don't imply a finished product.

### Step 4 — Draft the contributor invitation
Include:
- a link to #52 (public contributor entry);
- the bounded opportunity prepared under #133 (ask Debayan for the exact wording);
- a **specific ask**: a reproduction report, a usability observation, or one contribution;
- where to reply and who accepts contributions;
- an explanation of the purpose of the demo.

The invitation **must work without private-board access or a Discord account.** Test that claim by following it as a stranger would.

### Step 5 — Route the Atelier and Broadcast boundary
- Keep **Atelier** visual creation distinct from **Broadcast** editing/publication.
- If the scene reveals a useful native experiment, record it for later selection in #62. **Do not** build Atelier or Broadcast here — that is #162.
- YouTube as a publishing destination does **not** need a native YouTube Reach adapter.

### Step 6 — Verify with a second person
- Have another participant (Nav or Debayan) **follow the scene and the newcomer steps from the invitation**, with no help from you.
- Note every place they stop or misread. Fix the instructions, or reduce the claim.

### Step 7 — Submit the packet for founder review
Packet contents:
1. the scene and its reproduction record;
2. the script/storyboard and the capture;
3. the invitation draft and its destination (where it would be posted);
4. the verification note from Step 6;
5. the list of what is unfinished.

Send to the founder (`zenithruneblade`) for editorial review. Publication is a **separate** step.

---

## 5. Capacity and scope (don't overreach)

- Keep **one** small demo packet active. The issue says this explicitly.
- Don't assume external contributor capacity or video quota.
- Don't build a new media platform integration.
- Production, review, and community follow-up count against your ordinary capacity.

---

## 6. Things that could go wrong

- **Overclaiming** — "the demo shows the agent answering well" when the Outcome matrix says answer quality is pending. Reduce the claim.
- **Leaking private data** — a visible token in a terminal or a real chat in the browser. Inspect frames.
- **Non-reproducible scene** — it works once on your machine. Test it on a clean checkout with Nav or Debayan.
- **Invitation that needs a Discord account** — violates the acceptance text. Test the route yourself.

---

## 7. Checklist

- [ ] Revision and profile recorded
- [ ] One scene chosen (purpose, starting action, observable result)
- [ ] Reproduction record written; each claim labelled
- [ ] Script/storyboard drafted (3–5 beats)
- [ ] One reviewable capture made with synthetic input
- [ ] Every frame inspected for credentials and private data
- [ ] Captions and text alternatives added
- [ ] Contributor invitation drafted; links to #52 and the #133 opportunity
- [ ] Invitation works without private board or Discord (tested)
- [ ] Atelier/Broadcast boundary kept; any experiment recorded in #62
- [ ] Second person followed the scene and entry steps; findings fixed
- [ ] Packet submitted to the founder for editorial review
