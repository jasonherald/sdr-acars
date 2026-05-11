## Summary

<!-- 1-3 bullet points describing the change. Lead with intent, not mechanics. -->

## Test plan

- [ ] `cargo fmt --all -- --check`
- [ ] `cargo clippy --all-targets -- -D warnings` (default features)
- [ ] `cargo clippy --all-targets --no-default-features -- -D warnings`
- [ ] `cargo test` (default features — runs the e2e acarsdec-compat test)
- [ ] `cargo test --no-default-features`
- [ ] `cargo doc --no-deps --all-features` (no rustdoc warnings — `RUSTDOCFLAGS=-D warnings`)

## Related issues

<!-- Closes #N / part of #N / etc. -->
