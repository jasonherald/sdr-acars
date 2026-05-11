# Regenerating the acarsdec snapshot

The e2e test `sdr_acars_cli_matches_acarsdec_on_test_wav`
(`tests/e2e_acarsdec_compat.rs`) diffs `sdr-acars-cli`'s output
against a committed snapshot of the C [`acarsdec`][acarsdec] tool's
output on `tests/data/acars_test.wav`. The snapshot is committed
(rather than running `acarsdec` at test time) so CI is deterministic
and the C tool isn't part of the test toolchain.

Refresh the snapshot when:

- `acarsdec` upstream changes its plain-text output format, or
- this crate's CLI printer changes a field that should still match.

## Procedure

```bash
# 1. Build acarsdec from a local checkout.
git clone https://github.com/TLeconte/acarsdec
cmake -S acarsdec -B acarsdec/build && cmake --build acarsdec/build

# 2. Generate raw output. `-o 2` selects the one-line + body
#    plain-text printer that sdr-acars-cli matches; redirect stderr
#    to drop the "exiting ..." banner.
./acarsdec/build/acarsdec \
    -o 2 \
    -f tests/data/acars_test.wav \
    > /tmp/acarsdec_raw.txt 2>/dev/null

# 3. Strip volatile fields and write the snapshot. The regex must
#    stay byte-equal to `strip_volatile_line` in
#    `tests/e2e_acarsdec_compat.rs` — keep them in sync.
sed -E 's/^\[#[0-9]+ \(L:[^)]+\)[ 0-9.]*--/[#X (L:N E:N) --/' \
    /tmp/acarsdec_raw.txt > \
    tests/fixtures/acarsdec_test_wav_expected.txt

# 4. Sanity-check: 7 ACARS messages on the fixture WAV.
grep -c '^\[#X' tests/fixtures/acarsdec_test_wav_expected.txt
# Expected: 7

# 5. Verify the test still passes.
cargo test --test e2e_acarsdec_compat
```

## Volatile fields

The strip regex covers everything that depends on wall-clock or
hardware state:

- `#<seq>` — per-message sequence counter (1-indexed)
- `L:<level>` — matched-filter signal level in dB
- `E:<errors>` — bytes corrected by parity FEC
- `<timestamp>` — wall-clock at decode time (`acarsdec` emits two
  trailing spaces with `printdate` off; `sdr-acars-cli` emits a Unix
  epoch with millis — the regex handles both)

Everything else (Mode, Label, Aircraft, Flight ID, Block ID, Ack,
message body, ETX/ETB) must match the C reference exactly.

[acarsdec]: https://github.com/TLeconte/acarsdec
