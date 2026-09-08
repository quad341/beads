# Release gate — Docs: init-safety re-clone gotchas (be-wv0d0)

- **Builder bead (CLOSED):** be-wv0d0 — twin of gascity `ga-fca59j`
  (investigator handoff from the 2026-09-08 gm store incident `ga-vrq5pu`).
  Pure docs: the init-safety re-clone playbook must document two gotchas hit
  live during a manual re-clone recovery.
- **Deploy bead:** be-guyc7
- **Review bead:** be-uxr4j — verdict **PASS** (beads/reviewer, 2026-09-08),
  `tdd_red` `27b41b8a3de6c1988227f99b3a9b7a4983b23657`, `tdd_green`
  `5cb87611f6e8aeb4815b519fe77bcf96b6cd3284`, evidence
  `c0d8da42d..5cb87611f` — 3 files, +203/-2, zero open findings.
- **Commit:** `005315355f3de27b533a57055f21588a528287e6` — deploy branch tip.
  This supersedes the bead's originally-recorded commit
  (`5cb87611f6e8aeb4815b519fe77bcf96b6cd3284`, the builder-branch tip at
  review time) because the deploy branch rebases the same reviewed red/green
  pair onto the current `origin/main` tip, per deploy-branch-freshness
  discipline and the mayor's own note anticipating this rebase (see below).
- **Branch:** `builder/be-wv0d0` (provenance only; pushed to `headfork`,
  unchanged, still points at `5cb87611f`); deploy branch
  `deploy/be-guyc7-gate` cut from `origin/main` with the reviewed red
  (`27b41b8a3` → rebased as `c4ee9e582`) and green (`5cb87611f` → rebased and
  amended as `005315355`) commits replayed on top, to be pushed to `headfork`
  (`quad341/beads-sec003-contrib`).
- **Evaluated:** 2026-09-08 by beads/deployer

## Scope

Docs-only fix to `docs/recovery/init-safety.md`'s re-clone playbook, adding a
new "re-clone-gotchas" section covering two live gotchas from `ga-vrq5pu`:

1. A damaged/set-aside Dolt store left **inside** `data_dir` makes every
   server start crash-loop with `root hash doesn't exist: <hash>` — must be
   moved outside instead.
2. A fresh clone lacks the dolt-ignored clone-local tables (`leases`,
   `wisps`, `events`, etc.); `bd list` fails with `table not found: leases`.
   Fix is `bd migrate schema` (no `--force`); "Schema already at v64" is the
   expected, reassuring output, not an error.

`cmd/bd/init_safety_help.go` gets a matching RE-CLONE GOTCHAS addendum in the
`bd help init-safety` Long text. `test/docsync/init_safety_recovery_test.go`
carries two regression tests. Diff scope, confirmed via
`git diff --stat origin/main HEAD`: 3 files, matching the reviewed scope
exactly, no unrelated files.

Builder bead `be-wv0d0`'s done-when criterion mentions "both init-safety
docs" (`docs/recovery/init-safety.md` and the generated
`docs/cli-reference/init-safety.md`). The review (be-uxr4j) already examined
and accepted why only the former is touched: the generated page is pinned to
a released tag (`docs/cli-docs.pin = v1.2.2`) and doesn't pick up a
Long-field edit at HEAD until a maintainer bumps the pin — confirmed
empirically by the reviewer (`git show v1.2.2:cmd/bd/init_safety_help.go`
still points at the deprecated path; a full
`./scripts/generate-cli-docs.sh` run produced zero diff). The fix correctly
targets the underlying Go help-text source instead. Not re-litigated here.

### Rebase artifact found and fixed on the deploy branch

Rebasing the reviewed red/green pair onto the current `origin/main` tip
surfaced a real defect, independently caught before trusting the branch:
`docs/recovery/init-safety.md` carried **two** `Last reviewed:` lines
(`2026-09-08` then `2026-09-07`) after the rebase.

Root-caused, not just patched over: the builder's own green commit cleanly
replaced `Last reviewed: 2026-06-09` → `2026-09-07` (a normal single-line
freshness bump, "Freshness markers bumped" per its own commit message).
Independently, `origin/main` had since moved that same line to
`2026-09-08` via `2298c6517` (upstream #6411, merged after the builder's
commit was authored). Rebasing onto the new base couldn't recognize these as
the same logical line — the old `2026-06-09` anchor no longer existed in the
new base — so the rebase left both lines instead of cleanly replacing one.

Fix applied directly on this isolated, not-yet-pushed deploy branch (squarely
within deployer's "prepare a clean isolated branch" mandate — nothing shared
or pushed at the time): removed the stale `2026-09-07` line, keeping
`origin/main`'s already-fresher `2026-09-08`, then amended the green commit.
Verified clean afterward: `git diff origin/main HEAD -- docs/recovery/init-safety.md`
now shows only genuine feature content (no marker-line noise), and the full
3-file diffstat (202 insertions, 1 deletion) matches the reviewed diffstat
(203/-2) to the exact single line accounted for by this now-redundant marker
replace. Content verified equivalent to what be-uxr4j reviewed and PASSed.

### Doc-freshness gate (be-mrl6j) — mayor's anticipated caveat, now resolved

be-guyc7's notes carry an explicit mayor instruction (16:37Z): this branch
edits the same file as upstream #6385/#6411, both racing to refresh a
90-day-stale freshness marker that fails the repo-wide doc-freshness gate
(`be-mrl6j`) on every PR; if opened before one lands, disclose the expected
red check in the PR body so maintainers don't misread it as a defect.

#6411 has since landed (`2298c6517`, already in this branch's ancestry,
confirmed above), independently refreshing the same marker #6385 would have.
Independently verified rather than assumed: `TZ=UTC ./scripts/check-doc-freshness.sh`
run directly on this branch reports `docs/recovery/init-safety.md` as
`PASS: Last reviewed marker is current: 2026-09-08 (0 days old)`, full script
RC=0. The condition that would have triggered the red check no longer holds
on this branch — noted in the PR body for transparency, but not as an active
caveat.

## Gate criteria

| # | Criterion | Verdict | Evidence |
|---|-----------|---------|----------|
| 1 | Review PASS present | **PASS** | be-uxr4j records `VERDICT: PASS` (beads/reviewer, 2026-09-08) for `builder/be-wv0d0` @ `5cb87611f`, full exit-contract line-by-line, zero open findings. |
| 2 | Acceptance criteria met | **PASS** | be-wv0d0's done-when re-checked directly: both gotchas documented with symptoms, gascity-twin consistency confirmed by reviewer, no duplication of the automated corrupt-aside convention beyond a pointer; the "both docs" clause's generated-page exception already reviewed and accepted (see Scope). |
| 3 | Tests pass | **PASS** | 8/8 `./test/docsync/...` tests green on independent re-run on the corrected deploy-branch tip, including both new tests. See "Tests run" below. |
| 3b | Policy/lint lane | **PASS** (documented exception) | `make ci-pr-policy` FAILs on `.githooks/commit-msg` missing BEADS INTEGRATION markers; independently confirmed not a repo file (`git ls-files` fails to find it, excluded via `.git/info/exclude`, absent from `origin/main`'s tree entirely) and untouched by this diff. Pre-existing, environment-local, unrelated to this PR. |
| 4 | No unresolved HIGH findings | **PASS** | Zero findings of any severity across the single review round (be-uxr4j). |
| 5 | Clean working tree | **PASS** | `git status --short` on the amended tip shows nothing staged/unstaged. |
| 6 | Clean divergence from `origin/main` | **PASS** | `git merge-base --is-ancestor origin/main HEAD` succeeds; `git rev-list --left-right --count origin/main...HEAD` = `0  2` — 2 commits ahead, 0 behind, cleanly fast-forward-able. |
| 7 | Single feature theme | **PASS** | Exactly the 3 reviewed files (`cmd/bd/init_safety_help.go`, `docs/recovery/init-safety.md`, `test/docsync/init_safety_recovery_test.go`); no unrelated files. |

## Tests run on release branch (independent re-verification)

Static checks, independently re-run on the amended tip (`005315355`) rather
than trusted from the reviewer's report:

| Check | Result |
|---|---|
| `go build ./...` | clean, rc=0 |
| `go vet ./...` | clean, rc=0 |
| `gofmt -l` on the 2 changed Go files | clean, 0 files listed |
| `TZ=UTC ./scripts/check-doc-freshness.sh` | PASS, rc=0 (see above) |

Diff-owned tests:

| Test | Result |
|---|---|
| `TestInitSafetyRecoveryDocCoversReCloneGotchas` (new) | PASS |
| `TestInitSafetyCLIHelpCoversReCloneGotchas` (new) | PASS |
| `TestMintNavigationPagesExist` | PASS |
| `TestEveryDocsPageIsPublished` | PASS |
| `TestDocsSiteLinks` | PASS |
| `TestEngdocsAndRootMarkdownLinks` | PASS |
| `TestMintRedirectsResolve` | PASS |
| `TestDoltVersionPinsAgree` | PASS |

8/8 green (`go test ./test/docsync/...`, 0.012s) — matches the reviewer's
reported 8/8 exactly, re-run independently on the corrected, rebased tip.

## Findings from review (no action required)

Zero findings of any severity from be-uxr4j's single review round.

## Verdict

**PASS** — all 7 criteria pass (3b passes with a documented,
independently-verified pre-existing-and-unrelated exception; the rebase
artifact found while cutting this branch was root-caused and fixed directly,
with content re-verified equivalent to what was reviewed). Cutting isolated
deploy branch `deploy/be-guyc7-gate` from the amended tip
`005315355f3de27b533a57055f21588a528287e6`, pushing to `headfork`, and
opening a PR against `gastownhall/beads:main`. PR body will note the
now-resolved doc-freshness-gate situation mayor flagged, for transparency.

**gastownhall/beads merge-authority carve-out:** this is a contributor-only
repository (`origin` push disabled; upstream is fetch-only). Per deployer
protocol, the job ends at the open PR — no merge-request routed to
mayor/mpr, no deploy-clearance status posted. Merge belongs to upstream
maintainers.
