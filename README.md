# Downstream CI for Rust's `libc` on RTEMS

Tests rust-lang/libc `main` on `armv7-rtems-eabihf` once per day. The test
binaries run on the RTEMS 6 `xilinx_zynq_a9_qemu` BSP under QEMU.

The tests fail until libc matches RTEMS 6: ctest prints each mismatch as a
`bad ...` line. The `FIXME(rtems)` skips in `patches/` cover what would
otherwise break the C build or the link.

The RTEMS toolchain, BSP and libbsd are built from the RTEMS 6 branch by
`toolchain.yaml` once per week and published to ghcr.io, since building them
takes too long for every run. `patches/` adds the target to libc's CI and
should go upstream once it is stable.

See <https://github.com/rust-lang/libc/issues/5209> for more details.

## Licensing

This is derived from libc's CI so available under the same licenses (MIT
or Apache-2.0).
