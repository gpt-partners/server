# Renovate Dependency Dashboard: analysis and guide

Refs #13. Analysis only: no configuration was changed. Decision pending.

## What it is

The Dependency Dashboard is one issue that Renovate creates and keeps up to date in the repository
(here #13, opened by `renovate[bot]`). It is switched on by default through `config:recommended`, which
`renovate.json` already extends. Nobody has to maintain it; Renovate rewrites the issue body on every run.

Source: <https://docs.renovatebot.com/key-concepts/dashboard/>

## What it shows

- **Detected dependencies** per manager. For this repo: GitHub Actions (`actions/checkout`,
  `actions/setup-python`, `python`) and our regex manager (restic, k3s, wg-easy from the `# renovate:`
  comments in `roles/*/defaults/main.yml`). This is a quick check that the regex manager still finds
  every pinned version.
- **Pending updates**: updates Renovate knows about but has not turned into a PR (rate limit, schedule,
  approval required, ignored/closed PRs, errors). Each has a checkbox: tick it and Renovate creates the
  PR on its next run.
- **Open and awaiting-schedule PRs** with links.
- **Deprecated / abandoned packages**, when the data source reports them.
- A checkbox to trigger a new Renovate run on demand (useful after editing `renovate.json`).

Today it says "no open or pending branches": nothing is waiting, so it only lists detected dependencies.

## Benefit for this repo

- One place to confirm that all pins (k3s, restic, wg-easy, Actions) are still tracked. A silently broken
  regex would otherwise just mean "no PRs", which looks the same as "up to date".
- Visibility of updates that were held back or closed without merge.
- Optional approval gate (see below), relevant because merging to `main` means root on the node.
- Cost: one extra open issue; the bot edits it often, so it generates notification noise if watched.

## Options (not applied)

1. **Keep as is (recommended).** Dashboard on, every update gets a PR immediately, the PR merge is the
   release approval (as described in the README). No change needed.
2. **Approval for risky updates only.** Add a `packageRules` entry with `"dependencyDashboardApproval": true`
   for k3s and/or major updates (`"matchUpdateTypes": ["major"]`). Renovate then lists them on the dashboard
   and opens the PR only after a checkbox is ticked. Security fixes are still opened right away. Renovate's
   docs advise against approval for all updates.
3. **Turn it off** (`"dependencyDashboard": false`). Not recommended: the issue is the only overview, and the
   `agent` label / project setup already treats it as an item.

## Observations

- The issue carries the `agent` label and is in the project as "In Progress". The dashboard is
  bot-maintained and never closes, so it cannot be "done" by a merge. Suggest that this PR (`Refs #13`)
  is the result and the issue is closed or the `agent` label dropped by the maintainer. Labels are not
  changed by the agent.
- The Renovate bot would overwrite any manual edit to the issue body.
- The README sentence "The Dependency Dashboard issue lists all pending updates" is accurate; nothing to
  change there.

## Decision needed

Option 1, 2 (which packages) or 3? Only option 2 or 3 needs a follow-up change to `renovate.json`.
