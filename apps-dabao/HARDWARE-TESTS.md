# dabao hardware test plan: generic HID and the Ethereum burner

Everything below has been verified in the emulator only (`cargo xtask dabao-emu`, where the app uses a TCP
socket instead of USB and a host file instead of the keystore). This file lists what still has to be
checked on a real board.

**Keeping it current:**
- When a feature is added, add its hardware checks here.
- When a check passes on hardware, set its status to ✅ with the date and board revision.
- Record failures in the Notes column.

Status: ⬜ not run · ✅ pass · ❌ fail · ⚠️ pass with remarks

## 1. Image

Built with the burner from `ethereum-app/dabao-app-eth-burner` (see its README):

```sh
cargo xtask dabao <path>/eth-burner --feature usb-bao1x/generic-hid
```

| # | Check | Status | Notes |
|---|---|---|---|
| 1.1 | `loader.uf2`, `xous.uf2`, `apps.uf2` flash with the usual dabao procedure; the prebuilt `eth-burner` binary is in `apps.uf2` (xtask log: "PID 1: eth-burner") | ⬜ | |
| 1.2 | Boot log on the DUART: no panics; keystore, USB and `eth-burner` start | ⬜ | |

## 2. USB (`usb-bao1x/generic-hid`)

| # | Check | How | Status | Notes |
|---|---|---|---|---|
| 2.1 | Default composite | Without `generic-hid`: the board still enumerates as "Dabao" with keyboard, FIDO and CDC serial; console works | ⬜ | |
| 2.2 | Generic HID enumeration | With `generic-hid`: one interface, product "Dabao HID", VID 0x1d50 / PID 0x6197, release 2.00. `lsusb -v` shows a single HID interface with usage page 0xFF00 and 64-byte IN/OUT interrupt endpoints | ⬜ | |
| 2.3 | Host access | `hidapi` opens the device (Linux may need a udev rule for hidraw) | ⬜ | |
| 2.4 | Report path | 64-byte OUT reports reach the app and IN reports reach the host, both directions, many times in a row | ⬜ | |
| 2.5 | Re-plug | Unplug and replug while idle: the device re-enumerates and the app keeps working | ⬜ | |

## 3. Ethereum burner (`eth-burner`)

No screen: watch the serial console. `eth_client.py` is in `ethereum-app/opencard-app-eth-signer/tools`.

| # | Check | How | Status | Notes |
|---|---|---|---|---|
| 3.1 | First boot | Console: "new burner seed created and stored", then "burner address m/44'/60'/0'/0/0: 0x…" | ⬜ | |
| 3.2 | Persistence | Power-cycle: "burner seed loaded", same address | ⬜ | |
| 3.3 | Different chips / fresh slots | Two boards (or after erasing app slots 132..133) give different addresses | ⬜ | |
| 3.4 | USB | `eth_client.py --hid get-version` → 0.1.0; `get-address` → the boot address, at once | ⬜ | |
| 3.5 | Signing | `sign-tx`, `sign-typed-data mail`, `sign-message 'Hello'`, each with `--expect <boot address>`: answered at once, signer recovered | ⬜ | |
| 3.6 | Wrong key | `sign-tx --address 0xA27DF20E6579aC472481F0Ea918165d24bFb713b` → error 2 "address mismatch" | ⬜ | |
| 3.7 | Keystore errors | Console has no "keystore read/write" error at boot (the app slots are open to apps on dabao) | ⬜ | |
| 3.8 | Speed | Time of the first boot's seed creation + BIP-39 (2048 PBKDF2 rounds) before "burner address"; time of a sign request | ⬜ | |

## Known emulator-only gaps

- The emulator has no USB: the burner uses a TCP socket instead, so section 2 and the HID side of section 3
  are never exercised before hardware.
- The burner's seed is in a host file in the emulator; the keystore slots and the chip TRNG only run on
  hardware (3.1-3.3, 3.7).
