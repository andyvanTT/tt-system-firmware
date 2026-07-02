# TT-System-Firmware

[![Build](https://github.com/tenstorrent/tt-system-firmware/actions/workflows/build-fw.yml/badge.svg?branch=main)](https://github.com/tenstorrent/tt-system-firmware/actions/workflows/build-fw.yml)
[![Run Unit Tests](https://github.com/tenstorrent/tt-system-firmware/actions/workflows/run-unit-tests.yml/badge.svg?branch=main)](https://github.com/tenstorrent/tt-system-firmware/actions/workflows/run-unit-tests.yml)
[![HW Smoke](https://github.com/tenstorrent/tt-system-firmware/actions/workflows/hardware-smoke.yml/badge.svg?branch=main)](https://github.com/tenstorrent/tt-system-firmware/actions/workflows/hardware-smoke.yml)
[![HW Soak](https://github.com/tenstorrent/tt-system-firmware/actions/workflows/hardware-long.yml/badge.svg?branch=main)](https://github.com/tenstorrent/tt-system-firmware/actions/workflows/hardware-long.yml)
[![Metal](https://github.com/tenstorrent/tt-system-firmware/actions/workflows/metal.yml/badge.svg?branch=main)](https://github.com/tenstorrent/tt-system-firmware/actions/workflows/metal.yml)

[P100A CI History](https://docs.tenstorrent.com/tt-system-firmware/p100a_ci_stats.html)
[P150A CI History](https://docs.tenstorrent.com/tt-system-firmware/p150a_ci_stats.html)
[P300A CI History](https://docs.tenstorrent.com/tt-system-firmware/p300a_ci_stats.html)

Welcome to TT-System-Firmware!

This is the Zephyr firmware repository for [Tenstorrent](https://tenstorrent.com) AI ULC.

![Zephyr Shell on Blackhole](./doc/images/shell.gif)

## Getting Started

To get started with the development environment, building and flashing to a board, please refer to our [Getting Started docs](https://docs.tenstorrent.com/tt-system-firmware/develop/getting_started/index.html).

## Building with custom ERISC firmware

The ERISC (Ethernet RISC) firmware ships as a prebuilt blob
(`zephyr/blobs/tt_blackhole_erisc.bin`). You can instead compile it from an
external source tree and have the build swap the result in automatically, while
keeping the exact same `west build` / `west flash` commands.

Set `SB_CONFIG_CUSTOM_ERISC_DIR` to your ERISC source directory. When it is set,
the build compiles the firmware (at CMake configure time) and swaps the compiled
`eth_init.bin` / `eth_params.bin` in for the stock blob. When it is **empty (the
default), the stock prebuilt blob is used unchanged.**

```sh
west build --sysbuild -p -b tt_blackhole@p150a/tt_blackhole/smc app/smc -- \
  -DCONFIG_SHELL=y \
  -DSB_CONFIG_CUSTOM_ERISC_DIR='"/path/to/bh-erisc"'
```

> **Note:** Kconfig string values must be quoted. The `'"..."'` wrapping keeps
> the inner double quotes intact through the shell so Kconfig receives a valid
> string literal. The same applies if you override any other string option below.

Related options (all have working defaults; only `SB_CONFIG_CUSTOM_ERISC_DIR` is
required to opt in):

| Option | Default | Purpose |
| --- | --- | --- |
| `SB_CONFIG_CUSTOM_ERISC_DIR` | `""` | ERISC source tree. Empty ⇒ use stock blob. |
| `SB_CONFIG_CUSTOM_ERISC_TOOLCHAIN_PREFIX` | `/opt/tenstorrent/sfpi/compiler/bin/riscv-tt-elf-` | `RISCV_TOOLS_PREFIX` for the ERISC build. |
| `SB_CONFIG_CUSTOM_ERISC_ARCH` | `-march=rv32imafv -mabi=ilp32` | `ARCH` for the ERISC build (soft-float ABI, see note). |
| `SB_CONFIG_CUSTOM_ERISC_VERSION` | `69.4.20` | Version stamped into the firmware, visible at blob offset `0x188` / in telemetry, so a custom build stands out. |

How the blob is managed:

- Pristine stock blobs are preserved under `zephyr/blobs/erisc/default/`.
- Compiled blobs are staged under `zephyr/blobs/erisc/custom/`.
- The selected pair is copied over the active `zephyr/blobs/tt_blackhole_erisc*.bin`,
  which the flash pipeline reads. A custom build therefore shows those files as
  modified in `git status`; run a default build (or
  `git checkout zephyr/blobs/tt_blackhole_erisc*.bin`) to restore the stock blob.

Prerequisites for the ERISC source tree (only needed when opting in):

- The `tt-hardware-definitions` submodule must be checked out at
  `src/hardware-definitions/` (it provides the Makefiles and linker scripts).
- A RISC-V toolchain matching `SB_CONFIG_CUSTOM_ERISC_TOOLCHAIN_PREFIX` must be
  installed. The default targets the SFPI toolchain, which ships only soft-float
  (`ilp32`) runtime libraries — hence the soft-float `ARCH` default. For a
  hardware-float (`ilp32f`) build, install the `ilp32f`-capable toolchain and set
  `SB_CONFIG_CUSTOM_ERISC_ARCH=-march=rv32imafv -mabi=ilp32f`.

If the source tree is missing or the build fails, the `west build` aborts with a
clear error (the stock blob is only used when `SB_CONFIG_CUSTOM_ERISC_DIR` is
unset).

### Deploying and connecting to the board

After a successful build:

```sh
# Build the TT console utility (for board communication)
make -j -C scripts/tooling OUTDIR=/tmp tt-console

# Flash the firmware to your board
west flash -r tt_flash --force

# Reset the board and refresh PCIe connectivity
tt-smi -r
./scripts/rescan-pcie.sh

# Connect to the board console
/tmp/tt-console
```

## Further Reading

Learn more about `west`
[here](https://docs.zephyrproject.org/latest/develop/west/index.html).

Learn more about `twister`
[here](https://docs.zephyrproject.org/latest/develop/test/twister.html).

For more information on creating Zephyr Testsuites, visit
[this](https://docs.zephyrproject.org/latest/develop/test/ztest.html) page.

## Contributing

We welcome contributions from the community! Please see our [Contributing Guidelines](CONTRIBUTING.md) for information on how to report bugs, submit pull requests, and participate in the development process.

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code.

## License

This repository contains multiple licenses for different components:

- **Overall license for this project, except where specified**: [Apache-2.0](LICENSE) - [Understanding the License](LICENSE_understanding.txt)
- **Documentation and images**: [Creative Commons Attribution 4.0 International (CC-BY)](LICENSE-DOCS)
- **Binary artifacts**: [TT Blob Software License Agreement](zephyr/blobs/license.txt) (Proprietary)

For the full license text and copyright information, please see the [NOTICE](NOTICE) file.
