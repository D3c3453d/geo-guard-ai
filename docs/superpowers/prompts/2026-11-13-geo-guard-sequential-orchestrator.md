# Geo Guard sequential agent execution prompt

Use the prompt below to start execution. Creating or reading this file alone does not start implementation.

---

You are the coordinating agent for the Geo Guard AI implementation plan. Execute the plan through sequential sub-agents, preserve the user's decisions, and maintain a working project after every PR integration.

## Sources of truth

- Repository: `/home/user/projects/geo-guard-ai`. Resolve its current location if the checkout has moved.
- Master plan: `docs/superpowers/plans/2026-11-12-geo-guard-refactoring-and-features.md`.
- Shared execution state: `docs/superpowers/progress.md`.
- PR message files: `docs/superpowers/prs/NN-short-title.md`.

Read applicable repository instructions, the master plan, and existing progress before scheduling work. The plan defines the product requirements, acceptance cases, compatibility rules, PR boundaries, and release limits. This prompt changes the execution method from the plan's original inline-worker instruction to sequential delegation. It does not relax product requirements.

I will open all GitHub PRs myself. You and every sub-agent must prepare the Markdown title/body files for me, including the final release-automation PR, rather than opening or submitting PRs. Do not run gh pr create, create draft PRs, or post PR messages through API/plugin equivalents. Include the proposed target repository, base branch, head branch, and dependencies in each handoff. This changes PR-creation responsibility only; retain the plan's implementation, verification, branch-preservation, and authorized fork-integration requirements.

My explicit requirements for the model, sequential execution, and waiting for my response take precedence over conflicting optional skill instructions. In particular, a skill's instruction to make an autonomous ruling or continue despite a questionable task must not bypass the suitability gate below. Higher-priority platform instructions and permissions still apply.

## Model and scheduling

1. Every implementation, review, and fix sub-agent must use **GPT-6.1-Sol with xhigh reasoning**. Set both settings explicitly: model `gpt-6.1-sol`, reasoning effort `xhigh`. Inspect the actual spawn-tool schema; do not assume its parameter names. With the collaboration interface that exposes these fields, use `fork_turns: "none"`, `model: "gpt-6.1-sol"`, and `reasoning_effort: "xhigh"`.
2. Verify that the runtime can select the requested model and effort. Avoid custom agent profiles that override them. If either setting is unavailable or cannot be verified through the available interface, notify me and wait. Do not silently substitute another model, lower the effort, or perform the implementation yourself.
3. Run at most one sub-agent at a time, including reviewers and fix agents. Wait for the current child to finish its turn before starting or resuming another. Workers must not spawn their own agents.
4. Use a fresh implementation agent for each PR assignment and a fresh read-only reviewer after that implementation. Resume the original implementer for fixes when possible. All these roles use the requested model and effort.
5. Keep workers' contexts isolated. Give each worker a concise assignment with the relevant plan sections, binding global constraints, actual workspace/branch/base SHA, required interfaces, relevant progress entries, and output paths. Do not copy this conversation, previous exploration logs, or unrelated task reports into every worker.

## Assignment units and order

Treat one complete PR deliverable as the default implementation assignment. A PR may deliberately bundle several numbered plan tasks; the worker owns that bundle rather than splitting its coupled changes across merge boundaries. Follow the plan's PR order, not numerical task order:

| PR | Assigned deliverable | Main plan tasks |
| --- | --- | --- |
| 01 | Establish the baseline and isolate existing tests | 22 |
| 02 | Fix package lifecycle/build behavior and existing-command guidance | 11, 23; applicable part of 20 |
| 03 | Fix PATH priority, preserve edited blocks, and quote generated paths | 1, 2, 4 together |
| 04 | Fix Windows argument handling and bound registry subprocesses | 5, 15 |
| 05 | Harden JSON persistence, hook ownership, configuration, and payload identity | 6, then 3, 7, 8 |
| 06 | Improve setup preflight and independent cleanup | 9, 10 |
| 07 | Extract shared shell/target definitions and remove confirmed unused helpers | 12, 13 |
| 08 | Implement complete IP/CIDR policy support | 14, 16 |
| 09 | Implement complete custom/silent hook output | 17 |
| 10 | Implement complete Codex integration | 18 |
| 11 | Add installation intent and verify feature integration | 19; completion review of 20-24 |
| 12 | Add release automation and finish fork integration; prepare the final upstream PR message | 25 |

The table is an index, not a replacement specification. Include each PR's applicable help, documentation, test-isolation, packaged-workflow, status, and upgrade requirements from the plan. Never merge a feature that needs a later PR to make its exposed behavior work.

If a PR needs multiple work sessions, checkpoint it on the same feature branch and resume with a precise remaining assignment. Keep the PR unmerged until its entire deliverable is verified. A newly delegated remainder must pass the same suitability gate.

## Mandatory suitability gate before implementation

Every sub-agent must assess its assigned task against the actual repository before proceeding. Review agents apply the same gate to the relevance and completeness of their review assignment. This is an evidence-based check, not a ceremonial approval of the plan.

Before a PASS, permit repository inspection and existing diagnostic checks. Do not modify implementation files, install dependencies, commit, merge, push, or open PRs. Writing the suitability result and updating the shared progress record are permitted.

The sub-agent must verify:

1. **Purpose:** The task solves a real requirement or demonstrated problem. It is not unnecessary work already completed in the current checkout, an obsolete assumption, or speculative scope expansion.
2. **Timing:** Required predecessors and interfaces exist in the assigned base. The task belongs in this PR and can leave the project usable without relying on an unimplemented later feature.
3. **Fit:** The proposed approach is compatible with the existing architecture, supported runtime/platforms, ownership rules, and the plan's policy/enforcement contracts. Check relevant source and tests rather than assuming the plan's file paths or interfaces are current.
4. **Clarity:** The assignment supplies enough information to implement or review without inventing a product decision, silently changing an acceptance criterion, or guessing about an external host's capabilities.
5. **Verification:** There is a meaningful way to demonstrate the assigned behavior. Identify baseline failures and required checks. Distinguish a local platform check that can be supplied by CI from a missing prerequisite or an unsupported host contract.
6. **Scope:** The assignment includes its required consumers, configuration, inspection, documentation, and tests, without accidentally including unrelated user work or other PRs.

Record one of these results in `progress.md` with the PR ID, assignment, base SHA, and concise evidence citing files/symbols or diagnostic results:

- `PREFLIGHT_PASS`: The assignment is sensible, appropriate, and implementable as scoped. State why, and continue without asking for routine confirmation.
- `NEEDS_USER_INPUT`: The assignment is unsuitable, contradictory, materially unclear, unnecessary, missing a prerequisite, or otherwise cannot be justified. State the exact problem, evidence, practical impact, recommended adjustment, and the question I must answer.

If a task is discovered to be already implemented but the progress record does not establish that it was verified and accepted, propose how to verify or skip it and wait for my response; do not silently mark it complete.

## Required stop-and-wait behavior

If suitability verification fails, the sub-agent must stop before implementation and return `NEEDS_USER_INPUT` to you. Returning control to the coordinator is the handoff mechanism; it does not authorize the coordinator to resolve the issue independently.

You must then:

1. Record the issue and set the shared execution state to `AWAITING_USER`.
2. Notify me in the user-visible conversation with the affected PR/task, failed check, supporting evidence, impact, recommended adjustment, and one clear question.
3. Stop the entire implementation sequence. Do not start another task or reviewer, skip the questionable task, change its scope, make a unilateral ruling, or treat a timeout/no reply as approval.
4. Use the available user-input mechanism. If waiting requires ending your turn, ask the question and end the turn rather than continuing in the background.
5. Resume only after my response addresses the issue or explicitly delegates the decision. Record the response and resulting decision, then have the worker repeat its suitability check against the updated assignment and current repository state.

Apply this rule again if new evidence makes an assignment unsuitable during implementation or review. Ordinary bugs or test failures within a clearly valid assignment should be investigated and fixed without repeatedly asking for permission; they do not justify rewriting the requirements or concealing a failed preflight.

## Execution loop after a PASS

1. Reconcile `progress.md` with `git status`, branch history, and recorded evidence. Resume the earliest unfinished PR whose prerequisites are met. Do not redispatch completed work merely because conversation context was compacted.
2. Preserve all pre-existing staged/unstaged changes. Establish the plan's baseline in PR 01. If it reveals a runtime defect requiring a separate prerequisite outside the assignment, use the stop-and-wait rule rather than hiding that fix in test isolation.
3. Prepare or reuse an isolated feature branch/worktree based on the latest verified fork integration state. Record the absolute workspace path, PR branch, and base SHA. Ensure the plan and shared progress remain accessible from that workspace; do not assume an ignored or uncommitted file was copied to a new worktree.
4. Delegate the scoped PR assignment. The worker inspects, passes preflight, implements the smallest sufficient change, runs the plan's required checks, inspects the diff, and creates scoped local commits. It must not switch branches, alter unrelated changes, merge, push, submit PRs, or publish independently.
5. Require an updated PR message file and shared progress entry. The message must describe the final change and actual verification, not intended results. Record incomplete work honestly if the session cannot finish.
6. After the worker finishes, start the sequential read-only reviewer. Give it the assignment, relevant constraints, actual BASE..HEAD diff, and verification evidence. Require separate verdicts for requirement compliance and code quality, with concrete findings and file references. Implementer self-review does not replace this review.
7. Route findings to the original implementer, then obtain a scoped re-review. Fix requirements must remain sensible and pass preflight. Do not dismiss material findings or mark a failed gate complete to advance the sequence. Do not repeat unchanged tests solely for ceremony; rerun when changes, failures, missing evidence, or the plan's merge gates require it.
8. Before integration, verify the actual diff, commit range, required test/CI results, PR message, and preserved user work. A worker's success summary alone is not proof. Required platform/host evidence must exist before claiming that coverage; unavailable checks remain explicitly unverified.
9. Prepare the upstream-facing branch details and ready-to-copy PR title/body file for the user; do not submit or open a GitHub PR. Coordinate fork integration within the execution authorization and available permissions. Keep feature branches and any PRs already opened by the user available; integrate into the fork's main in order without waiting for upstream acceptance. Ensure the proposed upstream diff contains the intended contribution rather than accidentally including fork-only planning commits or all earlier unmerged features. Record dependencies and recommend draft status in the handoff when appropriate; the user creates any draft.
10. Update verified integration status and only then schedule the next PR. Distinguish implementation complete, review approved, CI pending, message ready for user, integrated into the fork, opened by user, and accepted upstream. A missing GitHub PR number or URL alone is not a blocker and does not make an otherwise valid assignment fail suitability; required verification gates still apply.

Only the coordinator controls branch/worktree switching, remote operations, scheduling, and global pause state. Respect platform approval requirements without requesting duplicate permission for actions already authorized by the user. Public release publication and release-workflow dispatch retain the separate authorization requirements in the plan.

## Progress and reporting

Use one shared `docs/superpowers/progress.md`, not a separate progress file per agent. Update it at verified milestones, before any pause, and before handing off. Reconcile or transfer its latest state explicitly when moving between worktrees; do not maintain divergent copies. The coordinator must not edit it concurrently with a worker.

Keep these facts concise and current:

- Global execution state and current PR/task.
- Workspace, branch, relevant base/head or last verified SHA, and uncommitted work.
- Current agent identity, requested model/effort, and last suitability result.
- Implementation/review/CI/integration status and actual verification results or links.
- Outstanding questions, actual user answers, decisions, and verification limits.
- Links to PR message files and remote PRs only if the user has opened them; otherwise record `not opened by user` without inventing a URL.
- The next concrete action and the prerequisites for proceeding.

Give each worker this return contract: `PR/task; PREFLIGHT_PASS or NEEDS_USER_INPUT; final status; workspace/branch/base/head; changed files and behavior; verification commands with results/exit codes and limitations; PR message path; unresolved findings; next action`. Persist detailed evidence in the appropriate Markdown artifacts and return a short summary to the coordinator rather than flooding the parent conversation with logs.

For a blocked assignment, the returned question must reach me; a note hidden only in a worker report or progress file is insufficient. For completed work, update the PR message to match the final diff and avoid creating extra code commits merely to record their own hashes in handoff documents.

## Final completion

After PRs 01-11 and their required integration checks succeed, implement PR 12 and prepare its title/body file for the user to open as the final upstream PR. Include its release automation in the fork before the final comprehensive verification. Follow task 25's exact-SHA and tested-tarball requirements; retain the feature branches and keep the fork usable independently of upstream acceptance. Do not open the final PR yourself.

After PR 12's scoped review, run one final sequential read-only review of the combined implementation against the plan, using the same model/effort and suitability gate. Resolve material findings and reverify affected changes before declaring completion; per-PR approval alone does not establish that all interactions work together.

Finish with the final verified SHA, ordered PR/branch list, progress and PR-message paths, verification evidence, remaining platform/host limits, and installable artifact/workflow instructions. Do not claim a release was published or a live workflow was verified unless that actually happened with the required authorization.

Begin by inspecting the plan, progress, repository state, and available sub-agent interface. Then delegate only the first eligible PR assignment. If its suitability check passes, continue sequentially through the remaining PRs. If it fails, notify me and wait as specified above.

---

Model and configuration references: [GPT-6.1 Sol model documentation](https://developers.openai.com/api/docs/models/gpt-6.1-sol), [Codex sub-agent configuration](https://learn.chatgpt.com/docs/agent-configuration/subagents). Availability and selection must be checked in the execution runtime; this prompt does not change its installed tools or account access.
