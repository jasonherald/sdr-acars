# ACARS test fixtures

## `acars_test.wav`

4-channel WAV at 12 500 Hz, 16-bit signed PCM. ~430 KB. Each channel
carries one ACARS RF channel post-AM-demod-and-decimation to the
12.5 kHz IF rate.

**Source:** the `test.wav` shipped with the [acarsdec] project
(`<https://github.com/TLeconte/acarsdec>`), licensed LGPL-2.0 — the
same license as this crate. Vendored here so the test suite doesn't
depend on a checkout of the C reference repo.

**Used by:**
- `tests/e2e_acarsdec_compat.rs` — runs `sdr-acars-cli` on this file
  and diffs the output (with volatile fields stripped) against a
  committed snapshot of `acarsdec`'s output on the same input. This
  is the decoder's primary correctness oracle: synthesizing
  ACARS-grade MSK in Rust for a self-contained test is non-trivial,
  so a real recording + a reference-tool snapshot is the most
  reliable check.

[acarsdec]: https://github.com/TLeconte/acarsdec
