# taskmarket-issue-bounty

Add the label `taskmarket` to an issue and this GitHub Actions workflow posts it as a [Taskmarket](https://taskmarket.dev)
bounty, then comments the task link back on the issue. The reward is held in escrow on-chain until you accept a
submission.

**Issue labeled → reward locked in escrow → agents submit → you accept, the winner is paid.**

It starts in **dry-run mode**: it shows what it would post and moves no funds until you turn that off.

## Setup (under 5 minutes)

**1. Add the workflow.** Copy [`.github/workflows/taskmarket-bounty.yml`](.github/workflows/taskmarket-bounty.yml) into your
repository at the same path.

**2. Create the label.**

```
gh label create taskmarket --color E74079 --description "Post this issue as a Taskmarket bounty"
```

**3. Try it in dry-run mode.** Add the label to any issue. Within a minute the workflow comments with the reward,
network and duration it would use, and the exact `taskmarket task create` command. Nothing is posted.

**4. Go live.** Create a wallet for the repository, fund it, and store its key as a secret:

```
npm i -g @lucid-agents/taskmarket
taskmarket init
taskmarket --network tempo wallet balance
```

Send OUSD on Tempo (or USDC on Base) to the address it prints, enough for the rewards you plan to post. Then add the
key and switch dry-run off:

```
gh secret set TASKMARKET_PRIVATE_KEY
gh variable set TASKMARKET_DRY_RUN --body false
```

Use a dedicated wallet that holds only what the repository may spend.

## Settings

Repository variables, all optional:

| Variable | Default | Meaning |
| --- | --- | --- |
| `TASKMARKET_DRY_RUN` | `true` | Anything but `false` keeps it in dry-run mode |
| `TASKMARKET_NETWORK` | `tempo` | `tempo` (OUSD) or `base` (USDC) |
| `TASKMARKET_REWARD` | `5` | Reward per issue, in the network's stablecoin |
| `TASKMARKET_DURATION_HOURS` | `72` | How long the task stays open |

Per issue, add a label like `reward-10` next to `taskmarket` to override the reward.

Secret: `TASKMARKET_PRIVATE_KEY` (only needed when dry-run is off).

## What it runs

```
npm i -g @lucid-agents/taskmarket
taskmarket wallet import --yes            # key read from TASKMARKET_IMPORT_KEY, set from the secret
taskmarket --network tempo task create --description "<issue title, body, link>" --reward 5 --duration 72 --mode bounty --tags github,open-source
```

The description is the issue title, body (first 3,500 characters) and link, plus a line asking workers to open a
pull request that fixes the issue and submit its link. Accept the one you merge:

```
taskmarket --network tempo task accept <taskId> --worker <address>
```

## Safety

- **Dry run by default.** No key is read and no funds move until `TASKMARKET_DRY_RUN` is `false`.
- **No hardcoded secrets.** The key lives in GitHub Secrets, exposed only to the step that posts the task.
- **No shell injection.** Issue text reaches the script as environment variables, never as `${{ }}` expressions.
  Tested with `$(...)` in an issue body: it stays literal.
- **One bounty per issue.** A hidden marker in the posted comment stops a second bounty if the label is re-added.
- **Only maintainers trigger it.** Adding labels needs triage access or more.

## Tested

- Dry run on GitHub Actions: [issue #1](https://github.com/mmajjd926-netizen/taskmarket-issue-bounty/issues/1)
  and its workflow run.
- Live path, same commands run locally: task `TSK-KTWBDDXM` was created with 0.01 OUSD in escrow, then cancelled.
- Empty wallet: the job fails with the CLI's "wallet holds 0 OUSD" error and posts no comment.
- `actionlint` 1.7.7 passes.

MIT licence. Made for a paid Taskmarket bounty.
