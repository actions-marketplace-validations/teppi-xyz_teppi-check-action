# teppi-check action

[![license](https://img.shields.io/badge/license-MIT-blue.svg)](#license)
[![teppi-check](https://img.shields.io/npm/v/teppi-check.svg?label=teppi-check)](https://www.npmjs.com/package/teppi-check)

**Catch a broken x402 endpoint in CI, before a paying agent does.**

Every run sends your endpoint the unpaid request an agent would send, reads the 402 it gets back,
and checks it against the x402 spec and what the official SDK needs in order to pay. Nothing is
paid. Results land in the job summary as a table, each line with the command that reruns it.

These problems are common. In a sample of 80 live 402s, 27 missed the published schema
(`serviceName` over 32 characters, more than five `tags`), 2 wrote the amount in dollars instead
of atomic units, and 4 Solana offers left out the `feePayer` that `@x402/svm` needs to build a
payment.

## Usage

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

A POST endpoint that needs a body:

```yaml
      - uses: teppi-xyz/teppi-check-action@v1
        with:
          urls: https://api.example.com/v1/extract
          method: POST
          body: '{"url": "https://example.com"}'
```

## What it checks

- **The 402 parses**: the terms are found in the header or the body and decode cleanly.
- **The terms fit the spec**: field formats and lengths, networks, assets, amounts in atomic units.
- **The official client can pay it**: everything `@x402/core`, `@x402/evm` and `@x402/svm` need is present.
- **The resource is yours**: a version 2 payment names the resource it pays for, and it must be the one asked.
- **An unfunded payment is refused** (optional): one payment signed by a fresh empty wallet, reported separately.

## Inputs

| Input | Default | Description |
|---|---|---|
| `urls` | required | Endpoints to check, one per line |
| `method` | tries GET, then POST | HTTP method to send |
| `body` | `{}` | JSON body sent with a POST |
| `strict` | `true` | Fail on a spec problem, not only on a broken endpoint |
| `try-unfunded` | `false` | Send one payment from an empty wallet and check it is refused |
| `version` | `0.5.0` | `teppi-check` version to run from npm |

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Every endpoint passed |
| `1` | A check failed or a defect was found |
| `2` | An endpoint could not be reached |

## Run it locally

The action wraps [`teppi-check`](https://www.npmjs.com/package/teppi-check), so the same check
runs from your terminal:

```sh
npx teppi-check https://api.example.com/v1/quote --spec
```

It sends the same request the Teppi prober sends, so what passes here is what the public record at
[teppi.xyz](https://teppi.xyz) will see. It never pays, and nothing it sends changes a grade.

## License

MIT
