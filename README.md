# Setup composite USB-gadget in a breeze !
**xg_multi** stands for *Extended* `g_multi` (Multifunction Composite USB-gadget) -- or -- *Cross*(-hosts OS) `g_multi`.\
Unlike `g_multi`, `xg_multi` seemlessly interoperates with Linux, macOS and Windows hosts.\
This Extended Multifunction Composite USB-gadget is built upon `libcomposite` `configfs` system.\
It is **not an actual driver** to load like its sibling, but **a shell program** to run on device in order to enable features.

## Features:
`xg_multi` does create the following gadgets on any device originaly in **OTG-peripheral** mode:
- serial (ACM): `ttyGS0` on device side
- ethernet (ECM for Linux/macOS, RNDIS for Windows - **switches gracefully**): `usb0` on device side
- mass-storage

## Benefits:
- Works across most host OS computers (Linux/macOS/Windows) without additional host-side drivers or configuration required.
- Should work on any linux device supporting OTG-peripheral mode (including Raspberry Pis).
- Does necessary sanity-checks on ports beforehand and returns clean if not relevant.

*Note:* application-specific ports configurations (i.e. serial options, console bring-up, networking adresses, ...) are not in the scope of this project: user shall take care of this once ports are created.\
(i.e: on Alpine Linux, after running `xg_multi`, port setup can be done during `setup-alpine`)

## Setup procedure:
Make sure `dwc2` (or `dwc3`) driver is previously loaded on device, and configuration is set to **OTG peripheral** mode: this may be driven by hardware (including cable) and/or software.\
(on supporting Pi devices, just add `dtoverlay=dwc2,dr_mode=peripheral` in `config.txt` to force both by software)

Then run `xg_multi` on device as described:
```
usage: xg_multi [-D <MAC address>>] [-H <MAC address>] [-V <file path>]
       xg_multi -r

Setup (or remove) Extended Multifuntion USB-gadget: serial, ethernet (ECM/RNDIS),
and mass-storage (if valid path is specified).
Ports are just created and are left unconfigured (i.e console, networking,...)

Options: -D|--Device <MAC address>  Specify MAC address for device
         -H|--Host <MAC address>    Specify MAC address for host
         -V|--Volume <file path>    Path to device/LUN file to use as mass-storage
         -r|--remove                Remove gadgets
         -h|--help                  help information and usage
```
Main execution steps are logged: `cat /var/log/messages | grep xg_multi`.

OpenRC files are also available to run `xg_multi` as a boot service; an Alpine Linux [package](https://pkgs.alpinelinux.org/packages?name=xg_multi&branch=edge&repo=&arch=&origin=&flagged=&maintainer=) is also available.

*Note:*
- `xg_multi` is intended to run on `ash` shell within `busybox`. It may work within other environments (`bash`,...), but has not been tested (yet).
- on Linux host featuring Modem Manager, user may need to be part of `dialout` group, and create some [filtering rule](https://linux-tips.com/t/prevent-modem-manager-to-capture-usb-serial-devices/284/2) to avoid spurious `AT` commands on serial line.
```
  cat /etc/udev/rules.d/99-ttyacms-gadget.rules
  ATTRS{idVendor}=="0x1d6b" ATTRS{idProduct}=="0x0104", ENV{ID_MM_DEVICE_IGNORE}="1"
```

## Credits
Kudos for various snippets from @geekman, @Leo-PL and many others trying to solve various MS particularites...
