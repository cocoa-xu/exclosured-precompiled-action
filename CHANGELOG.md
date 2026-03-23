# Changelog

## v1.0.0

Initial release.

- Composite GitHub Action for building Exclosured WASM modules
- Installs Elixir, Rust, wasm32 target, and wasm-bindgen-cli
- Runs `mix compile` to build WASM modules via Exclosured
- Packages modules into `.tar.gz` archives with `.sha256` sidecars
- Outputs archive directory and file list for upload steps
- Configurable Elixir/OTP/Rust versions and cargo arguments
- Cargo bin caching for faster wasm-bindgen-cli installation
