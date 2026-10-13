# x2000-extras

OpenWrt feed with one package, `x2000-extras`, for the GL.iNet GL-X2000
(Spitz Plus) running plain OpenWrt. It fills in what the OEM firmware did on
its own.

- **Internet LED** follows real connectivity over whatever uplink is active
  (mwan3-aware), instead of the ethernet WAN port only.
- **Signal bars** (the four indicator LEDs) show the modem's signal quality
  from ModemManager.
- **Mode toggle on the reset button**: hold 2-5 s to switch between
  - *home*: Wi-Fi off, the box is a cellular backup WAN behind another router
  - *travel*: Wi-Fi back on, normal travel router

  A fresh install starts in travel mode. Open (passwordless) SSIDs are never
  brought up: set a Wi-Fi password, then toggle or run `router-mode travel`.
  Also available as `router-mode home|travel|toggle|status`.
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
| hold 2-5 s | toggle home/travel mode |
| hold 10 s+ | factory reset (power LED starts blinking at 10 s) |

Factory reset is moved up from the stock 5 s so a slow toggle cannot wipe the
box. The stock handler is kept as `/etc/rc.button/reset.stock` and put back if
the package is removed.

## What it configures

Runs once, on first boot of an image or right after `apk add`:

- puts the box in travel mode (Wi-Fi on), unless it already has a mode
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

## Building

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

mwan3 needs kernel modules (`kmod-ipt-*`, `kmod-nft-compat`). These only work
when built with the same kernel as the image, so build the package into the
image rather than installing it later onto an image built without them.
