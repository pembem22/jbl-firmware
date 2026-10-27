# JBL Firmware Files
This repository contains firmware files for JBL Bluetooth speakers. All of those are official and were downloaded from the [JBL server](http://storage.harman.com/), except for the Charge 2 one, which I got from somewhere else online I don't remember.

*NOTE: All firmware files marked as 8.8.8, 888 or similar are only for testing purposes and were never released publicly.*

Some models have different chipsets (CSR, QCC, or Vimicro), so make sure to use correct files from corresponding directories. You can use [ConnectPlus](https://github.com/pembem22/connect-plus/) app to check what chipset speaker has.

## How to find new firmware files on the JBL server

The official JBL Portable app (Android package `com.harman.ble.jbllink`) doesn't use any authentication, API key or signature to fetch firmware. It just downloads a small JSON config for your speaker from a public server, reads the latest firmware filename out of it, and downloads that file from the same server. So you can grab any current firmware with nothing but `curl`.

Every speaker is identified by a 4‑hex‑digit **PID** (e.g. Boombox 4 = `214e`, Charge 5 = `2040`). There are two delivery schemes:

**1. Newer models — "dynamic OTA"** (Boombox 4, Charge 6, Flip 7, Grip, Go 5, Xtreme 5, Tuner 3, Flip Essential 3, …). Base host `https://appstorage.onecloud.harman.com/jbl_portable/ota/`:

```
config:   https://appstorage.onecloud.harman.com/jbl_portable/ota/<PID>/ota_config.json
firmware: https://appstorage.onecloud.harman.com/jbl_portable/ota/<PID>/<bin>
```

The config's `firmware.bin` field is the filename; `firmware.version` and `firmware.md5` let you verify it. Example for Boombox 4 (`214e`):

```bash
curl -s https://appstorage.onecloud.harman.com/jbl_portable/ota/214e/ota_config.json
# -> {"pid":"214e","name":"JBL Boombox 4","firmware":{"version":"26.16.10",
#     "md5":"21f1bafe41b875e37ede09fed0a6d803","bin":"Boombox4_v261610_260414-ota.bin", ...}}
curl -O https://appstorage.onecloud.harman.com/jbl_portable/ota/214e/Boombox4_v261610_260414-ota.bin
```

**2. Older models — "legacy OTA"** (Charge 5, Flip 6, Pulse 4/5, Xtreme 3/4, Clip 5, Go 4, Boombox 2/3, and the CSR/QCC speakers). Base host `https://storage.harman.com/release/portable/`:

```
config:   https://storage.harman.com/release/portable/<model>/<model>_ota.json
firmware: <the binUrl field inside that config>   # already an absolute URL
```

Example for Charge 5:

```bash
curl -s https://storage.harman.com/release/portable/charge5/charge5_ota.json
# -> {"version":"0.8.9.0","targetPID":"2040","md5":"...",
#     "binUrl":"https://storage.harman.com/release/portable/charge5/Charge5_ota_all0890.bin", ...}
```

The full PID → config-path table for all ~37 speakers lives in the app asset `assets/product_list_config.json` (each entry has a `pid`, an `otaUrl`, and a `dynamicOTAUrl` flag). PIDs are also listed in the bundled `DAConfig_Product.xml`.

Always verify the download against the `md5` in the config before trusting it.

### Two app channels: `jbl_portable` and `jbl_one`

The newest speakers (Boombox 4, Xtreme 5, Grip, Go 5, Flip/Charge Essential 3, …) are also served by the **JBL One** app, on the same host under a different path segment — `jbl_one/` instead of `jbl_portable/`:

```
https://appstorage.onecloud.harman.com/jbl_one/ota/<PID>/ota_config.json
```

The two channels can point at **different** versions. In practice the latest release often appears on `jbl_one` first while `jbl_portable` lags on an older build — e.g. for Boombox 4 (`214e`), `jbl_portable` served `26.16.10` while `jbl_one` already had `26.32.10`. Check **both** channels (and both the production and `appstorage-stage` hosts) to find the newest file for a model.

### Release-candidate / staging builds

There is a parallel **staging** host that sometimes carries a newer, not-yet-public build:

```
https://appstorage-stage.onecloud.harman.com/jbl_portable/ota/<PID>/ota_config.json
```

These are unreleased release candidates (treat them like the `8.8.8` test builds — not officially shipped), but it's the only place to catch a version *before* it hits production.

Some files in this repo were pulled from that staging host and are **release candidates, not public releases** — flash them at your own risk. Current ones: Boombox 4 `26.16.11` / `26.30.11` / `26.42.11`, Xtreme 5 `26.40.10`, Flip 7 `0.7.1.0`, Go 5 `26.23.11`, Flip Essential 3 SE `26.25.10`.

**Limitation — why there is no full history here:** the server has directory listing disabled (any unknown path returns `AccessDenied`), the bucket sits behind CloudFront with versioned reads blocked, and each config only ever points at the **single latest** firmware. When a new version ships, the previous file is **deleted** from the server — verified: every superseded filename now returns `403`. So old versions can't be re-fetched, brute-forced, or recovered from JBL at all; they survive only where someone saved the `.bin` while it was current (like this repo). **Practical takeaway: grab the current file for your model now — once it's superseded it's gone.**