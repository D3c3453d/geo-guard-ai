# Geo Guard AI refactoring and feature implementation plan

> For agentic workers: use superpowers:executing-plans to implement this handoff in the PR sequence below, completing each PR's requirements together before merging. The original 24 tasks retain their numbers; task 25 adds fork integration and the final release-automation PR. Task numbers are stable requirement references, not the execution order. The user requested numbered instructions, so this document intentionally does not use checkboxes.

**Goal:** Fix PATH and configuration correctness, remove obsolete npm hooks, add IP/CIDR policies and configurable hook messages, and integrate the Codex CLI without breaking existing Claude Code or Cursor installations.

**Architecture:** Keep the existing TypeScript CLI and explicit setup/uninstall workflow. Reuse small shared filesystem and shell utilities, keep host-specific hook-file adapters separate, and evaluate country/IP policies through one decision layer used by prompt checks and command wrappers.

**Tech stack:** TypeScript, CommonJS output, Node built-ins, node:test, existing POSIX and PowerShell test scripts. Keep the current Node engines expression: >=18.20.0 <19.0.0 || >=20.12.0. Do not raise the runtime floor or add runtime dependencies as an incidental refactor.

**Specification:** The user's combined 24-item list in this conversation, supplemented by the three confirmed choices below and the subsequent request to preserve upstream PR branches, integrate into the fork's main, verify the combined commit, and prepare release automation for the final upstream PR. The user opens all GitHub PRs; agents prepare their Markdown messages only. This edit changes the handoff only; it does not execute implementation, remote merges, workflow dispatch, or publication.

**Status:** Revised for eleven implementation PRs followed by a twelfth, final release-automation PR, independent use of the completed fork, and one coordinated release. Product implementation is not part of this document-writing task.

## Confirmed requirements and explicit planning choices

1. Confirmed by the user: bare geo-guard shows guidance and help; it does not automatically start setup.
2. Confirmed by the user: IP policies support individual IPv4/IPv6 addresses and CIDR ranges.
3. Confirmed by the user: an empty interception message suppresses all package-emitted human-readable prompt-hook output, including network and configuration errors, while enforcement remains active.
4. The exact field names, flags, normalization rules, defaults, metadata format, and status semantics below are recommended implementation decisions made for this handoff. They are specified explicitly so an executor does not silently invent alternatives.
5. There are exactly three policy modes: countries, ips, and both. Both means AND. There is no OR mode. Existing installations default to countries.
6. Hook-message configuration is global for this release. Per-tool message overrides, message templates, automatic allowlisting of the current IP, and CIDR aggregation are outside this scope.
7. Existing country endpoints remain unchanged. IP detection uses a separate provider list and does not require GeoIP in ips mode. Combined mode evaluates the two current observations independently; it does not promise that different providers observe the same address family or egress route.
8. Ordinary automated tests use mocked fetches, fake binaries, or local HTTP servers. Live prompts, real provider credentials, and production geolocation endpoints are not automated-test fixtures.
9. Keep all uncommitted user work. At drafting time package.json and package-lock.json already contain user changes, including TypeScript ^5.9.3. Do not revert or attribute those changes to this plan.
10. Review and merge complete changes through separate PRs, then publish one coordinated release after the final integration checks. Every merged state must remain usable. In particular, new PATH templates, edited-block preservation, and legacy-template recognition must pass together in the same PR. Merging a PR does not authorize publishing.
11. Negative setup flags skip installation; they do not silently remove an already installed hook or shim. Report an existing component separately from a newly skipped component. Removing existing components remains explicit uninstall behavior.
12. Keep the feature branches for upstream PRs and preserve any PRs the user has opened while integrating the verified changes into D3c3453d/geo-guard-ai's main in the planned order. The fork's operation and release preparation do not depend on the original maintainer merging those PRs. Prepare the GitHub Actions release workflow and its message as the final PR for the user to open in the original repository. Agents must not create or submit GitHub PRs, including drafts, or post PR messages through CLI, API, or plugin tools.

## Execution contract

1. Before editing, inspect repository instructions, git status, the relevant source files, and the tests named by the task. Recheck paths and interfaces against the current checkout.
2. Establish a baseline before runtime implementation. A fresh typecheck passed during drafting. npm test failed in check.test.cjs, config.test.cjs, hooks.test.cjs, setup.test.cjs, and status.test.cjs under Node v26.13.1. The cause was not established. Reproduce and classify those failures in PR 01, before describing any later failure as a regression. Fix test-isolation defects there. If a failure requires a runtime fix, review that narrow prerequisite separately and establish a passing baseline before proceeding with the remaining PRs; do not hide runtime changes in the test-isolation PR or weaken assertions.
3. Follow the PR sequence and dependencies below instead of numerical task order. Each task identifies its owned files and acceptance cases; shared files may be extended by later PRs. Tasks 20, 21, 22, and 24 apply incrementally. Tests, help, documentation, and any required status changes belong in the PR introducing the behavior, not in a deferred catch-up PR.
4. For each behavior change, add a focused regression case, observe its failure for the intended reason, implement the smallest fix, and run the focused suite. Do not create tests solely for deleting unused functions or changing comments.
5. Keep unrelated formatting, dependency upgrades, broad parser replacements, and speculative abstractions out of the changes.
6. Preserve arbitrary user-owned JSON fields, shell content, hooks, backups, and non-owned executables except where the task explicitly changes package-owned data. Only reset retains its existing documented whole-config replacement semantics.
7. Do not infer successful installation from a zero process exit alone. Inspect the required files, outputs, side effects, and host behavior described by the acceptance cases.
8. If a source or host capability contradicts a requirement, record the exact conflict. Do not replace blocking interception with notifications or launch-only checks and call the requirement complete.
9. Report each task as implemented, verified, or blocked, with actual evidence. Do not mark an unavailable platform check as passed.
10. For every PR, including the final release-automation PR, the agent must write a Markdown PR message file at docs/superpowers/prs/NN-short-title.md for the user to copy when opening the PR manually. NN is the plan's PR sequence number, not GitHub's assigned PR number. Include the proposed title and a ready-to-use PR body explaining the problem, resulting behavior, relevant implementation details, actual verification results (including failures or unavailable checks), and any dependencies or material limitations. Record the proposed target repository, base branch, and head branch alongside the message. Keep it concise and aligned with the final diff; update it when the change evolves, preserve Markdown and real newlines, and link the file in the completion report or handoff. Do not run gh pr create or any equivalent API/plugin operation. A missing GitHub PR number or URL alone does not block implementation or message preparation; record not opened by user until a real link is available. Required verification gates still apply.
11. Every executing agent must maintain one shared docs/superpowers/progress.md, creating it at the start of implementation if absent. Read it at the start of each session and reconcile its contents with the actual Git state and code before continuing. Update it after each verified milestone and before ending or handing off a session. Record completed and current PR/task references, the working branch, the last verified commit SHA, uncommitted work, unresolved failures or pending/unavailable checks, decisions needed by subsequent agents, and the next concrete action. Link the relevant PR message files for detailed verification evidence rather than duplicating their contents or copying full test logs. Keep completed, in-progress, and blocked work distinct; do not mark unverified work complete. Maintain this single shared handoff file rather than separate progress files for individual agents.
12. When an orchestrator delegates work, every implementation, review, fix, and final-review sub-agent must use the same model and effort settings as the orchestrator at dispatch or resumption. Prefer the runtime's parent-inheritance mechanism, leaving overrides unset when that preserves both settings. If explicit values are required, use the orchestrator's actual current values; do not hard-code a model, provider, or named effort level, or let a role profile substitute different settings. Where effort is not exposed, preserve the parent's default or unset state. If matching settings cannot be established through the available interface, notify the user and wait rather than choosing a fallback. Apply any later orchestrator-setting changes to subsequent or resumed sub-agents, reconfiguring or replacing a child with a handoff when needed.

## PR boundaries and merge dependencies

Use the following default merge sequence into the fork's main: PR 01 -> PR 02 -> PR 03 -> PR 04 -> PR 05 -> PR 06 -> PR 07 -> PR 08 -> PR 09 -> PR 10 -> PR 11 -> PR 12. Each integration PR is based on the preceding merged result. A dependent PR may be reviewed against its predecessor while that predecessor is open, but must be updated and reverified against the actual merge result before it merges. Do not treat these as twelve independent changes that can land in arbitrary order. Keep the upstream review branches; upstream merge timing does not gate this fork sequence.

| PR | Deliverable | Numbered task order within the PR | Requirements that must land together |
| --- | --- | --- | --- |
| 01 | Reliable baseline and isolated tests | 22, for existing behavior | Reproduce the recorded failures, isolate current fixtures, and record the supported Node/npm baseline. Add fixture support for later features in their own PRs. |
| 02 | Explicit package lifecycle and reproducible builds | 11 and 23 together; existing-command portion of 20 | Remove obsolete install/removal hooks together with fresh tarball builds, developer-hook separation, installation/cleanup guidance, and installed-package tests. |
| 03 | Safe PATH priority and generated paths | 1, 2, and 4 together | New generators, old/new template recognition, read-back parsers, preservation of edited blocks, and actual startup/quoting tests. |
| 04 | Reliable Windows process handling | 5, then 15 | Argument round trips and bounded registry subprocesses, with Windows smoke coverage and registry restoration. |
| 05 | Safe hook/configuration handling | 6, then 3, 7, and 8 | Adopt the JSON writer in existing callers, preserve foreign hooks, reject malformed existing policy, and retain country-only enforcement for unknown hosts. |
| 06 | Honest setup and independent cleanup | 9, then 10 | Preflight all currently supported destinations, protect user content, report partial setup, and continue independent cleanup after failures. |
| 07 | Shared shell and target definitions | 12, then 13 | Mechanical extraction and confirmed unused-helper removal with unchanged current behavior. Update package-content checks for moved modules. |
| 08 | Complete IP/CIDR policy support | 14, then 16 | Parsing, detection, configuration, setup, prompt checks, launch wrappers, unknown-host conjunction, mode-aware status, CLI help, and packaged policy tests. |
| 09 | Complete custom/silent hook output | 17 | Message configuration, early quiet parsing, generated commands, hook ownership/status handling, output completion, and exact-output tests. |
| 10 | Complete Codex integration | 18 | Target/profile registration, setup, hooks/shims, enforcement, quiet commands, basic status/trust reporting, explicit removal, and host verification. |
| 11 | Installation intent and feature integration | 19; final coverage review of 20-24 | Metadata persistence, setup preflight and uninstall support for it, refined structural status, and the full accumulated release matrix. |
| 12 | Fork release automation; final upstream PR message | 25, after successful completion of 1-24 | Preserve upstream branches, reconcile ordered fork merges, verify the combined version, add the reusable GitHub Actions release workflow, reverify the final commit, and prepare the final PR title/body for the user. |

The count is a review aid, not a target. Combine adjacent dependent implementation PRs if their separation would require temporary compatibility machinery or leave an incomplete feature. Keep release automation as the last upstream PR. Do not split tasks 1/2/4, task 16's consumers, task 17's generator/parser/output changes, or task 18's lifecycle integration across merge boundaries.

### Compatibility requirements for every merge

1. Keep the current application usable throughout the sequence: existing country-only configuration and Claude/Cursor installations continue to work unless an explicit correctness fix rejects previously malformed data. Preserve documented inheritance and supported legacy artifacts.
2. Introduce configuration writers and all affected readers in the same PR. Do not expose a new setup flag, saved policy mode, hook argument, or target until its enforcement and inspection paths support it. No intermediate stubs, permissive fallback, or temporary feature flags are needed for this split.
3. Update existing status/config display, all help forms, both READMEs, relevant release notes, and packaged workflows with the behavior they describe. Advertise only implemented commands and flags. These are parts of tasks 20, 21, and 24 owned by each PR, not separate final-only deliverables.
4. Before PR 08, retain the existing country-only decision behavior. PR 08 changes both runCheck and runWrap to the shared evaluator and makes status evaluate the selected mode; IP-only status must not require GeoIP. Preserve countries as the default for old configuration. Do not defer these status changes to PR 11.
5. PR 09 must never install a --quiet command before the same binary accepts it and suppresses errors correctly. PR 10 must not install a Codex handler before that binary recognizes its explicit profile. Both extend the same ownership and preservation rules already established in PR 05.
6. Before PR 11, keep installation expectations based on existing discovery, extended for each supported feature. PR 10 already distinguishes Codex file presence from unverified trust. PR 11 adds installation-intent metadata and richer diagnostics; no earlier PR imports a future installation-state module or creates placeholder metadata.
7. Test upgrades from fixtures of the currently shipped configuration, hooks, PATH blocks, and shims. Re-run setup on the accumulated version and assert no-op behavior where appropriate, preserved user content, and complete explicit cleanup. Add fixtures for new artifacts with their owning feature PR.

### Verification and release gates

1. Each PR includes its focused acceptance cases and runs npm run typecheck, npm test, npm run test:e2e, and npm run test:pack against that PR's accumulated behavior. Use the existing Windows unit enumeration and scripts/windows-smoke.ps1 for Windows coverage. A check for a feature not yet introduced is added when that feature lands, rather than disabled to permit an incomplete implementation.
2. Required CI checks must pass before merging. Record command results and relevant versions; an unavailable local platform check is reported as unavailable and needs the applicable platform evidence before claiming that coverage. Do not use unexplained baseline failures or a later PR as substitutes for a passing merge gate.
3. Extend task 24's installed-binary tests with every affected workflow. PR 02 establishes lifecycle-warning-free installation; PRs 03-07 retain and extend existing workflows; PRs 08-11 add their new policy, output, host, and metadata cases. Cross-feature checks cover IP policies with quiet hooks and Codex with explicit profiles, IP policies, and quiet output.
4. After PR 11, execute the complete task-24 matrix and review documentation and tarball contents against the original 24 tasks before starting PR 12. After merging PR 12 and any fork-only version metadata, run that matrix again against the actual final combined commit, including release-workflow validation. Record its full SHA; any subsequent commit requires fresh verification. Preserve platform/host verification limits in the handoff. Intermediate merges do not trigger a release, and public publication remains separately authorized.

## Fork integration and manual upstream PR handoff

1. At execution time, verify that origin identifies the user's fork, currently D3c3453d/geo-guard-ai, and verify the original repository identity, currently AAA55555/geo-guard-ai. Keep feature branches locally and remotely for upstream review. Do not delete them, close their PRs, rewrite their history, or force-push merely because their changes have landed in the fork.
2. Merge verified feature changes into the fork's main in the table's order, retaining ancestry through ordinary merges or fast-forwards. At the end of PR 11, audit that every intended feature tip is included and merge any outstanding changes in order. This is a reconciliation step if earlier fork merges already happened; do not duplicate those merges or wait for upstream acceptance. Record branch tips, merge commits, PR message paths, and upstream PR links only when the user has opened them.
3. Run the complete task-24 checks against the combined PR 01-11 commit. Only after those stages succeed, implement task 25 and prepare its release-automation message for the user to open as the final upstream PR. Keep the intended upstream diff limited to release automation, necessary CI reuse/packaging-test support, and documentation. Do not propose the fork's accumulated main as the PR head if that would include all unmerged feature changes. Prepare a separate upstream-facing branch when needed, identify prerequisites, and recommend draft status in the handoff if the upstream base cannot yet exercise the required contracts; report fork and upstream validation separately. The user creates the PR, including any draft.
4. Merge the release-automation change into the fork's main without waiting for the original maintainer. Commit any distinct fork version before final verification, keeping fork-specific version metadata out of the upstream automation PR. Re-run the full integration matrix and workflow validation against this final combined SHA. Task 25's release workflow must build, test, and target that immutable revision rather than a moving main reference.
5. Hand over the verified SHA, retained branches, all PR message files, any existing user-opened PR links, workflow instructions, and an installable npm tarball. Local installation and use do not require an upstream PR to have been opened or merged, or npm registry publication. The workflow is manually invoked to prepare a GitHub draft release; do not dispatch it or publish a release merely because the implementation changes merged. Record any unavailable live platform/host or release-workflow checks instead of claiming them passed.

## Files and interfaces

1. Keep the entry point in bin/geo-guard.js and routing in src/cli.ts.
2. Existing owners remain: config.ts for config storage/layering; setup.ts for setup orchestration; uninstall.ts for cleanup; status.ts for diagnostics; run.ts for checks/wrapping; geo.ts for provider access; i18n.ts for English/Russian text.
3. Add src/json-file.ts for the narrow atomic JSON writer in task 6. It must not become a general filesystem framework.
4. Add src/shells.ts for shared shell/rc-path definitions and src/targets.ts for live command-target definitions in task 12. Leave legacy alias parsing and migration in shell-alias.ts.
5. Add src/ip.ts for IP/CIDR normalization and matching, and src/policy.ts for mode-aware policy evaluation in task 16. IP parsing must not import config, avoiding a dependency cycle.
6. Add src/codex-hook.ts for the Codex hook-file adapter in task 18.
7. Add src/installation-state.ts for the small expected-component record used by task 19.
8. Add test/helpers/sandbox.cjs only for shared test isolation used by multiple suites.
9. Test additions may be focused files such as test/ip.test.cjs, test/policy.test.cjs, test/json-file.test.cjs, and test/codex-hook.test.cjs. Existing equivalent tests should be extended rather than duplicated.
10. Task 25 adds .github/workflows/release.yml and makes .github/workflows/ci.yml reusable. Reuse existing build/package tests, extending scripts/pack-check.sh to accept an existing tarball; do not add a second build system or an npm-publishing pipeline.

## Review focus

1. A path containing shell metacharacters is data, never shell code; an edited managed block must preserve its user additions.
2. Mixed country-only and IP-only profiles must not become more permissive when the hook host cannot be identified.
3. Silence must survive invalid JSON and network failure; it must never turn a block into an allow.
4. A Codex UserPromptSubmit event must not accidentally select the Claude profile.
5. A successful tarball install with scripts disabled must still include every required runtime file.

## Numbered implementation tasks

1. **Make generated PATH initialization restore shim priority.**

   **Files:** src/shell-path.ts, src/windows-path.ts, test/shell-path.test.cjs, test/windows-path.test.cjs, scripts/docker/login-file-check.sh, scripts/windows-smoke.ps1.

   **Required behavior:** For a managed shim directory S, the resulting PATH starts with exactly one S. Remove equivalent occurrences of S from their old positions and preserve every other entry in its original order. Preserve unrelated duplicates and existing empty PATH entries; do not invent an empty entry. An unset PATH becomes S. A set-but-empty POSIX PATH retains its existing empty entry after S.

   POSIX comparisons are case-sensitive. Treat a trailing slash on S as equivalent except for root. Windows comparisons are case-insensitive, accept slash/backslash equivalence, and ignore a trailing separator except for roots. Do not resolve unrelated entries or symlinks while rewriting PATH.

   Generate correct behavior for bash, zsh, fish, and PowerShell. Apply equivalent move-to-front behavior to the stored Windows user PATH, preserving its registry value type. Repeated sourcing must be idempotent. Do not claim that moving the user PATH entry changes Windows machine-PATH ordering.

   **Acceptance cases:** S absent; first; middle; last; duplicated; equivalent trailing separator; empty PATH; unset PATH; unrelated duplicate and empty entries; Windows case variants. Execute the generated snippets, not just substring assertions.

   Reproduce the actual startup sequence with a temporary home: .profile sources .bashrc; .bashrc adds S; .profile prepends another directory; the .profile managed block runs. Put a fake real claude in the competing directory. Final command resolution must select the shim. Pair migration/rewrite verification with tasks 2 and 4 before merging PR 03, not only before release.

2. **Classify edited PATH blocks conservatively.**

   **Files:** src/shell-path.ts, src/shell-alias.ts where shared block operations require adjustment, test/shell-path.test.cjs, test/setup.test.cjs.

   A block is generated only if its complete body matches a supported complete template for its recovered directory. Preserve explicit recognition of the existing shipped templates when task 1 introduces new templates. Do not identify ownership from one matching PATH assignment.

   Preserve unknown lines inside a marked block, missing end markers, ambiguous duplicate marker pairs, and unknown templates byte-for-byte. Report that setup/uninstall left the block for manual handling. Do not implement force-overwrite of foreign content.

   Removing a managed block must not globally normalize blank lines elsewhere in the file. Preserve text outside the removed block; remove only a separator attributable to the package.

   **Acceptance cases:** A valid old block migrates once; a new block remains unchanged on repeat setup; an old block plus export USER_SETTING=keep is preserved by setup and uninstall; a comment or command added inside the block is preserved; broken/duplicate markers are preserved; outside whitespace remains unchanged. Check content and modification time where no change is required.

3. **Recognize only supported hook invocations as package-owned.**

   **Files:** src/hook-shared.ts, src/claude-hook.ts, src/cursor-hook.ts, test/hooks.test.cjs, test/hook-invariants.test.cjs. Extend the same rules to codex-hook.ts in task 18.

   Keep recognition of a direct geo-guard check invocation, supported executable suffixes/paths, the legacy geo-check script forms already tested, and valid package-added arguments such as --quiet and --profile codex. Treat the entire command as a supported invocation rather than searching arbitrary shell text for a substring.

   Commands that merely mention the program, or combine it with another command, are not automatically package-owned. Do not evaluate shell code or add a general shell parser. If a shell wrapper cannot be confidently recognized, preserve it and report the ambiguity.

   **Acceptance cases:** Positive cases include the current bare command, geo-guard.cmd check, the tested old absolute geo-check.sh path, quiet mode, and the Codex profile suffix. Negative fixtures must include echo geo-guard check, echo geo-check, my-geo-check.sh, a quoted string containing the command, and a compound invocation followed by another command.

   Do not use isOurHook itself as the oracle defining foreign fixtures in every invariant test. Include explicitly labeled foreign entries and assert they survive unchanged.

4. **Quote literal paths in generated shell code.**

   **Files:** src/shell-path.ts, src/shim.ts, test/shell-path.test.cjs, test/shim.test.cjs, scripts/windows-smoke.ps1.

   Quote executable and directory paths using the rules of the shell that executes them. Paths are literal data. User-supplied extra-argument strings retain their existing documented shell-syntax meaning; do not apply the literal-path transformation to those strings.

   Update read-back parsers and template recognition together with generators, retaining compatibility with old generated files. Continue to preserve custom shim flags and protect foreign executables.

   **Acceptance cases:** Execute generated code using temporary paths containing spaces, an apostrophe, a dollar sign, a double quote where supported, and parentheses. On POSIX, include backslashes and a literal command-substitution-looking filename. Assert exact selected path and no unintended side-effect marker file. On Windows, use only characters legal in Windows paths and add percent/exclamation cases.

   Reject unsupported control characters before writing a generated script; do not silently change a path.

5. **Preserve arguments through Windows batch wrappers.**

   **Files:** src/run.ts, src/shim.ts if its batch invocation needs coordination, test/check.test.cjs, scripts/windows-smoke.ps1.

   Preserve native direct spawning for executables. Use the necessary batch-file path only for .cmd/.bat on Windows. Treat argv values received by Node as already-parsed argument data; the caller's shell must not be asked to parse them a second time.

   Replace quoting based solely on whitespace/double quotes with a Windows-tested transport/escaping strategy. It must handle paths to both Node and the target containing spaces and retain target exit codes. Preserve recursion protection and signal behavior.

   **Acceptance cases:** A temporary npm-style batch wrapper invokes an argv-echoing Node fixture. Compare JSON argv arrays for empty strings, spaces, embedded quotes, trailing backslashes, Unicode, &, |, <, >, ^, parentheses, %NAME%, and !NAME!. Use a harmless sentinel to prove metacharacters did not execute a second command. Invoke the wrapper shell with delayed expansion disabled. The target fixture must not explicitly enable delayed expansion or deliberately transform its arguments; transformations performed by a target script itself are outside the transport's guarantee.

   Do not replace these tests with tests that assert the escaping helper's own string representation.

6. **Use atomic, no-op-aware JSON persistence.**

   **Files:** new src/json-file.ts; src/config.ts; src/claude-hook.ts; src/cursor-hook.ts; new test/json-file.test.cjs; existing hook/config tests.

   **Interface:** writeJsonFile(file: string, value: unknown, options?: { backup?: boolean }): { changed: boolean }. Callers retain schema validation and merge decisions.

   Serialize with the existing two-space indentation and trailing newline. If existing serialized bytes already equal the proposed bytes, return changed=false without writing, renaming, or making a backup. On a real change, write a uniquely named same-directory temporary file and rename it over the destination. Clean up the temporary file after failure.

   Hook callers request the existing one-time .bak behavior; config callers do not start making new backups as an unrelated change. Never overwrite an existing backup. Preserve existing file permissions where the platform supports that operation.

   Preserve an existing symlink rather than replacing the link with a regular file: resolve and update its target, or fail clearly if that cannot be done safely. Do not silently fall back to truncate-and-write after an atomic-write failure.

   **Acceptance cases:** Create; change; unchanged bytes/mtime; existing backup retained; simulated write failure leaves old destination intact; rename failure cleans up its temporary file; POSIX file-mode and symlink cases. Atomic replacement does not promise a multi-file transaction or concurrent-writer conflict resolution.

7. **Validate configuration without turning malformed policy into defaults.**

   **Files:** src/config.ts, src/config-cmd.ts where error handling is needed, src/i18n.ts, test/config.test.cjs, test/check.test.cjs.

   Validate known fields after JSON parsing and before layering/writing. Country lists accept a comma-separated string or array of strings; normalize case and deduplicate. Wrong scalar types, non-string members, and invalid country-code shapes are errors. Keep existing shape validation rather than adding an ISO-country database.

   Preserve legacy null-as-inherit behavior for existing optional policy fields and profile sections where currently supported. Missing values inherit. New fields introduced by task 16 or 17 reject explicit null unless their contract below explicitly allows it.

   Profiles must be an object or legacy inherited/null value; known profile entries must be objects or legacy inherited/null values. Preserve unknown keys without applying them. Validate non-null file timeoutMs as a finite positive number, and provider lists as arrays of nonempty valid HTTP/HTTPS URLs. Local HTTP test endpoints remain supported. Preserve the documented empty/invalid timeout-environment fallback behavior.

   File/env empty country or IP allowlists are deliberate deny lists, not defaults. Empty provider lists mean unavailable detection, not fallback providers. CLI list setters continue rejecting an explicitly empty list. Explicit whole-config reset remains the recovery operation that may replace invalid JSON.

   **Acceptance cases:** allowed:42 errors rather than permitting NL; mixed valid/invalid list errors; existing allowed:null inheritance passes; wrong profile shapes error; unknown fields survive ordinary writes; each precedence layer selects the expected value; failed validation leaves bytes unchanged. Every check-path error blocks with exit 2.

8. **Recover hook identity only from a trustworthy top-level field.**

   **Files:** src/hook-payload.ts, src/run.ts, test/check.test.cjs.

   For a complete valid JSON object, use only its top-level hook_event_name. Recover a partial prefix only when the reader knows it stopped at the byte cap or time budget, not after ordinary EOF with invalid JSON. For that explicitly truncated input, allow a small bounded top-level-field scanner that distinguishes JSON strings and nesting. It may recover a complete top-level event string from a syntactically valid object prefix; arbitrary malformed surrounding text, nested fields, and a completed object followed by garbage must not select a profile.

   Store at most 1,048,576 bytes. When a chunk crosses the limit, retain only the remaining permitted bytes. Keep the existing 1,000 ms read budget. Remove listeners/timers on every finish/error path. A terminal or true empty EOF means no host; bytes from an unidentified host mean unknown host. A timed-out pipe with no EOF must not be treated as positively identified manual input.

   **Acceptance cases:** Complete Claude/Cursor payloads; explicit Codex profile in task 18; event split across chunks; a valid prefix in a payload over 2 MiB; event string nested in another object; escaped event-looking prompt text; garbage containing a quoted event field; truncated field; invalid JSON ending at normal EOF; a completed object followed by junk; unknown event; stdin error; open stdin; empty EOF. PR 05 retains the existing country-only strictest-policy behavior for unknown hosts. PR 08 replaces it with task 16's mode-aware conjunction in the same PR that exposes IP policies; neither stage may use a permissive fallback.

9. **Preflight actual setup targets and report partial failures honestly.**

   **Files:** src/setup.ts, src/shell-alias.ts or shells.ts after task 12, src/shim.ts, src/i18n.ts, test/setup.test.cjs.

   Resolve all intended destinations first: overridden config path, selected hooks, shim directory, selected shell rc files, actual bash login file, and alias-migration files that would change. Validate inputs and existing owned-file shapes before the first write.

   PR 06 preflights the currently supported artifacts. Extend this same preflight in PR 08 for mode/IP inputs, PR 09 for message choices, PR 10 for Codex destinations, and PR 11 for installation metadata. Do not import future modules or create placeholder artifacts in PR 06. New gates and metadata must never be added without their preflight in the same PR.

   A missing file is different from an unreadable file. Treat only ENOENT as absent; do not overwrite content after EACCES, EPERM, EISDIR, or other read errors. Preflight should inspect the nearest existing parent instead of creating directories while merely validating.

   Writability probes are not guarantees. If a later operation fails, stop dependent operations, report which changes succeeded and what remains, and return exit 1. Do not print the normal setup-complete message. Do not promise rollback of arbitrary user files or add a generic transaction framework.

   A requested gate that cannot be installed because foreign or unrecognized content was protected is incomplete setup, not successful protection. Preserve that content, print the specific corrective action, return exit 1, and do not save a new successful installation-intent record.

   **Acceptance cases:** Bad country/IP input; malformed later hook file; unreadable existing shim; unwritable bash login destination; nested missing parent; simulated failure after one successful write; Windows ACL denial at write time. Invalid-input and preflight failures leave the sandbox unchanged.

10. **Make uninstall independent, repeatable, and truthful.**

    **Files:** src/uninstall.ts, relevant removal helpers, src/i18n.ts, test/setup.test.cjs, test/hooks.test.cjs, scripts/windows-smoke.ps1.

    Attempt cleanup of each independent hook file, each shim, each rc file, user PATH entry, configuration, and installation-state record even if a previous independent step failed. Do not erase foreign content to achieve a clean-looking result.

    PR 06 implements cleanup for existing artifacts. PR 10 adds Codex cleanup and PR 11 adds installation-state cleanup in the same PRs that create those artifacts; the final acceptance cases below accumulate as support is introduced.

    Collect errors and print a final partial-cleanup report. Return exit 1 when cleanup encountered an operational failure; preserve exit 0 for a complete cleanup and an already-clean installation. Existing protected foreign/custom content is reported, not reclassified as an operational error solely because it was intentionally retained.

    --quiet suppresses routine progress, not operational errors. --keep-config retains config.json but still removes installation-state metadata. Parse arguments before writes and reject unknown flags and stray positional arguments; explicitly document or retire the old ignored --shell compatibility spelling rather than silently consuming arbitrary input.

    **Acceptance cases:** Malformed Claude JSON does not prevent removing the Cursor/Codex hook or owned shims; unreadable rc does not stop other rc cleanup; keep-config behavior; foreign files; custom/broken blocks; retry after partial failure; second uninstall no-ops; quiet failure still reports errors.

11. **Remove obsolete lifecycle scripts and only their orphaned code.**

    **Files:** package.json; delete bin/postinstall.js and bin/preuninstall.js; src/uninstall.ts; src/i18n.ts; scripts/pack-check.sh comments as applicable.

    Remove the postinstall and preuninstall script entries. Remove runPreuninstall and its now-exclusive preuninstallError message type/catalog entries. Remove postinstallHint type/catalog entries. Keep runUninstall and the CLI's --quiet cleanup option.

    Implement this task together with task 23 in PR 02. Keep prepare/build while task 23 establishes development and Git-install behavior in that same PR. Preserve existing user changes to dependency versions and the lockfile. Update lockfile lifecycle metadata only if the npm lockfile actually requires it; do not perform an unrelated lockfile rewrite.

    **Acceptance:** Repository search shows no executable references to removed hooks, scripts, or message keys. Historical changelog entries may remain as history. The published entry point still loads the compiled CLI. Package consumers need neither hook approval nor a compiler.

12. **Separate shared shell and live-target definitions from legacy aliases.**

    **Files:** new src/shells.ts and src/targets.ts; src/shell-alias.ts; src/shell-path.ts; src/shim.ts; src/setup.ts; src/status.ts; affected tests.

    shells.ts owns ShellName, the single supported-shell list, shell detection/normalization, and rc-path discovery. shell-alias.ts retains legacy alias marker parsing, classification, migration, and removal. Keep shared marked-block operations in a clearly shared location or existing small helper; do not duplicate them.

    targets.ts owns live command target identity, command spelling, and profile mapping for Claude and cursor-agent; task 18 adds Codex there. Shim targets must no longer be aliases by type definition. Legacy alias metadata extends/associates with the applicable live targets; do not invent legacy Codex aliases.

    Preserve current shell selection, GEO_GUARD_RC behavior, bash login-file precedence, saved shim arguments, recursion protection, and foreign-file rules. Avoid runtime import cycles.

    **Acceptance:** Existing shell/alias/shim suites continue passing without changing their behavior expectations. Each supported shell and live target has one authoritative definition.

13. **Remove confirmed unused helpers without expanding the cleanup.**

    **Files:** src/args.ts, src/shell-alias.ts or its post-task-12 location.

    Search the current code and tests for callers of inlineValue and rcPathForShellResolved. Remove either only if it is still unused. If an earlier task gave one a real caller, keep it and explain that in the task report.

    Do not remove migration code merely because modern installations do not create aliases. Do not delete helpers used by meaningful tests as an incidental export-count cleanup.

    **Acceptance:** Typecheck passes, imports resolve, and no references to deleted exports remain. No dedicated deletion-only tests are necessary.

14. **Cancel provider work that can no longer affect the decision.**

    **Files:** src/geo.ts, test/core.test.cjs; extend to the new IP race in task 16.

    Retain first-valid-response behavior within a country-provider race. Failed, malformed, or non-OK responses do not win. Once a winner exists, abort remaining requests and clear their timeout resources. An all-failed race returns unavailable.

    If fetchCountry needs a parent AbortSignal, add it as an optional parameter so existing two-argument calls retain their meaning. Relay cancellation and clean up relay listeners using APIs supported by the declared Node floor.

    Only cancel work that is no longer required. A successful country result in both mode must not cancel the required IP check.

    **Acceptance:** Slow first provider does not delay a faster valid result; invalid fast response does not beat a valid slow response; losing fetch receives cancellation; all failures return unavailable; separate invocations perform fresh requests; no unhandled rejections or lingering timers keep a subprocess alive.

15. **Bound Windows registry-related subprocesses.**

    **Files:** src/windows-path.ts, test/windows-path.test.cjs, scripts/windows-smoke.ps1.

    Give each PowerShell registry subprocess a 5,000 ms timeout and a 1 MiB output limit. Preserve raw registry PATH values, their value names, and String/ExpandString types.

    Map timeout, spawn failure, nonzero exit, invalid output, and excess output into the existing unavailable/error result path. Do not retry with a more destructive registry API or flatten machine PATH into user PATH.

    **Acceptance:** Simulated hanging/error subprocesses return unavailable within the configured bound; add/remove/move-to-front preserve unrelated entries and registry type; a Windows smoke test restores the original account PATH in finally. Ordinary unit tests do not touch the real registry.

16. **Add exact-IP/CIDR policies with explicit mode and inheritance semantics.**

    **Files:** new src/ip.ts and src/policy.ts; src/config.ts; src/config-cmd.ts; src/setup.ts; src/geo.ts; src/run.ts; src/status.ts; src/i18n.ts; new test/ip.test.cjs and test/policy.test.cjs; existing config/setup/check/status tests; scripts/e2e.sh; scripts/pack-check.sh; scripts/windows-smoke.ps1; README.md; README.ru.md; CHANGELOG.md.

    **PR 08 scope:** Land the full policy path together with task 14, the applicable help/documentation from tasks 20-21, and installed-binary coverage from task 24. Both prompt checks and launch wrappers use evaluatePolicy. Make status evaluate/report the effective mode and applicable allowlists rather than always running a country check. Exercise IP-only status with unavailable country providers, and verify that existing country-only configuration retains its prior behavior. Installation-intent metadata remains task 19; accurate reporting of these policies does not wait for it.

    **Configuration contract:** Keep allowed as the existing country field. Add mode: 'countries' | 'ips' | 'both', allowedIps: string[], and ipProviders: string[]. The resolved defaults are mode='countries', allowedIps=[], and ipProviders=['https://api64.ipify.org']. Existing country defaults and providers remain unchanged.

    Allow mode, allowedIps, and ipProviders in known profile sections. Apply the existing precedence independently to each field: scoped environment, shared environment, profile file, shared file, default. Use GEO_GUARD_MODE, GEO_GUARD_IPS, and GEO_GUARD_IP_PROVIDERS, plus _CLAUDE/_CURSOR suffixes; PR 10 extends the same convention to _CODEX when task 18 registers that profile. Do not advertise Codex support in PR 08. Empty IP/provider environments mean empty lists; empty/unknown mode is an error. New fields do not accept null.

    **CLI contract:** setup and config accept --mode countries|ips|both and --ips LIST, including --flag=value forms. IP LIST is comma-separated; accept normalized comma-separated strings or string arrays when reading the file. config's existing --profile scopes these new policy setters. setup's new policy flags target the shared policy; use config --profile for per-tool mode/IP overrides. Do not silently infer mode from the existence of an IP list.

    An existing setup without a new mode/list keeps them. Fresh interactive setup offers countries as the default choice, ips as the alternative, and a separate combine-both question defaulting to no. The resulting both mode is the only AND switch; do not add a redundant second boolean. Prompt only for lists needed by the selected mode, offering existing values. Noninteractive ips/both requires a nonempty applicable saved or supplied IP list before writes; both also requires a nonempty applicable country list. Do not automatically trust the current IP. Config changes must not rewrite hooks or shell files.

    **IP normalization:** Export normalizeIpEntry(raw: string): string and ipMatches(ip: string, entries: readonly string[]): boolean from ip.ts. Validate addresses with Node's IP facilities and use numeric address bits for matching. Do not compare CIDR strings or IPv6 text prefixes. Trim entry whitespace, reject hostnames, ports, bracketed URL notation, zone identifiers, wildcards, address ranges other than CIDR, and out-of-range prefix lengths. Prefix ranges are 0..32 for IPv4 and 0..128 for IPv6. Normalize host bits to the network address and deduplicate equivalent entries without merging overlapping networks. Store IPv6 in a deterministic lowercase compressed form. IPv4 and IPv6 are separate families, including IPv4-mapped IPv6; do not silently equate them.

    **Detection:** Add detectPublicIp(config: GeoGuardConfig): Promise<string | null> in geo.ts. IP providers return a plain-text address, trimmed and validated; country providers continue returning country text. Race valid IP-provider answers using task 14's cancellation rules. ips mode never contacts country providers. countries mode never needs an IP-provider result. Empty required providers, invalid responses, timeout, and network failures block the relevant mode.

    **Decision interface:** policy.ts exports evaluatePolicy(config: GeoGuardConfig): Promise<PolicyDecision>. PolicyDecision includes allowed:boolean, observed country/IP when available, and a stable denial category: country-unavailable, country-denied, ip-unavailable, or ip-denied. Configuration exceptions remain errors handled by the caller. For both, start both required checks concurrently so their default timeouts overlap; use country denial before IP denial only to choose deterministic diagnostic text, never to bypass either predicate.

    | Country check | IP check | countries | ips | both |
    | --- | --- | --- | --- | --- |
    | allow | allow | allow | allow | allow |
    | allow | deny/unavailable | allow | block | block |
    | deny/unavailable | allow | block | allow | block |
    | deny/unavailable | deny/unavailable | block | block | block |

    **Unknown host:** Replace the country-list-only strictest-policy shortcut with conjunction of the effective shared policy and each configured known profile policy. Return allow only if every relevant policy allows. Do not intersect CIDR strings or discard a policy because its mode differs. Read config once per invocation. Keep explicit --profile precedence over event inference.

    **Acceptance cases:** Exact v4/v6; equivalent expanded/compressed IPv6; /32 and /128; /0; subnet boundary in/out; host-bit normalization; invalid entries; separate families; empty lists; all table cells; file/env/profile precedence; profile reset/unset; no writes on invalid setup; GeoIP country changes for an unchanged allowed IP still allow in ips mode; the same change blocks both mode when country is no longer allowed; mixed-profile unknown host blocks if any policy blocks.

17. **Make prompt-hook text configurable without changing enforcement.**

    **Files:** src/setup.ts; src/config.ts; src/cli.ts; src/run.ts; src/hook-shared.ts; existing hook adapters; src/status.ts if its hook inspection needs adjustment; src/prompt.ts only if necessary for an explicit choice; src/i18n.ts; test/check.test.cjs; test/setup.test.cjs; test/hooks.test.cjs; affected status tests; packaged-workflow scripts; README.md; README.ru.md; CHANGELOG.md.

    **PR 09 scope:** Configuration, early quiet parsing, hook command generation/recognition, error handling, and output completion land together. Existing status must recognize installed quiet hooks correctly. Include help/documentation and packaged default/custom/silent cases in this PR; extend Codex's adapter when it is introduced in PR 10. Keep setup/status/uninstall output available as specified below.

    Add optional top-level hookMessage:string. Missing means existing localized diagnostic text; a nonempty string is literal replacement text; exactly empty means silent. It is global, not a profile field. Do not implement templates, shell expansion, or placeholders. Preserve supplied text, including whitespace; do not evaluate backslash escapes. Emit custom text without a prefix, adding one final LF only if the configured string does not already end with LF. Whitespace-only text is not the empty-string sentinel.

    setup accepts --hook-message TEXT, including --hook-message=, and --default-hook-message to remove the override. These two options are mutually exclusive. If neither is supplied, repeat/noninteractive setup preserves the existing setting. Interactive setup offers keep/default/custom/silent explicitly so pressing Enter to keep is not confused with requesting silence. A custom-text read must not accidentally apply the current default when the user selects silence.

    Add check --quiet as a command-level override. Parse it before reading configuration or producing diagnostics. In silent setup, the generated Claude and Cursor hook commands both become exactly geo-guard check --quiet; their byte-for-byte equality must remain true. In normal mode they retain exactly geo-guard check. Task 18's Codex command adds the same quiet suffix after its explicit profile.

    An empty message selects quiet commands during setup so a later malformed or unreadable config cannot re-enable human-readable errors. Remove statusMessage from the package-owned hook handler when silence is explicitly chosen, including a previous custom statusMessage on that owned handler. Preserve unrelated handlers, timeouts, and other user fields.

    In quiet mode, every denial/error produces no package text on stdout or stderr and exits 2. On success, preserve the exact existing non-TTY control JSON {"continue":true}, with no trailing newline, and exit 0. Suppress interactive success confirmations as well when --quiet is explicit. The control JSON is protocol, not a human message. Normal custom/default mode prints its selected block text on stderr, never as plain-text stdout. Returning from silent mode to default mode restores the adapter's default status text where one normally exists.

    The top-level CLI catch must not print a second diagnostic after a quiet check error. Silence affects prompt-hook execution only; setup/status/uninstall diagnostics and launch-wrapper messages remain available.

    Do not terminate the process before required stdout/stderr writes have completed or reported failure. A stream-write failure must preserve a blocking exit 2, not fall through to a generic exit 1. Include subprocess assertions for complete small output as well as quiet output; do not make a dropped-output bug appear to satisfy silence.

    **Acceptance cases:** Default message retains existing locale behavior; custom literal text; empty text; explicit quiet with malformed JSON, unreadable file, invalid profile, unknown check option, denied country/IP, unavailable providers, and unexpected thrown error. Assert exact stdout, stderr, and exit code. Assert hook command equality and absence of owned statusMessage in silent mode.

    This controls package-emitted text. A host may display its own generic block/error UI; do not promise that the package can suppress host-generated notices.

18. **Integrate the Codex CLI using its actual blocking hook.**

    **Files:** new src/codex-hook.ts; src/targets.ts; src/config.ts; src/setup.ts; src/run.ts; src/status.ts; src/uninstall.ts; src/i18n.ts; test/codex-hook.test.cjs; existing shim/config/check/setup/status/hook tests; packaged-workflow scripts and Windows smoke coverage; README.md; README.ru.md; CHANGELOG.md.

    **PR 10 scope:** Add profile/target registration, setup preflight, hook/shim installation, enforcement, existing status integration, and independent removal together. Include mode/IP inheritance, explicit-profile selection, quiet hooks, CLI help, documentation, and packaged integration cases. Basic status must distinguish file presence from unverified host trust now, without importing task 19's future metadata module. Task 19 refines installation expectations and diagnostics rather than making Codex usable for the first time.

    **Verified host facts:** Official OpenAI documentation describes UserPromptSubmit, user-level hooks.json, blocking via exit 2, and review/trust of changed non-managed hooks. Use synchronous execution. Do not use PreModelRequest, background hooks, or notification callbacks as the gate. The installed CLI reported version 0.167.2 during drafting. Recheck release behavior at execution time; do not invent a minimum supported version or claim an untested version works. [OpenAI Hooks documentation](https://learn.chatgpt.com/docs/hooks)

    **Integration decisions:** Add codex to known policy profiles and to live shim targets. Generate geo-guard check --profile codex, with --quiet appended in silent mode. Explicit profile selection avoids confusing its event name with Claude's. Merge only the owned handler into the user hooks JSON; preserve other events, fields, custom settings, and existing backups. Honor the configured Codex state root rather than always assuming the default. [Codex environment variables](https://learn.chatgpt.com/docs/config-file/environment-variables)

    setup adds --codex/--no-codex for its prompt hook, --codex-shim/--no-codex-shim for launch gating, --codex-args for saved shim arguments, and --codex-countries for parity with existing setup country overrides. Defaults: install the hook when the Codex home already exists; install the shim only when a real Codex binary is discoverable excluding owned shims. Explicit positive flags may override discovery. --no-shim applies to all targets unless an explicit target-positive flag overrides it, following existing cursor-agent behavior.

    Codex home resolution: use a nonempty inherited CODEX_HOME if present, otherwise the user's .codex directory. Do not change CODEX_HOME as a scratch variable. If an explicitly configured root does not exist, report that before writes rather than silently falling back. For a default root absent on a machine, --codex may create it. No project-local Codex config, trust database, or managed-policy file is rewritten by setup.

    Print the supported /hooks review instruction after creating or changing the handler. Do not add a trust-bypass flag or silently manipulate trust. Manage the owned handler only in the user hooks JSON and prevent duplicates within that file. Leave inline TOML and other hook sources untouched; explain that independently configured sources can also run and that /hooks is the supported place to inspect them. Do not promise cross-source deduplication or parse/rewrite arbitrary TOML using string replacement.

    **Acceptance cases:** Correct profile despite UserPromptSubmit naming collision; preserve foreign JSON/events; install twice gives one owned handler; quiet hook command; native and Windows launch shims; recursion prevention; absent Codex is skipped unless requested; custom state root; independent removal; live-host integration on a recorded supported CLI version using a fake backend confirms a denied prompt never reaches that backend. If a required live-host test is unavailable, report the integration unverified.

19. **Track installation intent and define status precisely.**

    **Files:** new src/installation-state.ts; src/setup.ts; src/status.ts; src/uninstall.ts; src/config-cmd.ts only to preserve reset behavior; src/i18n.ts; test/status.test.cjs; test/setup.test.cjs.

    Store installation.json beside configPath(), using a versioned format with booleans for claudeHook, cursorHook, codexHook, claudeShim, cursorShim, and codexShim, plus the selected shells and shim directory. Export installationPath():string, readInstallationState():InstallationState|null, and writeInstallationState(state:InstallationState):void. Use task 6's writer. An unsupported or invalid state record is reported, not overwritten as though missing.

    Introduce metadata only in PR 11, extending the preflight established by task 9 and the cleanup established by task 10 in this same PR. Read and validate existing metadata during preflight and include its write destination among the planned targets. Save the expected-component record only after successful setup. Do not mark failed or intentionally protected installation steps successful. Config policy changes and config --reset do not edit this installation-intent record. Uninstall removes it even with --keep-config. Earlier PRs retain discovery-based installation expectations, with the feature-aware reporting specified in tasks 16-18.

    status reports expected-installed, deliberately-skipped, missing/broken, and unavailable/unknown states distinctly. A negative setup flag alone does not prove an existing component is disabled. Inspect actual owned artifacts even when not expected, report present-but-not-requested components, and do not hide a stale gate behind a skip flag. For old installations with no record, retain legacy discovery expectations and label the expectation source as inferred; status itself never migrates or writes metadata.

    Exit 0 means required installation structure is present and no inspected component has a known structural failure. Exit 1 means a required component is missing/broken, configuration/metadata cannot be read, or required structural state cannot be inspected. A denied country/IP is a policy result, not an installation failure.

    Report Codex hook trust/effectiveness separately from file presence. If no supported read-only interface can verify trust, say unverified and show the review instruction; do not read private trust storage or claim prompt protection is active. That trust-unknown annotation alone does not change the structural exit code to failure. Wording must explain that structural exit 0 is not proof of host trust.

    Show effective mode, country/IP allowlists, value sources, and per-profile policy results. Report every section even if another one errors. Do not classify a policy change as a broken install.

    **Acceptance cases:** Hook-only; shim-only; deliberately disabled components; stale artifacts; new/missing/invalid metadata; pre-upgrade install; each new policy mode; independent profiles; untrusted/unverified Codex hook; malformed one section does not hide later sections. Snapshot files, mtimes, and backups to prove status is read-only.

20. **Make bare invocation and help useful, stable, and side-effect-free.**

    **Files:** src/cli.ts; src/i18n.ts; test/core.test.cjs; new or existing CLI/help subprocess tests.

    **Incremental ownership:** PR 02 establishes the existing-command quick start, all four help forms, explicit cleanup guidance, and side-effect-free help tests. Until PR 08, use country-only wording in the setup quick start. PR 08 introduces the final countries-or-IPs wording below and the mode/IP options; PR 09 adds message/quiet options; PR 10 adds Codex options. Each PR keeps both languages accurate and all four forms on the same renderer. PR 11 checks completeness; it must not be the first PR to document earlier flags.

    Keep geo-guard, geo-guard help, geo-guard --help, and geo-guard -h on one help renderer. Show quick-start guidance near the top, then complete command and flag information. Do not automatically start setup or require a working config.

    The English quick start must include:
    ~~~text
    Getting started:
      geo-guard setup       Configure allowed countries or IPs and enable checks
      geo-guard status      Inspect the installation and effective policy

    Before removing the package:
      geo-guard uninstall
      npm uninstall -g geo-guard-ai
    ~~~

    Provide equivalent Russian text. Document --mode, --ips, exact-IP/CIDR syntax, --profile codex, message/default/quiet flags, and Codex setup flags with the same spellings as the parser. Remove the claim that uninstall --quiet exists for npm preuninstall; keep its script-use explanation.

    **Acceptance cases:** All four help forms exit 0 in a clean home and with malformed config, closed stdin, unreachable providers, and absent tools. Assert no prompts, provider requests, file writes, metadata updates, or backup creation. Check locale flag parity and key instructions, not a brittle snapshot of the entire help string.

21. **Bring documentation and examples into agreement with the implemented contracts.**

    **Files:** README.md, README.ru.md, CHANGELOG.md.

    **Incremental ownership:** Update both languages and relevant release notes in the PR implementing each behavior. PR 02 covers installation versus activation, contributor setup, and explicit uninstall/recovery; PRs 03-04 cover PATH and Windows behavior; PRs 05-07 cover any changed validation/maintenance contracts; PR 08 covers policy modes and IP/CIDR; PR 09 covers messages; PR 10 covers Codex and trust; PR 11 covers installation intent and final structural status. The requirements below describe the final combined documentation, not permission to defer it all to the end.

    Update both language versions in the same change. Explain CLI installation versus setup activation; new PATH move-to-front behavior; modes and the AND truth table; IPv4/IPv6/CIDR normalization and family rules; environment/profile precedence; IP-only independence from GeoIP; literal/custom/silent hook text; config-error silence via generated quiet commands; Codex installation/review and verified coverage.

    Give concrete commands for countries mode, ips mode, both mode, profile-scoped IP changes, silent setup, restoring default messages, status, and two-step uninstall. Use documentation-only IP ranges such as 192.0.2.0/24 and 2001:db8::/32; do not present them as usable current external IPs or auto-approved defaults.

    Correct the Cursor recovery paragraph so it never implies npm uninstall alone removes hooks. Preserve recovery instructions for hooks/shims left behind when the binary is already gone. Explain that init-time PATH promotion cannot stop another tool from rewriting PATH later.

    State what structural status can and cannot verify about Codex trust. Do not claim complete model-request, IDE, or resumed-session coverage beyond the tested integration.

    **Acceptance:** Every documented command maps to an implemented parser path and an example fixture or exercised workflow; English/Russian option names match; release notes describe final behavior. Historical release notes remain historical.

22. **Make test environments independent of the developer's machine.**

    **Files:** new test/helpers/sandbox.cjs; affected test/*.test.cjs; test/fixtures/mock-fetch.cjs; scripts/e2e.sh; scripts/pack-check.sh; scripts/windows-smoke.ps1.

    **PR 01 scope:** Start with the current commands, files, policies, provider fixtures, and installed targets; reproduce the recorded baseline before changing fixtures. Add tests that show any inherited-environment defect before fixing it. Keep this PR free of new runtime features. Add isolation for new IP fields, message options, Codex state, and installation metadata with PRs 08-11 when their corresponding behavior exists.

    Share small helpers only where multiple suites need them. Build child environments from a controlled copy: remove inherited GEO_GUARD_* overrides before applying fixture values, replace inherited NODE_OPTIONS with deliberately quoted fixture preloads, and redirect all config/shim/rc/user-home writes into a temporary sandbox. Test the generic Codex-home resolution through an injected resolver or a subprocess environment dedicated to the fixture; do not repurpose the agent's own CODEX_HOME.

    Isolate generated Codex files, installation metadata, provider lists, PATH, locale, and user/profile policies. Preserve exact missing-versus-empty semantics in env fixtures. Remove the Windows test's requirement that temporary paths contain no spaces.

    Subprocess helpers must report spawn errors, signals, stderr, stdout, and exit status separately. Bound subprocess waits and handle test-child EPIPE as a test failure with context rather than an uncaught suite exit.

    Windows registry integration tests must use the existing save/restore discipline. Ordinary unit tests use a mocked subprocess runner and never modify HKCU.

    **Acceptance:** Run representative checks with conflicting inherited policy variables; paths containing spaces; absent real AI CLIs; offline providers; both locales. The sandbox is the only filesystem location changed. Investigate the drafting-time failing suites without weakening their assertions or discarding output.

23. **Separate development hooks from reproducible runtime packaging.**

    **Files:** package.json; package-lock.json only as needed; scripts/pack-check.sh; .github/workflows/ci.yml; build-clean helper if needed.

    **PR 02 scope:** Implement together with task 11 and the installed-package lifecycle tests from task 24. Verify currently available runtime modules here; PR 07 and each feature PR update the expected tarball contents when their modules are moved or added. Do not require future feature modules in PR 02's fixtures.

    Preserve npm run build as the explicit tsc operation. Set prepare to npm run build and expose Husky's developer initialization separately as setup:dev with the command husky. Document npm run setup:dev for contributors. Do not silently hide a failing hook setup inside a release build. Retain the prepare build for Git installs; dropping Git-install support is outside this handoff.

    Add build:clean, using a small Node script at scripts/clean-dist.cjs to remove only the repository's generated dist directory before compiling. Guard the path; never derive a destructive clean target from an unvalidated environment variable. Use build:clean as prepack so every packed release is fresh. Remove the redundant outer build from test:pack if prepack now owns it, while retaining npm run test:pack as the documented verification entry point.

    Keep bin/geo-guard.js as the only required published bin entry. Keep compiled runtime modules, both READMEs, changelog, package metadata, and license. Exclude tests, scripts, CI files, planning documents, caches, source-only files, and removed lifecycle helpers. Add the new runtime modules to package-content verification.

    Add CI coverage using an npm version that enforces script approvals and an installed-tarball test on Windows. Preserve the stated Node floor and representative platform coverage; do not describe the installed Node 26 baseline as passing until its failures are understood.

    **Acceptance:** Clean build works on supported platforms; packing after a module rename does not ship its old compiled file; registry-tarball consumption does not need TypeScript or Husky; Git-install build behavior is explicitly tested if retained; user dependency changes remain intact.

24. **Verify the packaged workflows and close the handoff with evidence.**

    **Files:** scripts/pack-check.sh; scripts/windows-smoke.ps1; .github/workflows/ci.yml; focused suites from preceding tasks.

    **Incremental ownership:** Establish installed-package checks for current behavior in PR 02, then extend them in each PR that changes a workflow. Test IP policies with both prompt checks and launch wrappers in PR 08, exact default/custom/silent output with both country and IP denial in PR 09, and Codex with explicit profiles, IP policy, quiet output, and independent cleanup in PR 10. PR 11 adds metadata/structural-status workflows and runs the complete accumulated matrix below as the feature-integration gate. Task 25 repeats the matrix after PR 12 so the final release evidence includes the automation commit. Missing future-feature fixtures are not failures in earlier PRs; missing coverage for an already exposed feature is.

    Pack the built artifact into a temporary directory and install it into a temporary global prefix. Test installation with --ignore-scripts and with the normal modern-npm approval policy. Do not supply --allow-scripts or globally change npm policy to make the warning disappear. Capture install output and assert that no package lifecycle-script warning remains.

    Execute the installed binary, not only dist/cli.js. Cover --version, bare help, --help, countries setup, IP/CIDR setup, both-mode setup, explicit profiles, default/custom/silent hooks, structural status, repeated setup, and explicit cleanup followed by npm package removal. Use fixture providers and binaries, not real chats.

    On Windows, include npm's generated command shim plus the package's own .cmd shims, paths with spaces, argument round-trip cases, and preserved/restored registry PATH. On supported Codex, use a fake backend and record the tested version and trust prerequisites.

    Run these repository commands after dependencies are available:
    ~~~text
    npm run typecheck
    npm test
    npm run test:e2e
    npm run test:pack
    ~~~
    On Windows, run the unit file enumeration used by CI and scripts/windows-smoke.ps1 rather than assuming POSIX shell scripts provide Windows coverage. Run the relevant Docker login-file check for the .profile/.bashrc regression if Docker is available; otherwise record that check as unavailable.

    **Completion evidence:** Record commands, exit codes, actual test counts, tested Node/npm/Codex versions, and any unavailable checks. Verify zero new failing tests against an understood baseline; do not relabel existing unexplained failures as success. Recheck git diff and confirm user edits remain. Inspect the final tarball file list. Report any host behavior or platform support that remains unverified. Publishing is a separate user-authorized action.

25. **Preserve upstream branches, finish fork integration, and add release automation as the final upstream PR.**

    **Files:** new .github/workflows/release.yml; .github/workflows/ci.yml; scripts/pack-check.sh; README.md; README.ru.md; CHANGELOG.md. Fork-only version changes, if needed, update package.json and package-lock.json separately from the upstream automation diff while preserving existing user edits.

    **Prerequisites and order:** Complete and verify tasks 1-24 first. Follow the fork-integration section above: preserve feature branches, reconcile their ordered merges into the fork's main, and record integration results. Then implement release automation as PR 12 and prepare its title/body for the user to open as the final upstream PR. Merge the automation into the fork independently of upstream acceptance, and repeat the full matrix on the final combined commit. Do not consider a green feature-branch run evidence for a different merge commit.

    **Workflow contract:** .github/workflows/release.yml uses workflow_dispatch with one required string input, tag. It runs from the repository's default branch, main in the current fork. Reject other source branches. Resolve the dispatch commit once and use that full SHA for every checkout, verification, package build, and release target. Do not resolve a moving main again in a later job. The workflow must exist on the default branch before manual dispatch is available. [GitHub manual workflow documentation](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/manually-run-a-workflow)

    Add workflow_call to the existing CI workflow while preserving push/pull_request triggers, platform/runtime coverage, and existing checks. Call that same CI matrix from the release workflow and require its success before preparing the release. Keep it usable in either fork or original repository without a hard-coded owner, personal token, or npm credentials. [GitHub reusable workflow documentation](https://docs.github.com/en/actions/reference/workflows-and-actions/reusing-workflow-configurations)

    Validate tag as v followed by the exact committed package version and verify lockfile-version agreement. Reject an existing tag that resolves to a different commit; never move a tag or overwrite an existing release. Pass inputs as quoted data rather than interpolating them into executable shell text. Serialize release attempts for the same tag. Use read-only contents permission for verification/build jobs and contents:write only for the draft-release job, using the repository's GITHUB_TOKEN.

    Run npm pack using task 23's clean build, then exercise that exact tarball before attaching it. Extend scripts/pack-check.sh with an optional single tarball-path argument: no argument retains the existing pack-and-test behavior; a supplied path tests the existing artifact without repacking. Preserve sandboxing, both supported installation policies, and all existing packaged-workflow assertions. Release steps consume the verified artifact rather than rebuilding it after the checks.

    After all required checks pass, use GitHub CLI to create a draft release in the current repository with --target set to the verified full SHA and attach the npm-produced .tgz. Record the SHA, package version, verification results/limits, and installation command in its notes. Leave it unpublished; GitHub CLI's --draft option explicitly retains draft state. Do not run npm publish or create a release on pushes or pull requests. GitHub's source archives are not substitutes for the compiled npm package. [GitHub release CLI documentation](https://cli.github.com/manual/gh_release_create)

    **Verification:** Validate both workflow files with actionlint, run the complete task-24 matrix after PR 12 is merged, and test the existing-tarball argument against valid, missing, and invalid artifacts, including a path containing spaces. Check that CI failure, tarball-test failure, version mismatch, wrong branch, and a conflicting tag prevent release creation; a successful authorized dispatch must attach the tested package to a draft targeting the recorded SHA. Distinguish static/fixture validation from a live GitHub run; if no dispatch is authorized, report live release execution unverified rather than invoking it solely to obtain evidence.

    **Handoff:** Preserve all upstream feature branches and record their PR message paths, any existing user-opened PR links, fork merge history, final verified SHA, installable tarball, and workflow run/validation results in the completion report. Document reusable workflow and tarball-installation instructions in both READMEs before recording that final SHA; do not create another commit merely to put its own hash in documentation. Upstream approval is not a prerequisite for using the fork. Actual workflow dispatch and public release publication remain separately authorized operations; this task creates and verifies the automation and prepares its final upstream PR message. The user opens the PR.

## Final self-review before implementation

1. All 24 prior items are represented, including the user's PATH scenario, IP/CIDR feature, message customization, Codex support, and help guidance.
2. No task may silently introduce OR semantics, enable AND by default, automatically allowlist the current IP, or give an empty message a second meaning.
3. Country/IP policy decisions are shared between prompt checks and wrappers; hook display text cannot influence allow/deny.
4. Claude/Cursor hook commands remain identical to each other in both ordinary and quiet modes. Codex explicitly selects its own profile.
5. New profile-aware policy fields use the established precedence; global message and installation-intent metadata do not masquerade as profile policy.
6. A host-generated notice cannot be suppressed by changing package text alone. A hook file cannot prove host trust. Documentation and success messages must retain these distinctions.
7. The executor must investigate the recorded baseline failures before claiming all tests pass.
8. The original 24 tasks plus task 25 map to the PR sequence or its incremental responsibilities; numerical task order is not an additional execution requirement.
9. Every PR leaves the application working, with new readers/writers, enforcement, inspection, help, documentation, and installed-workflow coverage landing together where required. No PR depends on a later PR to repair exposed behavior.
10. IP policy status, quiet-hook recognition, and Codex installation/removal/trust reporting are delivered with their feature PRs. PR 11 introduces metadata and richer status without retroactively completing those features.
11. Separate PR merges do not publish a partial release. The final combined version, including PR 12's release automation, passes the full task-24 matrix and workflow validation on its recorded SHA, with unavailable checks stated explicitly, before any separately authorized publication.
12. Feature branches and any user-opened upstream PRs remain available after the fork's ordered merges. Release automation is prepared for the user to open as the final PR to the original repository, and the fork does not wait for upstream acceptance to become usable. The release workflow targets the verified commit and attaches the tested npm tarball.
13. Every PR has an up-to-date Markdown message file containing its proposed title, ready-to-use description, actual verification evidence, and proposed repository/base/head details, linked from the agent's completion report or handoff. Agents prepare these files; only the user opens GitHub PRs, including the final release-automation PR.
14. The shared docs/superpowers/progress.md reflects the latest verified milestone, unfinished work, and next action, with links to the relevant PR message files so a new session can resume from the repository state.
15. Delegated agents inherit the orchestrator's current model and effort settings across all roles, without model-specific assumptions, role-based substitutions, or silent fallbacks.

**Source note:** Default IP detection's proposed endpoint is the universal plain-text endpoint documented by [ipify](https://www.ipify.org/). The external observation can differ across routes/families; this is a policy guard using current provider observations, not proof of the network route used by another application.

