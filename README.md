# x2000-extras

OpenWrt feed with one package, `x2000-extras`, for the GL.iNet GL-X2000
(Spitz Plus) running plain OpenWrt. It fills in what the OEM firmware did on
its own.

- **Internet LED** follows real connectivity over whatever uplink is active
  (mwan3-aware), instead of the ethernet WAN port only.
- **Signal bars** (the four indicator LEDs) show the modem's signal quality
  from ModemManager.
- **Wi-Fi on/off on the reset button**: hold 2-5 s to turn all Wi-Fi off or
  back on, like the hardware Wi-Fi switch on other GL.iNet routers. Handy when
  the box is used as a plain modem/uplink for another router. Radios you
  turned off on purpose stay off when Wi-Fi comes back.

  A fresh install starts with Wi-Fi on. Open (passwordless) SSIDs are never
  brought up: set a Wi-Fi password, then toggle or run `wifi-switch on`.
  Also available as `wifi-switch on|off|toggle|status`.
- **Two uplinks with failover**: the modem and the ethernet WAN port, managed
  by mwan3, ethernet preferred.
- **Cellular watchdog**: brings the modem connection back after a carrier-side
  hangup, with exponential backoff so the SIM is not put into LTE retry
  backoff.

Installing it pulls in ModemManager, the QMI/serial modem drivers and mwan3.

## Reset button

| Press | Action |
| --- | --- |
| tap (< 1 s) | reboot |
| hold 2-5 s | toggle Wi-Fi on/off |
| hold 10 s+ | factory reset (power LED starts blinking at 10 s) |

Factory reset is moved up from the stock 5 s so a slow toggle cannot wipe the
box. The stock handler is kept as `/etc/rc.button/reset.stock` and put back if
the package is removed.

## What it configures

Runs once, on first boot of an image or right after `apk add`:

- turns Wi-Fi on, unless the box already has a Wi-Fi switch state
- detaches the Internet LED from the `wan` netdev trigger
- finds the existing modem (`proto modemmanager`) and ethernet WAN
  (`proto dhcp` on device `wan`) interfaces; creates `wwan` / `waneth` for
  whichever is missing. A created `wwan` has no APN: set
  `network.wwan.apn` if your carrier needs one
- gives them metrics 10 (ethernet) and 20 (modem) unless already set, and adds
  both to the `wan` firewall zone
- replaces mwan3's example config with an ethernet-then-modem failover (a
  config you have already customised is left alone)
- schedules the watchdog in cron

## Installing

The GL-X2000 is supported on OpenWrt snapshots only (not yet in a stable
release), so these steps assume a snapshot image.

### 1. Get an image with the dependencies built in

mwan3 needs kernel modules, and those only load on the exact kernel they were
built with. Snapshot repos move daily, so `apk add` on an image that is a few
days old usually fails to install them. Baking the dependencies into the image
avoids that:

1. Open the [Firmware Selector for the GL-X2000](https://firmware-selector.openwrt.org/?version=SNAPSHOT&target=qualcommax%2Fipq50xx&id=glinet_gl-x2000).
2. Under *Customize installed packages*, append:
   ```
   modemmanager kmod-usb-net-qmi-wwan kmod-usb-serial-option mwan3
   ```
   and, for the web UI, `luci luci-proto-modemmanager luci-app-mwan3`.
3. Request the build and flash it (sysupgrade from OpenWrt, or the factory
   image from the stock GL.iNet firmware).

### 2. Install the package

On the router:

```sh
apk update
wget -O /tmp/x2000-extras.apk \
  https://github.com/gilbertorconde/x2000-extras/releases/latest/download/x2000-extras.apk
apk add --allow-untrusted /tmp/x2000-extras.apk
```

`--allow-untrusted` is needed because the package is not signed with the
OpenWrt key. It takes effect immediately, no reboot needed.

To update, run the same commands again. To remove: `apk del x2000-extras`
(puts the stock reset button handler back).

### Building it into your own image

Add the feed to `feeds.conf`:

```
src-git x2000 https://github.com/gilbertorconde/x2000-extras.git
```

then:

```sh
./scripts/feeds update x2000
./scripts/feeds install x2000-extras
make menuconfig   # Utilities -> x2000-extras
make
```

## Releasing

CI builds the package on every push. Pushing a tag `vX.Y.Z` that matches
`PKG_VERSION` in `x2000-extras/Makefile` publishes a GitHub release with the
`.apk` attached.
