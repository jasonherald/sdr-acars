# sdr-acars

[![CI](https://github.com/jasonherald/sdr-acars/actions/workflows/ci.yml/badge.svg)](https://github.com/jasonherald/sdr-acars/actions/workflows/ci.yml)
[![crates.io](https://img.shields.io/crates/v/sdr-acars.svg)](https://crates.io/crates/sdr-acars)
[![docs.rs](https://docs.rs/sdr-acars/badge.svg)](https://docs.rs/sdr-acars)
[![License: LGPL-2.0-only](https://img.shields.io/badge/license-LGPL--2.0--only-blue.svg)](#license)

A Rust ACARS (Aircraft Communications Addressing and Reporting System)
decoder — MSK demodulation, frame parsing with parity FEC, multi-block
reassembly, [`acarsdec`]-compatible JSON, and a CLI. A faithful port of
Thierry Leconte's C [`acarsdec`]: pure DSP + parsing, no SDR-driver or
UI dependency, so it drops into any Rust radio pipeline that can hand
it samples.

[`acarsdec`]: https://github.com/TLeconte/acarsdec

## CLI

```console
$ cargo install sdr-acars
$ sdr-acars-cli capture.wav            # N-channel WAV at 12.5 kHz IF, one ACARS channel per WAV channel
$ sdr-acars-cli --iq rec.cs16 \        # raw interleaved-i16 complex IQ ...
      --rate 2500000 --center 130337500 \
      --channels 131.550,131.525,130.450,130.425,130.025,129.125
```

The WAV path expects audio already AM-demodulated and decimated to the
12.5 kHz IF rate (one channel per WAV channel) — the same input
`acarsdec`'s file mode takes. The `--iq` path takes a wideband complex
recording and does the channelization itself. Output matches
`acarsdec`'s `-o 2` plain-text format.

## Library

Two entry points depending on what you can feed it:

### Wideband IQ → multi-channel decode

```rust,no_run
use num_complex::Complex32;
use sdr_acars::ChannelBank;

# fn read_iq_block() -> Vec<Complex32> { Vec::new() }
# fn main() -> Result<(), sdr_acars::AcarsError> {
// US VHF ACARS cluster — fits inside a 2.5 MHz Nyquist window
// centered on the midpoint of the channel extremes (130.3375 MHz).
const US_ACARS: &[f64] = &[
    129_125_000.0, 130_025_000.0, 130_425_000.0,
    130_450_000.0, 131_525_000.0, 131_550_000.0,
];
let mut bank = ChannelBank::new(2_500_000.0, 130_337_500.0, US_ACARS)?;
loop {
    let iq: Vec<Complex32> = read_iq_block();
    if iq.is_empty() { break; }
    bank.process(&iq, |msg| {
        let label = String::from_utf8_lossy(&msg.label);
        println!("{} {label} {}", msg.aircraft, msg.text);
    });
}
# Ok(())
# }
```

### Pre-decimated 12.5 kHz IF audio → single-channel decode

Drive [`MskDemod`] + [`FrameParser`] directly — that's what the CLI's
WAV path does, one pair per WAV channel. See `src/bin/sdr-acars-cli.rs`.

### JSON output

`serialize_acars_json(&msg, station_id)` produces an `acarsdec`-shaped
JSON object (the `output.c::buildjson` schema) plus a
`reassembled_blocks` extension field. Pure data → string — the caller
owns the I/O (write JSONL, feed a UDP socket, …):

```rust
# use sdr_acars::{AcarsMessage, serialize_acars_json};
# fn demo(msg: &AcarsMessage) {
let line = serialize_acars_json(msg, Some("MYSTATION"));
println!("{line}");   // {"timestamp":..., "label":"...", ..., "app":{"name":"sdr-acars","ver":"0.1.0"}}
# }
```

## Cargo features

- **`cli`** *(default)* — builds the `sdr-acars-cli` binary; pulls in
  `clap`, `tracing-subscriber`, and `hound`. Library-only consumers
  build with `default-features = false` and skip all three.

## Correctness

`tests/e2e_acarsdec_compat.rs` runs `sdr-acars-cli` on the `test.wav`
vendored from `acarsdec` and diffs the output (volatile fields
stripped) against a committed snapshot of the C tool's output on the
same input. This is the decoder's primary correctness oracle — see
`tests/fixtures/REGENERATE.md` for how the snapshot is refreshed.

## Minimum supported Rust version

`1.95`. Bumping the MSRV is a minor-version change.

## License

**LGPL-2.0-only** — the same license as upstream [`acarsdec`], whose C
source (the MSK demod, frame decoder + FEC, syndrome table,
channelizer, label parsers, JSON schema, and text printer) is
transcribed directly into this crate. See [LICENSE](LICENSE) for the
full text and [NOTICE](NOTICE) for attribution.

An LGPL-2.0 library can be linked from MIT / BSD / Apache / proprietary
programs — the LGPL'd parts (this crate) just have to stay LGPL and
remain replaceable, per the LGPL's linking terms.
