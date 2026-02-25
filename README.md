# rust-vhpi

Project structure

* `vhpi-sys/` - raw low-level generated bindings to C API.
* `vhpi/` - higher level Rust bindings.
* `dumper/` - example plugin.
* `tests/test_simple/` - assertion-based plugin for `tb_simple` checkpoints.

Test with

```bash
cargo build && nvc --vhpi-trace --load ./target/debug/libdumper.so -r toplevel
```

## VHPI + nvc test strategy

Use the reusable runner to validate a VHPI plugin (for example `dumper`) against
multiple VHDL testbenches with isolated `nvc` work libraries:

```bash
./scripts/run_vhpi_nvc_checks.sh --plugin dumper
```

What it does:

* Builds the selected plugin crate as a shared library (`cdylib`).
* Creates a separate work directory per testbench under `target/nvc-work/`.
* Runs `nvc` analyze/elaborate/simulate for each `test_examples/*.vhdl` file.
* Loads the plugin via `--load` and checks for VHPI lifecycle markers in logs.

Useful options:

* `--test <tb_name>` run only selected testbench(es).
* `--release` build and run with release shared library.
* `--trace` enable `nvc --vhpi-trace` during simulation.
* `--work-root <path>` override the default work/log directory root.
* `--expect <regex>` add required regex marker(s) per simulation log.

For non-`dumper` plugins, pass one or more `--expect` patterns that match
the plugin's expected VHPI output markers.

`test_simple` can be run via the same strategy script:

```bash
./scripts/run_vhpi_nvc_checks.sh --plugin test_simple --test tb_simple \
	--expect 'test_simple plugin loaded' \
	--expect 'test_simple: all 6 checkpoints passed'
```

## tb_simple assertion plugin

Build and run the assertion plugin against `tb_simple.vhdl`:

```bash
cargo build -p test_simple
nvc -a test_examples/tb_simple.vhdl
nvc -e tb_simple
nvc -r tb_simple --load=target/debug/libtest_simple.so
```

Expected success marker:

```text
test_simple: all 6 checkpoints passed
```
