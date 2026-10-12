# qcom-firmwares

Device-proprietary firmware blobs for the [novadeck](https://github.com/Nova-Deck/novadeck)
SteamOS-on-Qualcomm project — GPU zap shaders, adsp/cdsp DSP images, the AudioReach audio
topology, ath12k Wi-Fi/BT, and the Renesas xHCI blob. These are Qualcomm/vendor blobs lifted
from each device's own Android vendor image; the **open** firmware (Adreno microcode, qca BT,
Iris VPU) lives in upstream [linux-firmware](https://gitlab.com/kernel-firmware/linux-firmware)
instead and is fetched separately by novadeck.

## Layout

The tree mirrors `/lib/firmware` directly — blobs are already namespaced by their on-device
path (e.g. `qcom/sm8650/...`), so there is no per-SoC top-level directory:

    sha256sums.txt    # per-file sha256; paths are /lib/firmware-relative
    <blobs...>        # installed verbatim under /lib/firmware on the device

Boards currently covered:

  * **SM8650** (Snapdragon 8 Gen 3): AYANEO Pocket S2, KONKR Pocket FIT.
  * **SM8550** (Snapdragon 8 Gen 2): AYN Odin 2 / 2 Mini / 2 Portal / Thor,
    AYANEO Pocket series, Retroid Pocket 6.
  * **SM8250** (Snapdragon 865): AYN Thor Lite. The other five SM8250 boards
    (Retroid Pocket 5 / Flip2 / Mini / Mini V2, MANGMI Pocket Max) need nothing
    from this repo — unlike the newer SoCs, their GPU zap, adsp and cdsp are all
    open and come from linux-firmware.
  * **SM6115** (Snapdragon 662): MANGMI Air X / Air Y. The MANGMI-signed GPU zap,
    cDSP and modem (which carries the WLAN firmware, `wlanmdsp.mbn`) under
    `qcom/sm6115/mangmi/`, the per-board WCN3950 Wi-Fi board files under
    `ath10k/WCN3990/hw1.0/mangmi-air-{x,y}/`, the WCN3950 Bluetooth blobs under `qca/`,
    the Himax HX83112A touch firmware and the AW88261 amp calibration. These came from
    [ROCKNIX/extra-firmware](https://github.com/ROCKNIX/extra-firmware) at
    `925dfb5efbe5d6c3ca654f81e3e1f367f8492777` (its `SM6115/` directory), byte for byte.
    The aDSP is the open QRB4210 one and stays in linux-firmware.

Blobs are per-device even when two boards happen to share identical bytes — a device must
never reference another device's path, or a later per-device fix silently regresses it.
The AYN Thor Lite's `aw883xx_acf.bin` is exactly that case: byte-identical to the SM8550
AYN Thor's, and stored separately anyway.
The exception is a file whose path the driver fixes rather than the DTS: `Himax_firmware.bin`,
`aw88261_acf.bin` and the `qca/` Bluetooth blobs sit where their drivers ask for them, and only
the SM6115 MANGMI boards use them today.

Known gap: `qcom/sm8250/slpi.mbn`, the sensor DSP every SM8250 board's DTS requests. It is
not in linux-firmware (which carries only the unrelated Thundercomm RB5 signing) and has not
been extracted from a device vendor image yet, so those boards have no accelerometer.

## Consumed by novadeck

novadeck's `firmware/fetch-qcom-fw.sh` fetches this repo at the commit pinned in novadeck's
`firmware/QCOM_FW.pin`, then verifies every file against `sha256sums.txt` before staging it
for the kernel build and the read-only rootfs.
