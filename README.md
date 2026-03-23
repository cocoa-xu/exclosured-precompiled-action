# Exclosured Precompiled Action

A GitHub Action that builds [Exclosured](https://github.com/cocoa-xu/exclosured)
WASM modules and packages them for precompiled distribution via
[exclosured_precompiled](https://github.com/cocoa-xu/exclosured_precompiled).

Since all Exclosured modules compile to `wasm32-unknown-unknown`, there is
only **one compilation target**. No build matrix needed.

## Usage

### Basic

```yaml
name: Release
on:
  push:
    tags: ["v*"]

jobs:
  precompile:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - name: Build and package WASM modules
        uses: cocoa-xu/exclosured-precompiled-action@v1
        with:
          modules: my_processor,my_filter
          project-version: ${{ github.ref_name }}

      - name: Upload to GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          files: |
            _build/precompiled/*.tar.gz
            _build/precompiled/*.sha256
```

### With custom options

```yaml
      - name: Build and package WASM modules
        uses: cocoa-xu/exclosured-precompiled-action@v1
        with:
          modules: my_processor,my_filter
          project-version: ${{ github.ref_name }}
          elixir-version: "1.17"
          otp-version: "27"
          rust-toolchain: stable
          wasm-bindgen-version: "0.2.114"
          project-dir: "./"
```

### Full release workflow with Hex publish

```yaml
name: Release
on:
  push:
    tags: ["v*"]

jobs:
  precompile:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - name: Build and package WASM modules
        uses: cocoa-xu/exclosured-precompiled-action@v1
        id: build
        with:
          modules: my_processor,my_filter
          project-version: ${{ github.ref_name }}

      - name: Upload to GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          files: |
            _build/precompiled/*.tar.gz
            _build/precompiled/*.sha256

      - name: Generate checksums
        run: |
          mix exclosured_precompiled.checksum \
            --local \
            --dir _build/precompiled \
            --module MyLibrary.Precompiled

      - name: Publish to Hex
        env:
          HEX_API_KEY: ${{ secrets.HEX_API_KEY }}
        run: mix hex.publish --yes
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `modules` | Yes | | Comma-separated list of WASM module names |
| `project-version` | Yes | | Version string for archive filenames |
| `project-dir` | No | `"./"` | Path to Elixir project root |
| `elixir-version` | No | `"1.17"` | Elixir version to install |
| `otp-version` | No | `"27"` | Erlang/OTP version to install |
| `rust-toolchain` | No | `"stable"` | Rust toolchain (stable, nightly, etc.) |
| `wasm-bindgen-version` | No | `"latest"` | wasm-bindgen-cli version |

## Outputs

| Output | Description |
|---|---|
| `archive-dir` | Directory containing generated archives |
| `archives` | Space-separated list of `.tar.gz` file paths |

## What it does

1. Installs Elixir, Erlang/OTP, and Rust with `wasm32-unknown-unknown` target
2. Installs `wasm-bindgen-cli`
3. Runs `mix deps.get` and `mix compile` (which builds the WASM modules)
4. Runs `mix exclosured_precompiled.precompile` to package each module into a `.tar.gz` with a `.sha256` sidecar
5. Outputs the archive directory and file list for uploading

## Acknowledgements

Inspired by [rustler-precompiled-action](https://github.com/philss/rustler-precompiled-action)
by Philip Sampaio.

## License

MIT
