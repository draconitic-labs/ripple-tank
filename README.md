# Ripple Tank

A gravity–capillary wave workspace from Draconitic Labs. A shared Rust engine powers a native Qt desktop, a worker-based WebAssembly browser app and a standalone command-line program.

Draw obstacles and depth changes, create coherent sources, change gravity/tension/drag and boundaries, measure a surface probe, and save complete reproducible experiments. The engine uses a conservative staggered mesh and velocity Verlet in physical units. [Numerics](docs/NUMERICS.md) explains the equations, tests and interpretation.

[Browser instrument](https://draconitic.com/labs/ripple-tank) · [Versioned downloads](https://github.com/draconitic-labs/ripple-tank/releases)

## Downloads

The [v0.1.1 release](https://github.com/draconitic-labs/ripple-tank/releases/tag/v0.1.1) has these files:

- `ripple-tank-0.1.1-fedora44-x86_64-desktop.tar.gz`: the native Qt desktop app with its bundled runtime, the `ripple` CLI, the installer and the licence texts.
- `ripple-tank-0.1.1-fedora44-x86_64-cli.tar.gz`: the `ripple` command-line program.
- `ripple-tank-0.1.1-web.zip`: the browser app in `web/`.
- `ripple-tank-0.1.1-source.zip`: the tracked source at the release commit.
- `SHA256SUMS`: checksums of the four archives. Check the ones you downloaded with `sha256sum -c --ignore-missing SHA256SUMS`.

## Fedora desktop

Extract the Fedora 44 x86_64 desktop archive and run the app from the extracted folder:

```sh
tar -xzf ripple-tank-0.1.1-fedora44-x86_64-desktop.tar.gz
cd ripple-tank-0.1.1-fedora44-x86_64-desktop
./ripple-tank/ripple-tank
```

The archive contains its Python/Qt runtime and Rust library; Python installation is not needed. It uses native Qt widgets, not an embedded web page. Tested platform and measurements are in [VERIFICATION.md](VERIFICATION.md). Other operating systems are not claimed as tested.

The optional `python3 install.py`, run from the extracted folder, installs into `~/.local/opt/flintglade-ripple` and adds an application-menu entry. `python3 install.py --uninstall` removes that installation and menu entry; user-added experiments and modified files are retained; the installer reports whether their directory remains. `--prefix` and `--applications-dir` support an isolated installation. Spaces are supported; paths containing desktop Exec-sensitive characters (percent, double quotes, backslashes, dollar signs or backticks) are rejected before installation.

The browser version runs locally after serving `web/` over HTTP. From the extracted `ripple-tank-0.1.1-web.zip`, for example: `python3 -m http.server --bind 127.0.0.1 --directory web 8080`, then open `http://127.0.0.1:8080/`. The numerical engine runs locally in your browser. Open/save and CSV operations do not upload experiments.

## Standalone CLI

`ripple-tank-0.1.1-fedora44-x86_64-cli.tar.gz` extracts to `ripple-tank-0.1.1-fedora44-x86_64-cli/`, which contains `ripple` and documentation. The desktop archive carries the same `ripple`. Examples, run from that folder:

```sh
./ripple new double-slit initial.ripple.json
./ripple run initial.ripple.json 2000 evolved.ripple.json field.csv
./ripple inspect evolved.ripple.json
./ripple bench 200 120 2000
```

Outputs refuse to overwrite existing files. `new` optionally accepts a config JSON file as its last argument. `ripple serve` reads one JSON command per line and emits one response per line for agent and batch integration; see [the interface contract](docs/IMPLEMENTATION-CONTRACT.md). Full states contain units/model parameters, velocities, geometry, sources, intensity, time and step count. Source files and all imported arrays are bounded and validated before replacing the active tank.

## Build from source

Rust1.96.1, Node22 and Python3/PySide6 are the development baseline; Cargo.lock pins Rust dependencies. Native builds require no Qt development toolchain because the frontend loads the Rust shared library through ctypes.

```sh
cargo build --release --locked
cargo test --release --locked
cargo clippy --all-targets --locked -- -D warnings
python3 desktop/ripple.py
rustup target add wasm32-unknown-unknown
cargo build --release --locked --lib --target wasm32-unknown-unknown
cp target/wasm32-unknown-unknown/release/flintglade_ripple.wasm web/ripple.wasm
node scripts/verify-wasm.mjs
python3 scripts/verify-desktop.py
python3 scripts/verify-import-controls.py
python3 -m http.server --bind 127.0.0.1 --directory web 8080
```

For the Python frontend, install a distribution-provided PySide6 or PySide6==6.11.1 in your own environment. To bundle the desktop, use PyInstaller6.22.3 with `--onedir --add-binary target/release/libflintglade_ripple.so:. desktop/ripple.py`; the release script supplies the exact build procedure, license files and inventory. The dynamic Qt libraries and Rust library can be replaced/rebuilt independently. No background services are needed.

Project code is MIT licensed. Bundled runtimes retain their respective licenses; see [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md).

To assemble the Fedora desktop package using the recorded build environment:

```sh
python3 -m venv --system-site-packages .packaging
.packaging/bin/python -m pip install -r requirements-build.txt
python3 scripts/build.py --desktop
.packaging/bin/python scripts/package.py
```

Packaging requires a clean reviewed Git checkout and distribution-provided PySide6/Qt. It records collected RPM versions, source RPM locations, license texts, Rust dependency checksums and binary hashes. Build provenance is recorded; byte-for-byte reproducibility across different Fedora toolchains is not claimed.

`web/workspace.html` is the single workspace fragment. The build script refreshes the standalone wrapper; `--site-path /path/to/isolated/site-checkout` also synchronizes only the Ripple Tank wrapper/assets in that checkout. Deployment and response CSP configuration remain separate reviewed operations.
