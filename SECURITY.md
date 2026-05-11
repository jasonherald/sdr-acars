# Security Policy

## Supported Versions

Only the latest release on the `main` branch is supported with security
updates. We do not backport fixes to older versions.

| Branch | Supported |
|--------|-----------|
| `main` | Yes       |
| Other  | No        |

## Reporting a Vulnerability

**Please do not open a public issue for security vulnerabilities.**

Use GitHub's private vulnerability reporting to submit a report:

1. Go to the [Security tab](https://github.com/jasonherald/sdr-acars/security)
2. Click **"Report a vulnerability"**
3. Provide a description, steps to reproduce, and any relevant details

### Alternative reporting

If you cannot use GitHub's private reporting, email **security@aaru.network**
with a description, steps to reproduce, and any relevant details.

### What to expect

- **Acknowledgment** within 48 hours
- **Assessment** of severity and impact within 1 week
- **Fix or mitigation** as soon as practical, depending on severity
- **Disclosure** 90 days after the fix is released, or immediately if the
  vulnerability is already public
- Credit in the fix commit (unless you prefer to remain anonymous)

## Security Scanning

This crate uses automated security scanning in CI:

| Tool | Integration | Coverage |
|------|-------------|----------|
| [cargo-audit](https://rustsec.org/) | GitHub Actions (PR + weekly) | Known CVEs in Rust dependencies (RustSec advisory database) |
| [cargo-deny](https://embarkstudios.github.io/cargo-deny/) | GitHub Actions (PR + weekly) | License compliance, duplicate-crate detection (warn-only), source restrictions |
| [CodeQL](https://codeql.github.com/) | GitHub Actions (PR + weekly) | Static analysis of GitHub Actions workflows |
| [Dependabot](https://docs.github.com/code-security/dependabot) | GitHub-native (weekly) | Automated dependency-update PRs for Cargo + GitHub Actions |

## Scope

This crate is an ACARS decoder: a Rust port of [`acarsdec`](https://github.com/TLeconte/acarsdec).
It:

- Demodulates 2400-baud MSK from real `f32` audio at a 12.5 kHz IF rate
- Channelizes a wideband complex-IQ stream (`ChannelBank`) into per-channel
  ACARS audio
- Parses ACARS frames (parity + CRC, optional parity-FEC recovery), including
  multi-block (ETB-chain) reassembly
- Parses OOOI metadata from message text bodies
- Serializes decoded messages to JSON
- (`sdr-acars-cli`) reads WAV files and raw `cs16` IQ recordings

Vulnerabilities in frame/label/byte parsing, the WAV / IQ file readers in the
CLI, the JSON serializer, or the channelizer/demod are in scope. The decoder
treats all bytes off the air (or out of a file) as untrusted input.

A security advisory in one of this crate's dependencies (`clap`, `hound`,
`tracing`, `serde_json`, `num-complex`, `arrayvec`, …) that affects this crate
**is in scope**: report it here and we'll bump the dependency and cut a patched
release. Notifying the upstream project as well is appreciated. (cargo-audit +
Dependabot also surface most of these automatically.)

### Out of scope

- Vulnerabilities in the upstream C `acarsdec` — report there
- A dependency advisory that does **not** touch this crate's code paths —
  report it to that project
- Malformed-but-harmless input that simply fails to decode (a corrupt WAV, a
  frame that doesn't pass CRC) — that's expected behavior, not a vulnerability
- Resource exhaustion from a deliberately enormous input file (the CLI streams,
  but you can still point it at a terabyte) — a property of file input, not a
  crate bug
