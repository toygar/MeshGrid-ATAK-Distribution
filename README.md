# MeshGrid-Node

Official **binary-only** distribution of **MeshGrid** mesh node firmware (**MeshLink v2**).

This repository publishes **installation documentation** and **[GitHub Releases](https://github.com/toygar/MeshGrid-Node/releases)** with prebuilt flash images. It does **not** include source code, protocol specifications, cryptographic implementation details, or hardware/component documentation.

**Product site:** [meshgrid.org](https://meshgrid.org)  
**Repository:** [github.com/toygar/MeshGrid-Node](https://github.com/toygar/MeshGrid-Node)  
**Latest release:** see **[Releases](https://github.com/toygar/MeshGrid-Node/releases)** (current package: **BUILD 135**)

---

## Download

Download the release zip from the **[Releases](https://github.com/toygar/MeshGrid-Node/releases)** page (Assets section). Example for BUILD 135:

| Asset | Description |
|-------|-------------|
| `MeshGrid_ESP32_BUILD135_release.zip` | Full flash package (merged image + app-only image + checksums) |

Do **not** clone this repository to obtain the binaries — use **Releases** only.

Inside the zip you will find (among others):

| File | Use |
|------|-----|
| `MeshGrid_ESP32_BUILD135_flash0x0.bin` | **Preferred** full image — flash at address **0x0** |
| `MeshGrid_ESP32_BUILD135_app_0x10000.bin` | Application-only image — flash at **0x10000** |
| `README_FLASH.txt` | Flash notes for this build |
| `SHA256SUMS.txt` | Checksums |

Verify checksums against `SHA256SUMS.txt` after download.

---

## Requirements

### Software

- **Python 3** + **esptool** for flashing (`pip install esptool`)
- USB data cable to the MeshGrid node
- Serial terminal at **115200** baud (optional; for boot verification and provisioning)

No IDE or SDK is required for end users — install prebuilt images from Releases.

### Network

- The same **mesh network password** on every node (and companion app) that should share one private mesh

---

## 1. Flash firmware

Download `MeshGrid_ESP32_BUILD135_release.zip` from [Releases](https://github.com/toygar/MeshGrid-Node/releases), then unzip.

### Find the serial port

**macOS:** `ls /dev/cu.usb*`

**Linux:** `ls /dev/ttyUSB* /dev/ttyACM*`

**Windows:** Device Manager → COM port

### Full image (recommended)

```bash
esptool.py --chip esp32 --port <PORT> --baud 921600 \
  write_flash -z 0x0 MeshGrid_ESP32_BUILD135_flash0x0.bin
```

### Application-only update

Use when the board already has a compatible MeshGrid partition layout and you only want to replace the application (stored mesh credentials usually survive):

```bash
esptool.py --chip esp32 --port <PORT> --baud 921600 \
  write_flash -z 0x10000 MeshGrid_ESP32_BUILD135_app_0x10000.bin
```

### Verify boot

Serial monitor **115200 baud**:

```text
BLE ready BUILD 135 node=<id> LoRa=enc BLE=enc ...
```

---

## 2. Provision the node

Unprovisioned nodes stay offline from the mesh until a network password is written over USB serial (**115200**).

1. Flash firmware
2. Provision with the **same password** used by your other MeshGrid nodes (and the companion app)
3. Optionally set a short device name and an authorized-node allow-list (roster), per your deployment process
4. Note the **`node=`** ID from the boot log and any BLE pairing PIN your provisioning process provides

**Important:** Mesh credentials and roster data **survive** normal application-only uploads. To change the network password cleanly, erase stored credentials (or full erase), re-flash if needed, then **re-provision every node** that should interoperate.

---

## 3. Connect the companion app

1. Pair the MeshGrid companion app with the node over BLE using your deployment’s pairing policy
2. Enter the **same mesh network password** in the app
3. Confirm chat / map traffic as appropriate for your deployment

---

## Multi-node roster

If your mesh uses an **authorized-node allow-list**, every node must list the same set of allowed `node=` IDs.

On each node (USB serial **115200**), after provisioning:

```text
roster set <id1>,<id2>,<id3>
roster
```

Replace IDs with your actual node IDs. Nodes missing from the roster may be unable to exchange traffic as intended.

---

## Troubleshooting

| Issue | Check |
|-------|--------|
| Flash fails | Correct serial port; retry with the board held in download mode per your USB bridge |
| Boot line missing / wrong BUILD | Re-download release asset; verify SHA-256; re-flash full `flash0x0` image |
| Nodes do not decrypt each other | Same mesh password on all nodes; re-provision after password changes |
| Traffic one-way or blocked | Roster includes every peer `node=` ID where allow-list is enabled |
| BLE pairs but no mesh data | Password match; peer is on-air and provisioned |

Open an issue on [this repository](https://github.com/toygar/MeshGrid-Node/issues) for release-asset or flash problems. Include the `BLE ready BUILD …` boot line and esptool output — **never** paste network passwords or pairing PINs.

---

## License and redistribution

Binaries distributed via [GitHub Releases](https://github.com/toygar/MeshGrid-Node/releases) are **proprietary**. Unauthorized copying, modification, reverse engineering, or redistribution is not permitted except as explicitly authorized by the copyright holder.

No source code, protocol documentation, hardware/component documentation, or implementation details are published in this repository.

For licensing, support, or updated builds: [info@meshgrid.org](mailto:info@meshgrid.org) · [meshgrid.org](https://meshgrid.org)
