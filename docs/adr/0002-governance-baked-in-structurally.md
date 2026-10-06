# ADR-0002: Template ships governance as structure, not convention

**Status:** Accepted (2026-06-14, reinforced 2026-06-25; reconstructed 2026-08-13; amended 2026-10-06, see end)

## Context

Every repo derived from this template is co-authored with Claude (Anthropic). The fleet needed
three guarantees without relying on per-repo discipline:

1. AI co-authorship is disclosed in every derived repo.
2. Automated CI behaviour that touches pull requests is an explicit, reviewable decision — not a
   default that silently runs on every PR.
3. The template's boilerplate `review_prompt` in `claude-code-review.yml` must be replaced before
   the automated review runs. A prompt written to describe the template repo will review every
   derived repo on the wrong rubric; the file comment documents this as the known failure mode.

## Decision

Bake governance into the template's file structure rather than document it as a convention:

1. **Authorship disclosure** — The `**Authorship:**` block is included in `README.md` with
   fill-in instructions. Omitting it requires a deliberate deletion, not merely skipping a
   checkbox. (Present at initial commit, 2026-06-14.)

2. **Auditable CI off-switch** — `claude-code-review.yml` was switched from `pull_request` to
   `workflow_dispatch` trigger via PR #2 (merged 2026-06-25). The decision is git-tracked with a
   dated comment directly in the workflow file: `>>> DISABLED 2026-06-25`. Re-enabling automated
   review requires a deliberate PR restoring the trigger — the off-state is not silent default,
   it is a visible, reviewable fact.

3. **Do-not-ship-boilerplate guard** — `claude-code-review.yml` is prominently headed with
   `>>> CUSTOMIZE THE review_prompt BELOW FOR THIS REPO. <<<`, and `SETUP.md` step 1 explicitly
   calls out: "Do not ship the boilerplate." The workflow comment states: "A copy-pasted prompt
   that describes the wrong repo is worse than none — it points the reviewer at the wrong rubric."

Evidence (verified against current repo):

- `README.md` lines 13–18: the `**Authorship:**` block with fill-in instructions.
- `.github/workflows/claude-code-review.yml` lines 7–13: boilerplate warning;
  line 18: `DISABLED 2026-06-25` comment; line 23: `workflow_dispatch` trigger.
- PR #2 (merged 2026-06-25): "Disable automated Claude PR review (manual-only trigger)" —
  confirmed merged; PR body records this as part of a fleet-wide change.

## Alternatives

**CONTRIBUTING.md convention** *(retrospective — not considered at the time)* — A conventions
document listing the authorship disclosure requirement and review configuration steps. *Worse*:
easy to miss; provides no structural enforcement — the exact failure mode shipping the structure
avoids.

**CI lint check** *(retrospective — not considered at the time)* — A workflow that asserts the
authorship block is present and detects the boilerplate `review_prompt`. *Lateral*: stronger
than a comment, but adds CI complexity and requires maintaining detection heuristics as the
boilerplate text changes. (`fleet-apply.sh` later grew a boilerplate-prompt scan, covering part
of this from the settings side.)

**Leave automated review enabled by default** *(the recorded prior state — replaced by PR #2)* —
Keeps the automated review running on every PR so issues are surfaced without manual invocation.
The specific problem: if the `review_prompt` is still the placeholder when a derived repo opens
its first PR, review runs against the wrong rubric. Enabling review-by-default without solving
the boilerplate problem first amplifies the failure mode rather than discouraging it. *Worse*
for the stated goal of ensuring review is useful before it runs.

**New-repo issue template as setup checklist** *(retrospective — not considered at the time)* —
A GitHub issue template pre-opened on repo creation that lists customisation steps including
updating the `review_prompt`, closed when complete. More visible than a comment in a workflow
file; still optional since nothing prevents closing the issue without completing all steps.
*Lateral*: adds a task-management surface for the setup checklist without providing structural
enforcement of any individual step.

## Consequences

- New repo authors encounter the authorship block and the boilerplate warning in the first files
  they open; neither can be missed silently.
- Automated review is off by default; re-enabling is a deliberate PR, not an accidental trigger.
- The auditable off-switch pattern (a dated disable comment + `workflow_dispatch` trigger) is
  itself now a fleet pattern, documented in this repo's README as a live example.

## Amendment (2026-10-06): the review workflow is retired

`claude-code-review.yml` was deleted from the template (and fleet-wide). It had
been dispatch-only since 2026-06-25, and CodeRabbit was installed org-wide on
2026-10-03 as the advisory AI reviewer, configured once in `lentago/coderabbit`
and never a merge gate. The evidence lines above describe the file as it stood
at its [last version](https://github.com/lentago/repo-template/blob/dc9c2ae82fef3525e9726a46f09f3ffd537fb64f/.github/workflows/claude-code-review.yml).

Guarantees 2 and 3 now hold by absence rather than by an off-switch: the
template ships **no** workflow that runs AI review on pull requests, so there is
no default to turn off and no per-repo `review_prompt` that could ship as
boilerplate. A derived repo that needs a different review rubric adds a
`.coderabbit.yaml` in a reviewable PR. Guarantee 1 (authorship disclosure) is
unchanged. The SETUP.md "replace the `review_prompt`" step was removed with the
file.
