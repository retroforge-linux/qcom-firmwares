# qcom-firmwares

Device-proprietary firmware blobs for [RetroForge](https://github.com/retroforge-linux), a
retro-emulation Linux distribution for Qualcomm handhelds — GPU zap shaders, adsp/cdsp DSP
images, modem/WLAN images, the AudioReach audio topology, ath10k/ath12k Wi-Fi and Bluetooth,
touch and amplifier firmware, and the Renesas xHCI blob. These are Qualcomm/vendor blobs lifted
from each device's own Android vendor image; the **open** firmware (Adreno microcode, WCN7850
qca BT, Iris VPU) lives in upstream [linux-firmware](https://gitlab.com/kernel-firmware/linux-firmware)
instead and is fetched separately by RetroForge. Blobs upstream has not taken yet (the WCN7860
Wi-Fi/BT set) live here until they land there.

## Layout

The tree mirrors `/lib/firmware` directly — blobs are already namespaced by their on-device
path (e.g. `qcom/sm8650/...`), so there is no per-SoC top-level directory:

    sha256sums.txt    # per-file sha256; paths are /lib/firmware-relative
    <blobs...>        # installed verbatim under /lib/firmware on the device

Boards currently covered:

  * **SM8750** (Snapdragon 8 Elite): AYN Odin 3, KONKR Pocket FIT Elite. Also carries
    the shared WCN7860 Wi-Fi (`ath12k/WCN7860/`) and Bluetooth (`qca/gngbt*`) blobs —
    unlike WCN7850, this chip's firmware is not in linux-firmware yet. The SM8750 GPU
    zap shader and the `vpu35` Iris firmware ARE upstream (20260810), so they are
    deliberately absent here. `soccp.mbn` is also deliberately absent: the SoCCP
    remoteproc only serves suspend-to-RAM, which RetroForge does not ship.
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
  * **SM4450** (Snapdragon G1 Gen 2): Retroid Pocket Classic (all revisions), Anbernic
    RG 55G1. The A613 GPU zap under `qcom/sm4450/`, each board's aDSP, WPSS (which carries
    the WCN3990 WLAN firmware) and AW88166 amp calibration under
    `qcom/sm4450/{RPClassic,RG55G1}/`, both AudioReach topologies under `qcom/sm8450/` (the
    card driver reports sm8450), and `qca/cmnv13t.bin`. These came from
    [ROCKNIX/extra-firmware](https://github.com/ROCKNIX/extra-firmware) at
    `2486a4167ec2be163ef4d8ee01f34b0b9c5cf44a` (its `SM4450/` directory), byte for byte.
    ROCKNIX's SM4450 `qca/cmbtfw13.*` and `cmnv13s.bin` are a newer WCN3950 Bluetooth build
    than the SM6115 ones at the same fixed paths, so they are not taken: one image can hold
    one, and replacing it would change the MANGMI boards. `cmnv13t.bin` is byte-identical to
    `cmnv13.bin`.

Blobs are per-device even when two boards happen to share identical bytes — a device must
never reference another device's path, or a later per-device fix silently regresses it.
The AYN Thor Lite's `aw883xx_acf.bin` is exactly that case: byte-identical to the SM8550
AYN Thor's, and stored separately anyway.
The exception is a file whose path the driver fixes rather than the DTS: `Himax_firmware.bin`,
`aw88261_acf.bin` and the `qca/` Bluetooth blobs sit where their drivers ask for them, and only
the SM6115 MANGMI boards use them today (the SM4450 boards share the `qca/` ones).

Known gap: `qcom/sm8250/slpi.mbn`, the sensor DSP every SM8250 board's DTS requests. It is
not in linux-firmware (which carries only the unrelated Thundercomm RB5 signing) and has not
been extracted from a device vendor image yet, so those boards have no accelerometer.

## Consumed by RetroForge

RetroForge's `firmware/fetch-qcom-fw.sh` fetches this repo at the commit pinned in its
`firmware/QCOM_FW.pin`, then verifies every file against `sha256sums.txt` before staging it
for the kernel build and the read-only rootfs. Adding a blob means adding the file at its
`/lib/firmware` path plus its line in `sha256sums.txt`, then bumping that pin.

## Origin

This repo started as a history-free import of
[Nova-Deck/qcom-firmwares](https://github.com/Nova-Deck/qcom-firmwares) at
`dbf238bd885d5858543b9588fbee9d0645a25d4f`, which provides the SM8650, SM8550 and SM8250
blobs above. The SM6115 and SM4450 blobs came from ROCKNIX/extra-firmware, as noted in their
entries.
