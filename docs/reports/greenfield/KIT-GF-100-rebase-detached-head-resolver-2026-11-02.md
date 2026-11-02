# KIT-GF-100: conflict resolver refuses an owned detached-HEAD rebase

Status: OPEN. No resolver fix is included in #2723. No fixed release is assigned.
This report also records the complete B rebase and its preservation checks.
No merge or release of B is authorized.

## Finding and isolated reproduction

The file resolver in cli_commands/transfer_repo_pre_pr.py calls
git branch --show-current and adds current_branch_missing for an empty result.
During a normal Git rebase, HEAD is detached even though rebase-merge/head-name
identifies the owned branch. The resolver therefore rejects a valid conflicted
file before copying or staging it. The general rebase route itself detects
the conflict and stops correctly; this finding concerns the file resolver.

An isolated local Git fixture created divergent edits to example.txt on main
and codex/rebase-probe, then rebased the latter onto main. With the conflict
present and branch --show-current empty, the actual command was:

```sh
agentic-kit transfer conflict-resolve-file example.txt --source resolved.txt --json
```

Actual result: return code 2, result_status BLOCKED, blockers
[current_branch_missing]. example.txt remained byte-identical and unmerged.
The adjacent JSON is the actual bounded command result. This is a negative
reproduction, not a PASS contract or a fix. The retained synthetic fixture has
no remote. No real workspace was downgraded or repaired by bypassing a gate.

A separate fix must validate the active rebase's owned branch and target before
accepting detached HEAD, preserve protected-branch/path guards, and add positive
and negative regression tests. That implementation is deliberately out of #2723.
The existing new-local-finding rule requires a machine contract. An added
negative-reproduction contract uses only the existing git_repository fixture
and runner; its 19 steps verify the BLOCKED result, detached HEAD, byte-identical
before/after conflict diff and still-unmerged index. Its PASS means only that
the OPEN failure is reproduced safely, never that resolution works or 100 is
fixed. The separate fix must replace/extend this with successful resolution and
invalid-state regressions; neither the resolver nor the runner is changed here.

## Exact B rebase continuation

Original B head: 6569fba5284fa41906db32cfddfcd300e2f3ab4b.
Original base: 6dde207eb96cbd9a0cee0a75113070fd4b836b75.
New base after GF-099 #2724 and handoff #2725:
daa149180937f03982187824b54f0d4acd11d999.

The actual Kit route was transfer rebase-on-upstream --upstream origin/main
--branch codex/external-projection-release-b --json. Its Git rebase origin/main
stopped at the first of all three B commits, reporting rebase_failed and
conflict_detected. Exactly five files were unmerged:

- CHANGELOG.md
- docs/DOCUMENTATION_REGISTRY.yaml
- docs/reference/EXTERNAL_HANDOFF_V2.md
- docs/reports/greenfield/external-findings-status.json
- src/agentic_project_kit/reference/external-findings-status.json

The curated changelog, registry and guide were combined. Both findings exports
were regenerated using the signed greenfield export plan/execute route;
doc-registry reconcile regenerated its scope summary. The retest index was
also regenerated. No generated export was hand-edited.

Before direct Git continuation, the helper verified that rebase-merge/head-name
was refs/heads/codex/external-projection-release-b, onto was exactly daa14918,
the unmerged set was exactly the five paths above, and those files contained
no conflict markers. The actual Git staging/continuation argv were:

```sh
git add -- CHANGELOG.md docs/DOCUMENTATION_REGISTRY.yaml docs/reference/EXTERNAL_HANDOFF_V2.md docs/reports/greenfield/external-findings-status.json src/agentic_project_kit/reference/external-findings-status.json docs/governance/DOC_REGISTRY_SCOPE_DECISION.md docs/reports/greenfield/EXTERNAL_FINDINGS_RETESTS.md
GIT_EDITOR=true git rebase --continue
```

All three B commits replayed. A subsequent full-diff inspection detected that
the first local conflict-combining helper had truncated the later changelog and
guide sections. This was an agent merge-helper error, distinct from KIT-GF-100.
Neither truncated version was pushed. Both complete original B documents were
restored, combined with main's GF-099 text, and given the requested durable
release wording. The release history and guide tail were checked intact.
The correction was committed via transfer commit as a fixup to the first B
commit, with only CHANGELOG.md and docs/reference/EXTERNAL_HANDOFF_V2.md paths.
Actual local history cleanup:

```sh
GIT_SEQUENCE_EDITOR=true GIT_EDITOR=true git rebase -i --autosquash origin/main
```

The resulting three-commit implementation head was
acd77b9b2a2a01d1c2338dca7569996058ba3354. There was no skipped B commit,
partial merge, reset, protected-branch push, or product-code fix for 100.

## Preservation and complete diff review

The following actual command outputs cover the entire rebased implementation.
The evidence/catalog-only commit containing this report follows that head.
Before the branch push, the exact final-head range-diff and full compare link
are additionally recorded in the #2723 PR report; this avoids a self-referential
commit hash in the file that defines that commit.

The original six B implementation/test files are byte-identical to the old B
head: external_projection_policy.py, greenfield_retest.py, kit_release_check.py,
test_external_projection_activation.py, test_kit_release_check.py and
test_release_greenfield_stamp.py. git diff --exit-code for that complete set
returned 0. Both later B commits remain patch-identical in range-diff.

The inherited GF-099 parser, preview, transition fixture, packaged identity data
and their tests are byte-identical to new main (git diff --exit-code returned 0).
Released changelog entries are byte-identical to main. The guide tail after
"When release B is active" is byte-identical to old B. The total diff has no
deleted file. Every remaining difference is original B activation/advisory/
test/evidence work, the requested durable documentation wording, regenerated
catalog metadata, or the new OPEN finding and this report. Main stays False;
only the B branch is True.

Full implementation compare:
https://github.com/vfi64/agentic-project-kit/compare/daa149180937f03982187824b54f0d4acd11d999...acd77b9b2a2a01d1c2338dca7569996058ba3354

### Full range-diff

```text
1:  7c009ee5 ! 1:  810b8517 Prepare external projection v2 activation for release B while preserving A evidence
    @@ CHANGELOG.md
     @@
      ## Unreleased
      
    -+- Changed: Enable external handoff projection v2 for release B (KIT-GF-094). First adopt release A **1.1.11**, then B. This activation remains on the PR branch until the owner confirms that all known external workspaces have adopted A; no merge or release is authorized yet.
    + - Fixed: Preserve the repository identity in target handoff previews by decoding Git-quoted origin values in the target and writing Git syntax in future planners. Legacy 1.1.11 payloads remain compatible; real-wheel regressions now use a GitHub-shaped origin distinct from the project name (KIT-GF-099). No release or v2 activation is performed by this fix.
    ++- Changed: Enable external handoff projection v2 for release B (KIT-GF-094). First adopt release A **1.1.11**, then B. All known external workspaces must adopt A before B is released.
     +- Changed: External content merges with current captured inputs require no refresh-only PR. Chat switches keep versioned handoff projections byte-identical; volatile identity remains in the ignored runtime file, producing no tracked handoff diff. Self-hosting retains its existing lifecycle.
     +- Fixed: The release-check advisory covers installations below 1.1.11. A direct jump from **1.1.10 or older** to B requires the documented migration route followed by another `kit update`; missing original adoption evidence leaves no governed installation return. KIT-GF-096 must remove external v1 compatibility no later than the second release after B.
    -+
    + 
      ## v1.1.11 - 2026-11-01
      
    - - Fixed: Stage external projection v2 across two releases: A retains v1 behavior while shipping the future migration planner; only a separately approved B activates the package-owned switch (KIT-GF-097).
     
      ## docs/DOCUMENTATION_COVERAGE.yaml ##
     @@ docs/DOCUMENTATION_COVERAGE.yaml: rules:
    @@ docs/DOCUMENTATION_COVERAGE.yaml: rules:
     
      ## docs/DOCUMENTATION_REGISTRY.yaml ##
     @@ docs/DOCUMENTATION_REGISTRY.yaml: documents:
    -   - path: docs/reports/release/release-run-1.1.11.json
    -     class: evidence/log
          owner: maintainers
    +     status: active
    +     summary: Sanitized counterprobe and current parser, preview and wheel-contract verification.
     +  - path: docs/reports/greenfield/KIT-GF-094-release-b-activation-2026-11-01.md
     +    class: evidence/log
     +    owner: maintainers
    @@ docs/governance/DOC_REGISTRY_SCOPE_DECISION.md: Decision outcome: active registr
      | docs/planning/ | 2 | 2 | 0 |  |
      | docs/reference/ | 11 | 11 | 0 |  |
      | docs/releases/ | 1 | 1 | 0 |  |
    --| docs/reports/ | 244 | 78 | 166 |  |
    -+| docs/reports/ | 245 | 79 | 166 |  |
    +-| docs/reports/ | 245 | 79 | 166 |  |
    ++| docs/reports/ | 246 | 80 | 166 |  |
      | docs/workflow/ | 12 | 12 | 0 |  |
      
      Notes:
    @@ docs/planning/PROJECT_DIRECTION.yaml: meta:
                  decision: F1
     
      ## docs/reference/EXTERNAL_HANDOFF_V2.md ##
    -@@
    - # External successor handoff v2
    +@@ docs/reference/EXTERNAL_HANDOFF_V2.md: strings. A GitHub-shaped fixture origin is transported only to a disposable
    + local bare repository through Git insteadOf rewriting. This parsing fix does
    + not change the projection activation policy.
      
    -+## Release B preparation and merge hold
    ++## Release B activation
     +
    -+This PR branch activates `EXTERNAL_PROJECTION_V2_ACTIVE = True`; its shipped
    -+default is projection contract 2. Main remains on release A (1.1.11) with the
    -+constant False until the owner explicitly confirms that **all known external
    -+workspaces have adopted A**. The B preparation PR stays open meanwhile. No
    -+partial merge, release, version bump or production migration is authorized.
    -+If main advances, rebase the complete PR before review.
    ++Release B activates `EXTERNAL_PROJECTION_V2_ACTIVE = True`; its shipped default
    ++is projection contract 2. Release A (1.1.11) retains the same single constant as
    ++False. There is no second switch or environment override.
     +
    -+After the local gates pass, the independent disposable consumer rehearsal uses
    -+published PyPI 1.1.11 as the actual planner and a wheel built from this PR as
    -+the target: A-to-B, B chat switch, a content merge with post-merge-check and no
    -+refresh PR, B-to-A return, and direct 1.1.10-to-B completion. Owner acceptance
    -+follows that rehearsal. The local self-built wheel matrix does not replace it.
    ++Before B is released, all known external workspaces must adopt A. The independent
    ++disposable consumer rehearsal uses published PyPI 1.1.11 as the actual planner
    ++and a B target wheel: A-to-B, B chat switch, a content merge with post-merge-check
    ++and no refresh PR, B-to-A return, and direct 1.1.10-to-B completion. Owner
    ++acceptance follows that rehearsal. Local self-built A wheels do not replace it.
     +
    -+GF-094 is CLOSED/unreleased in this branch so KIT-GF-091 can stamp it at the
    -+future B release, subject to the unchanged release prerequisites. Its release
    -+version is not guessed. The A fixture retains PARTIAL/unreleased and excludes
    -+094 from A stamping. KIT-GF-096 expires no later than the second release after B.
    ++KIT-GF-091 stamps GF-094 at release B after these prerequisites are satisfied;
    ++the fix version is not guessed. Release A retains PARTIAL/unreleased and
    ++excludes 094 from A stamping. KIT-GF-096 expires no later than the second release
    ++after B.
     +
      ## Two-release adoption (KIT-GF-097)
      
    @@ docs/reference/EXTERNAL_HANDOFF_V2.md
      `external_projection_policy.py`. It ships the future target preview, adoption
      receipt, runtime-consumer and return capabilities while external generation and
      checks retain v1. It adds no `projection_contract_min` and does not report
    + `NEEDS_PROJECTION_MIGRATION` through its external workflows. Module presence is
    + never activation. There is no environment or workspace activation option.
    + 
    +-Known external workspaces first adopt A. A later, separately approved release B
    +-changes that one constant to True. The installed A planner previews B, signs the
    ++Known external workspaces first adopt A (1.1.11). Release B changes that one
    ++constant to True. The installed A planner previews B, signs the
    + projection, manifest minimum and ignore-file changes, and writes adoption
    + evidence after successful execution. A-to-A-prime remains a v1 update.
    + 
     @@ docs/reference/EXTERNAL_HANDOFF_V2.md: starts with B. B1 adoption additionally waits for recorded cycle 008.
      
      ### Direct legacy-to-B completion and B release notes
    @@ docs/reference/EXTERNAL_HANDOFF_V2.md: starts with B. B1 adoption additionally w
      
      If an old planner installs B before detecting missing migration, preserve its
      BLOCKED result and completed steps. Use the existing
    +@@ docs/reference/EXTERNAL_HANDOFF_V2.md: The local A/B/A-prime matrix uses real noneditable wheels, loopback package
    + resolution, owned runtimes and real bare-origin squashes. Only PR observation
    + is a test double. A direct jump simulates the unsupported preview boundary;
    + installation and completion remain real. Independent consumer rehearsal and
    +-owner acceptance remain release gates; no release is authorized here.
    ++owner acceptance remain release gates.
    + 
    + When release B is active, KIT-GF-094 replaces commit freshness with deterministic content freshness only
    + for external workspaces. Self-hosting retains its v1 contract, C4 administrative
     
      ## docs/reports/greenfield/EXTERNAL_FINDINGS_RETESTS.md ##
     @@ docs/reports/greenfield/EXTERNAL_FINDINGS_RETESTS.md: Expected result fields: `{"assertions": ["release_check.installed equals the run
    @@ docs/reports/greenfield/external-findings-status.json
      {
        "authority": "Kit adjudication; VERIFIED_FIXED belongs to the consuming workspace",
        "counts": {
    --    "CLOSED": 95,
    -+    "CLOSED": 96,
    +-    "CLOSED": 96,
    ++    "CLOSED": 97,
          "NOT_APPLICABLE": 0,
          "OPEN": 1,
     -    "PARTIAL": 1
    @@ docs/reports/greenfield/external-findings-status.json
            "external_retest": {
     @@
        "kind": "external_findings_status",
    -   "machine_contracts": 56,
    +   "machine_contracts": 57,
        "schema_version": 1,
    --  "source_sha256": "caa4b3f57ab770c30992ce93f99a9747c66723edac3ec2d6830db7a167e84a86"
    -+  "source_sha256": "40dd85fcd079c166c47f32082eb2ef431f1a55671b34827860abcf412475838b"
    +-  "source_sha256": "8da0c620c16689fe14517b68ae5451d88d52090230f631eef1434a8dc366f9de"
    ++  "source_sha256": "86017599ecc5ee91d5070c177d19fa1760b7f02a2978ba0fc9235082d152e3c1"
      }
     
      ## src/agentic_project_kit/external_projection_policy.py ##
    @@ src/agentic_project_kit/reference/external-findings-status.json
      {
        "authority": "Kit adjudication; VERIFIED_FIXED belongs to the consuming workspace",
        "counts": {
    --    "CLOSED": 95,
    -+    "CLOSED": 96,
    +-    "CLOSED": 96,
    ++    "CLOSED": 97,
          "NOT_APPLICABLE": 0,
          "OPEN": 1,
     -    "PARTIAL": 1
    @@ src/agentic_project_kit/reference/external-findings-status.json
            "external_retest": {
     @@
        "kind": "external_findings_status",
    -   "machine_contracts": 56,
    +   "machine_contracts": 57,
        "schema_version": 1,
    --  "source_sha256": "caa4b3f57ab770c30992ce93f99a9747c66723edac3ec2d6830db7a167e84a86"
    -+  "source_sha256": "40dd85fcd079c166c47f32082eb2ef431f1a55671b34827860abcf412475838b"
    +-  "source_sha256": "8da0c620c16689fe14517b68ae5451d88d52090230f631eef1434a8dc366f9de"
    ++  "source_sha256": "86017599ecc5ee91d5070c177d19fa1760b7f02a2978ba0fc9235082d152e3c1"
      }
     
      ## tests/test_external_projection_activation.py ##
2:  54c50745 = 2:  861bf621 Make B retest fixtures idempotent and isolate returned owned runtimes
3:  6569fba5 = 3:  acd77b9b Record B activation verification and preserve future release-stamp compatibility
```

### Total diff against main: complete changed-file set and totals

```text
M	.agentic/greenfield_findings.json
M	CHANGELOG.md
M	docs/DOCUMENTATION_COVERAGE.yaml
M	docs/DOCUMENTATION_REGISTRY.yaml
M	docs/TEST_GATES.md
M	docs/governance/DOC_REGISTRY_SCOPE_DECISION.md
M	docs/planning/PROJECT_DIRECTION.yaml
M	docs/reference/EXTERNAL_HANDOFF_V2.md
M	docs/reports/greenfield/EXTERNAL_FINDINGS_RETESTS.md
A	docs/reports/greenfield/KIT-GF-094-release-b-activation-2026-11-01.json
A	docs/reports/greenfield/KIT-GF-094-release-b-activation-2026-11-01.md
M	docs/reports/greenfield/external-findings-status.json
M	src/agentic_project_kit/external_projection_policy.py
M	src/agentic_project_kit/greenfield_retest.py
M	src/agentic_project_kit/kit_release_check.py
M	src/agentic_project_kit/reference/external-findings-status.json
M	tests/test_external_projection_activation.py
M	tests/test_kit_release_check.py
M	tests/test_release_greenfield_stamp.py

 .agentic/greenfield_findings.json                  |   6 +-
 CHANGELOG.md                                       |   3 +
 docs/DOCUMENTATION_COVERAGE.yaml                   |   2 +
 docs/DOCUMENTATION_REGISTRY.yaml                   |  10 +
 docs/TEST_GATES.md                                 |  12 +-
 docs/governance/DOC_REGISTRY_SCOPE_DECISION.md     |   2 +-
 docs/planning/PROJECT_DIRECTION.yaml               |   8 +-
 docs/reference/EXTERNAL_HANDOFF_V2.md              |  29 +-
 .../greenfield/EXTERNAL_FINDINGS_RETESTS.md        |   4 +-
 ...KIT-GF-094-release-b-activation-2026-11-01.json | 477 +++++++++++++++++++++
 .../KIT-GF-094-release-b-activation-2026-11-01.md  |  84 ++++
 .../greenfield/external-findings-status.json       |  12 +-
 .../external_projection_policy.py                  |   2 +-
 src/agentic_project_kit/greenfield_retest.py       |  13 +-
 src/agentic_project_kit/kit_release_check.py       |   6 +-
 .../reference/external-findings-status.json        |  12 +-
 tests/test_external_projection_activation.py       |  31 +-
 tests/test_kit_release_check.py                    |  15 +-
 tests/test_release_greenfield_stamp.py             |  28 +-
 19 files changed, 710 insertions(+), 46 deletions(-)
```

## Verification

At acd77b9b: full suite 4,526 PASS, 560 warnings, 393.48 seconds.
Fresh noneditable B wheel: machine contracts 094, 095, 097, 098, 099 all PASS.
096 has no machine contract. Ruff, check, check-docs, doctor and site build PASS.
The isolated 100 probe above remains the expected BLOCKED negative test.
Final evidence/catalog checks are recorded in the B activation report and PR.

The independent consumer must repeat published PyPI 1.1.11 -> rebased B without
a probe patch. All-known-workspace A adoption and owner acceptance remain pending.
The PR remains a draft, with no merge, release, version bump or auto-merge.

## Katalogpflicht und gezielte Gegenprobe

The initial catalogue addition without a machine contract failed the unchanged
new-local-finding gate: 4525 tests passed, one failed
(test_only_the_named_local_retests_lack_a_contract). No exemption or test change
was made. The existing-runner negative reproduction then passed all 19 steps.
The finding remains OPEN with no assigned fixed release; successful resolution
is not claimed. The full final suite result is recorded in the PR report.
