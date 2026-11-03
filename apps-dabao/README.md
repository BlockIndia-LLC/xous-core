# Apps for Dabao

These are applications targeting a minimal set of Xous services for hardware configurations that are basically "just the chip".

The image flashed onto dabao was built using the following command:

`cargo xtask dabao dabao-console --no-timestamp --kernel-feature debug-proc`

See [the Baochip README](../README-baochip.md) for more details.

## Generic HID and hosted runs

* `--feature usb-bao1x/generic-hid` replaces the USB composite (keyboard, FIDO, serial console) with a
  single vendor-defined HID interface (usage page 0xFF00, 64-byte reports, product "Dabao HID") for apps
  that talk to a host over raw reports (`UsbHid::hid_wait_incoming` / `hid_send`). There is then no USB
  console.
* Prebuilt app binaries can be given to `cargo xtask dabao` as paths; they go into `apps.uf2`.
* `cargo xtask dabao-emu <apps>` runs apps on the host with only the base services (ticktimer, log,
  names).

Example: the Ethereum burner in `ethereum-app/dabao-app-eth-burner`. Hardware checks still to run are in
[HARDWARE-TESTS.md](HARDWARE-TESTS.md).

