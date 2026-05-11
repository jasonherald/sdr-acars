# Changelog

All notable changes to `sdr-acars` are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] - 2026-05-11

Initial release. Extracted from the `rtl-sdr` Rust application's
workspace (where it lived as `crates/sdr-acars`) into a standalone
crate. A faithful Rust port of [acarsdec] — pure DSP + parsing, no
SDR-driver or UI dependency.

### Added

- **MSK demodulator** (`msk` module) — 2400-baud minimum-shift keying
  over 1200/2400 Hz tones, 12.5 kHz IF input, Gardner-style bit-timing
  recovery on a 133-tap oversampled matched filter. Port of acarsdec's
  `msk.c`.
- **Frame parser** (`frame` module) — bit-by-bit streaming state
  machine: pre-key / SOH / address / label / text / suffix, parity +
  CRC validation, and optional parity-FEC recovery via the 1936-entry
  syndrome table (`syndrom` module). Port of acarsdec's `acars.c`.
- **Multi-channel bank** (`channel` module) — `ChannelBank` decimates a
  wideband complex-IQ stream down to per-channel 12.5 kHz IF and runs
  one demod+parser pair per ACARS frequency. Port of acarsdec's
  `rtl.c`.
- **Multi-block reassembly** (`reassembly` module) — merges ETB-chained
  block sequences into a single logical message keyed by
  `(aircraft, message_no)`, with a configurable partial-message
  timeout.
- **OOOI label parsers** (`label`, `label_parsers` modules) — extract
  Out/Off/On/In airport codes + event timestamps from the ~40 known
  label codes. Port of acarsdec's `label.c`.
- **JSON serializer** (`json` module) — `acarsdec`-compatible JSON
  schema (matching `output.c::buildjson`) plus a `reassembled_blocks`
  extension field; pure data → string, no I/O.
- **`sdr-acars-cli`** — decode a WAV (one ACARS channel per WAV
  channel, pre-decimated to 12.5 kHz) or a raw `cs16` IQ recording,
  printing in `acarsdec`'s `-o 2` plain-text format. Gated behind the
  default-on `cli` feature; build with `--no-default-features` for the
  library alone (skips `clap` / `tracing-subscriber` / `hound`).
- Correctness oracle: `tests/e2e_acarsdec_compat.rs` runs the CLI on
  the vendored `acarsdec` `test.wav` and diffs against a committed
  snapshot of the C tool's output (volatile fields stripped).

[acarsdec]: https://github.com/TLeconte/acarsdec
[Unreleased]: https://github.com/jasonherald/sdr-acars/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/jasonherald/sdr-acars/releases/tag/v0.1.0
