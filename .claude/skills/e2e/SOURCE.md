Source: https://github.com/tester-army/e2e (skills/e2e, commit 34f85c3a1675, 2026-11-06), Apache-2.0 (LICENSE and NOTICE copied beside this file).
Reviewed 2026-11-06 before adding: Markdown guidance only, no scripts. It tells the agent to run `npx e2e ...` inside the project under test (downloads the `e2e` npm package) and has an `npx e2e feedback` command that sends reports to the e2e team; the skill itself requires `--dry-run` and the user's agreement first.
The e2e CLI sends anonymous usage telemetry; every project disables it: `E2E_TELEMETRY_DISABLED=1` (or `npx e2e telemetry disable`).
Update: `npx e2e guide <topic>` prints the reference of the installed package version; prefer it when it differs from this copy. Re-copy from the source repo and re-review before changing this copy.
