[![latest packaged version(s)](https://repology.org/badge/latest-versions/xg-multi.svg)](https://repology.org/project/xg-multi/versions)

# Setup composite USB-gadget in a breeze !
**xg_multi** stands for *Extended* `g_multi` (Multifunction Composite USB-gadget).\
`xg_multi` seemlessly interoperates with Linux, macOS and Windows hosts (unlike [`g_multi`](https://www.kernel.org/doc/Documentation/usb/gadget_multi.txt)[^1]).

This Extended Multifunction Composite USB-gadget is built upon `libcomposite` `configfs` system[^2].\
It is **a shell script** to run on device in order to enable features, rather than a driver to load (like for its sibling).

## Features:
`xg_multi` does create the following gadgets on any device with a USB device controller (UDC) originaly set as **OTG-peripheral**:
- serial (ACM)
- ethernet (ECM for Linux/macOS, RNDIS for Windows - **switches gracefully**)
- mass-storage

## Benefits:
- simple: one single command/service to run (no kernel parameters-list & drivers fiddling)
- Interoperates with most host OS computers (Linux/macOS/Windows) without additional host-side drivers or configuration required.
- Supports any linux device with OTG-peripheral capability (including Raspberry Pis[^3]).
- Performs initial OTG ports sanity-checks and returns diagnostics if not properly set.

## Setup procedure:
Make sure `dwc2` (or `dwc3`) driver is **previously loaded** on capable device, **and** configuration is set to **OTG peripheral** mode: depending on devices, this may be driven by hardware (including cable) and/or software.\
(e.g., on supporting Pi devices[^3], just add `dtoverlay=dwc2,dr_mode=peripheral` in `usercfg.txt` to force both by software)

Then connect device to host via USB cable, and run `xg_multi` on device as follows:
```
usage: xg_multi [-D <MAC address>>] [-H <MAC address>] [-V <file path>]
       xg_multi -r

Setup (or remove) Extended Multifunction USB-gadget: serial, ethernet (ECM/RNDIS),
and mass-storage (if valid path is specified).
Ports are just created and are left unconfigured (i.e console, networking,...)

Options: -D|--Device <MAC address>  Specify MAC address for device
         -H|--Host <MAC address>    Specify MAC address for host
         -V|--Volume <file path>    Path to device/LUN file to use as mass-storage
         -r|--remove                Remove gadgets
         -h|--help                  Help information and usage
```
Main execution steps are logged: `cat /var/log/messages | grep xg_multi`.

OpenRC and Systemd services files are provided to run `xg_multi` as a boot service (check [wiki](https://github.com/macmpi/xg_multi/wiki/Install) for details).\
A complete Alpine Linux [package](https://pkgs.alpinelinux.org/packages?name=xg_multi&branch=edge&repo=&arch=&origin=&flagged=&maintainer=) is also available.

[![Packaging status](https://repology.org/badge/vertical-allrepos/xg-multi.svg)](https://repology.org/project/xg-multi/versions)

*Note:*
- application-specific ports setup (i.e. serial options, console bring-up, networking configuration, ...) are not in the scope of this project: user shall take care of this after gadget ports are created.\
(i.e: on Alpine Linux, after running `xg_multi`, networking port setup can be done with `setup-interfaces`)
- for serial connection from Linux host featuring Modem Manager, host user may need to be part of `dialout` group, and create some [filtering rule](https://linux-tips.com/t/prevent-modem-manager-to-capture-usb-serial-devices/284/2) to avoid spurious `AT` commands on serial line.
```
  cat /etc/udev/rules.d/99-ttyacms-gadget.rules
  ATTRS{idVendor}=="0x1d6b" ATTRS{idProduct}=="0x0104", ENV{ID_MM_DEVICE_IGNORE}="1"
```
- `xg_multi` is initially intended to run on `ash` shell within `busybox`, and works within other environments.

##
[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/macmpi)

## Credits
Kudos for info & snippets from @geekman, @Leo-PL and many others to understand/work-around various MS-Windows particularites...

[^1]: Windows does NOT support dual configurations (ECM/RNDIS) with composite gadget.
[^2]: Check required Linux kernel configuration [options](https://github.com/macmpi/xg_multi/wiki/Linux-kernel-configuration)
[^3]: OTG capable Pi devices include Zero serie/A/A+/3A+/4B/400/5/500/Compute-Modules
