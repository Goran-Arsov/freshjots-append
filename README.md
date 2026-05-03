# freshjots-append

GitHub Action that appends a line to a [Fresh Jots](https://freshjots.com)
notebook from any workflow. One step, no install.

```yaml
- uses: Goran-Arsov/freshjots-append@v1
  with:
    note: deploys-prod
    text: "deployed ${{ github.sha }} (${{ job.status }})"
  env:
    FRESHJOTS_TOKEN: ${{ secrets.FRESHJOTS_TOKEN }}
```

That's it. The note is created on first append; subsequent runs append a
line each. Read the history from your phone, anytime.

## Why use it

Every workflow run gets a one-line entry in a notebook of your choice,
addressed by filename. Pair the action with the dead-man's-switch
deadline and a scheduled workflow becomes its own watchdog: if the run
stops happening, you get an email.

```yaml
on:
  schedule:
    - cron: "0 3 * * *"   # nightly backup at 03:00 UTC

jobs:
  backup:
    runs-on: ubuntu-latest
    steps:
      - run: ./backup.sh
      - uses: Goran-Arsov/freshjots-append@v1
        if: always()        # log success and failure
        with:
          note: nightly-backup
          text: "backup ${{ job.status }} (${{ github.sha }})"
          append-deadline-hours: 26    # alert if no run within 26h
        env:
          FRESHJOTS_TOKEN: ${{ secrets.FRESHJOTS_TOKEN }}
```

The first run creates the note, sets the deadline, and writes the first
line. If the workflow stops firing, Fresh Jots emails you within ten
minutes of the deadline lapsing — same dead-man's-switch primitive
[Healthchecks.io](https://healthchecks.io) built a business around.

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `note` | yes | — | Filename of the notebook (e.g. `deploys-prod`). Created on first append. |
| `text` | yes | — | Text to append. Newlines are stored verbatim. |
| `append-deadline-hours` | no | unset | Dead-man deadline in hours (1–720). Set on every run; idempotent. |
| `api-url` | no | `https://freshjots.com/api/v1` | Override only for self-hosted instances. |

The action authenticates via the `FRESHJOTS_TOKEN` env var. Mint a
token at <https://freshjots.com/settings/api_tokens> and store it as a
[repository or organization
secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets).
Per-token scopes are supported — for CI use, **write-only** + scoped to
a single note is the recommended posture.

## Outputs

| Output | Description |
| --- | --- |
| `note-id` | Numeric id of the note that received the append. |

## Examples

### Slack-style channels for bots

```yaml
- uses: Goran-Arsov/freshjots-append@v1
  with:
    note: payments-prod
    text: "stripe webhook ${{ github.event.payload.type }} processed"
  env:
    FRESHJOTS_TOKEN: ${{ secrets.FRESHJOTS_TOKEN }}
```

### Failed-deploy log

```yaml
- uses: Goran-Arsov/freshjots-append@v1
  if: failure()
  with:
    note: failed-deploys
    text: "${{ github.workflow }} ${{ github.ref_name }} failed @ ${{ github.sha }}"
  env:
    FRESHJOTS_TOKEN: ${{ secrets.FRESHJOTS_TOKEN }}
```

### Per-environment notebook

```yaml
- uses: Goran-Arsov/freshjots-append@v1
  with:
    note: deploys-${{ inputs.environment }}
    text: "deployed ${{ github.sha }} to ${{ inputs.environment }}"
  env:
    FRESHJOTS_TOKEN: ${{ secrets.FRESHJOTS_TOKEN }}
```

## Versioning

Pin to the major version (`@v1`) for non-breaking updates, or to a
specific release (`@v1.0.0`) or commit SHA for full reproducibility.
The `v1` ref will always point to the latest 1.x release.

## Pricing

Requires a Fresh Jots [Dev tier](https://freshjots.com/pricing) account
($99/yr) or higher. The Action itself is free and MIT-licensed.

## License

MIT — see [LICENSE](LICENSE).
