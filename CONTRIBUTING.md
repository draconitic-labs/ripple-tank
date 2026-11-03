# Contributing to Ripple Tank

Bug reports, numerical test cases and focused fixes are welcome. Open an issue before starting large changes.

Before opening a pull request, run:

```sh
cargo fmt --all -- --check
cargo test --release --locked
cargo clippy --all-targets --locked -- -D warnings
```

Numerical changes need a test in `tests/physics.rs` that fails without the change. By submitting a contribution you agree that it is licensed under the project's [MIT License](LICENSE). Report security problems privately as described in [SECURITY.md](SECURITY.md).
