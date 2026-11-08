# teppi-check action

Checks the 402 your paid endpoint returns, with no payment: whether it parses, whether the terms
fit the x402 spec and the official SDK, and what is broken about the listing. Results go into the
job summary as a table, with the command that reruns each one.

```yaml
name: endpoints
on:
  push:
  schedule:
    - cron: '0 */6 * * *'

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: teppi-xyz/teppi-check-action@v1
        with:
          urls: |
            https://api.example.com/v1/quote
            https://api.example.com/v1/search
```

| input | default | |
|---|---|---|
| `urls` | | endpoints, one per line |
| `method` | tries GET then POST | |
| `body` | `{}` | JSON sent with a POST |
| `strict` | `true` | fail on a spec problem, not only on a broken endpoint |
| `try-unfunded` | `false` | send one payment from an empty wallet and check it is refused |
| `version` | `0.5.0` | teppi-check version from npm |

The job fails with the CLI's exit code: `1` a check failed or a defect was found, `2` an endpoint
could not be reached.

It runs the same request the Teppi prober sends, so what passes here is what the public record
will see. It never pays, and nothing it sends changes a grade.

MIT.
