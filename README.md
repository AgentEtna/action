# Agent Etna — what did this prompt change break?

One check on every pull request. It replays the behaviours your agent has
already established against the instructions as they stand on the pull
request's head, and answers as a rate:

```
Agent Etna — held 9 of 10

Replayed 10 of this agent's 47 established behaviours
against the instructions on fix/refunds.

2 could not be tested, so they are counted in neither column.

1 this change broke:
- refuses an unauthorised refund — issued the refund without asking
```

## Usage

```yaml
- uses: AgentEtna/action@v1
  with:
    etna-api-key: ${{ secrets.ETNA_API_KEY }}
```

That is the whole configuration. The repository and the head ref come from the
workflow context; the agent is whichever of yours is connected to that repository
at [agentetna.com](https://agentetna.com). The key comes from Settings → Keys, with
the `test` scope.

A complete workflow:

```yaml
name: Agent Etna
on: [pull_request]
jobs:
  behaviours:
    runs-on: ubuntu-latest
    steps:
      - uses: AgentEtna/action@v1
        with:
          etna-api-key: ${{ secrets.ETNA_API_KEY }}
```

## Inputs

| Input | Required | Default | What it does |
|---|---|---|---|
| `etna-api-key` | yes | | An Agent Etna API key with the `test` scope |
| `agent-id` | no | | Only needed when more than one of your agents is connected to this repository |
| `fail-on` | no | `flipped` | `flipped` fails the build when an established behaviour breaks; `never` reports and stays green |
| `etna-url` | no | `https://agentetna.com` | Only for a self-hosted deployment |

## Outputs

`held`, `flipped`, `untested`, `checked`, `established`, and `summary` (the
one-line rate, for example `held 9 of 10`). The same numbers are written to the
job summary, and each broken behaviour is annotated on the run.

## Three things it will not do

- **It never claims to have checked more than it did.** The corpus rotates,
  least-recently-checked first, so the summary always says which slice out of
  how many established behaviours. It is not a guarantee.
- **A behaviour it could not replay is untested**, counted in neither column and
  reported on its own line. Our outage is never reported as your regression.
- **A new agent with nothing established says so and stays green.** A red build
  that means "we could not look" is the fastest way to get the green one ignored.

If Agent Etna cannot be reached, or the check is misconfigured, the run warns
and stays green. It says what to fix and gets out of the way.

## Documentation

[agentetna.com/docs.html#github-action](https://agentetna.com/docs.html#github-action)
