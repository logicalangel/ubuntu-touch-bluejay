# Ubuntu Touch for the Google Pixel 6a (bluejay)

An unofficial [Ubuntu Touch](https://ubports.com) port for the Google Pixel 6a
(codename bluejay, Google Tensor gs101). It runs Ubuntu Touch 24.04-2.x with
Halium 16 on top of the phone's stock Android 16 firmware.

> **Status: alpha.** Most of the hardware works (see below). Installing erases the phone. 
> The bootloader stays unlocked and your data isn't encrypted. Don't install this on a phone
> you depend on.

## Status

Tested on the phone with the development build of 0.1.0 (2026-10-16):

| Area | State |
|---|---|
| Kernel and first-stage drivers (UFS, USB, display, power, modem) | Working: all 203 vendor_boot modules load |
| Display, GPU (Mali-G78), Lomiri | Working |
| Touch | Working |
| Wi-Fi | Working (connects, internet) |
| Bluetooth | Working; audio devices work too |
| Sound output (speaker), volume keys | Working: 16-bit audio policy and a rebuilt PulseAudio droid module (`packages/`) |
| Microphone | Working (records with video) |
| Cameras (front and back), video recording | Working; no ultrawide (tested with [this app](https://open-store.io/app/camera-test.jakub/)) |
| GPS | Working (Broadcom BCM4776 through the stock GNSS HAL; the first fix needs sky view) |
| Sensors (rotation) | Working |
| Vibration | Working |
| Battery and charging | Working |
| adb over USB | Working |
| Modem | Both SIM slots come up in oFono; incoming/outgoing calls and mobile data working. eSIM/EUICC seems to be broken as it doesn't show up on [uSim](https://github.com/muhammad23012009/usim) |
| Wi-Fi hotspot | Not tested |
| Standby (screen off) | Works: about 1.3% battery per hour overnight (Wi-Fi on, no SIM). No deep sleep yet |
| File transfer over USB (MTP) | Not supported: the kernel has no MTP function |
| Fingerprint reader | Not supported by this port |

## Quick start

You need a Pixel 6a on stock Android **BP4A.251205.006**, slot a, with an
unlocked bootloader, and a macOS or Linux computer with Python 3.8 or newer,
`fastboot` and `zstd`. The [install guide](docs/install.md) explains every
step, including how to put that Android build on the phone. From a newer
build that's a downgrade, which bootloader anti-rollback can block: read the
guide first.

1. Download the six files of a release (`boot.img`, `boot-installer.img`,
   `system.img.zst`, `install.py`, `build-info.txt`, `SHA256SUMS`; not the
   "Source code" archives) from the
   [Releases page](https://github.com/logicalangel/ubuntu-touch-bluejay/releases)
   into one empty folder.
2. Turn the phone off, hold Power and Volume Down until the bootloader
   (fastboot mode) appears, and connect it over USB.
3. In that folder, run:

   ```sh
   python3 install.py
   ```

To update later, run the new release's installer with `--keep-data`.

## How it works

This is a Halium GSI port, built the same way as other UBports community ports:

| Part | Source |
|---|---|
| Firmware, vendor and vendor_dlkm | Stock Android 16 from Google, build **BP4A.251205.006**, left untouched on the phone |
| Kernel | UBports' generic GKI kernel, branch [`android14-6.1-halium`](https://gitlab.com/ubports/porting/community-ports/android12/generic/kernel-android-common/-/tree/android14-6.1-halium), pinned in `build.sh`, plus `kernel/bluejay.config` |
| Android system image (in an LXC container) | Generic Halium 16 GSI built by UBports CI |
| Rootfs | Ubuntu Touch 24.04-2.x, with libhybris from the 26.04-1.x archive |
| Shared GKI ramdisk and rootfs overlays | UBports' [halium-gki](https://gitlab.com/ubports/porting/community-ports/android12/generic/halium-gki) port |
| Patched `vndservicemanager` | UBports' [Pixel 9 port](https://gitlab.com/ubports/porting/community-ports/android16/google-pixel-9/google-zumapro) |
| Rebuilt PulseAudio droid module | `packages/` in this repository |
| Device overlay (`overlay/`) | This repository |

The Halium kernel keeps the stock GKI kernel's ABI, so the stock vendor modules
load unchanged. The Halium 16 system image matches the vendor partitions of
BP4A.251205.006, so **keep the phone on that build**.

The installer writes only the boot image (`boot_a`), the userdata partition
and the one-time boot command in `misc`. Ubuntu Touch lives in
`/userdata/ubuntu.img`, which the Halium initramfs loop-mounts. The bootloader,
modem firmware, vbmeta and the stock `super` partition stay as they are, and
the installer doesn't need fastbootd. The [porting notes](docs/porting-notes.md)
explain each decision.

## Documentation

- [Installing](docs/install.md): requirements, installing, upgrading, developer
  access, going back to Android, troubleshooting
- [Building](docs/building.md): CI, local builds, image variants, releases
- [Debugging](docs/debugging.md): the installer shell, logs, pstore, adb and
  SSH
- [Porting notes](docs/porting-notes.md): how the port works and why
- [Changelog](CHANGELOG.md)

## Credits

This port is built on the work of others:

- [UBports](https://ubports.com): Ubuntu Touch, the build tools, the generic
  GKI kernel, halium-gki and the Halium GSI builds
- [Halium](https://github.com/Halium): the GSI, the initramfs and the hybris
  patches
- UBports' Pixel 9 (zumapro) port, the first Halium 16 Tensor port: its
  patched vndservicemanager and many of the Android 16 and Tensor fixes this
  port follows
- [libhybris](https://github.com/libhybris/libhybris) and
  [mer-hybris](https://github.com/mer-hybris) (pulseaudio-modules-droid)
- UBports' Pixel 3a (sargo) port, the pattern for the udev rules

## License

Files in this repository are licensed under the GNU General Public License
v3.0 or later. See [LICENSE](LICENSE). Components fetched during the build keep
their own licenses.
