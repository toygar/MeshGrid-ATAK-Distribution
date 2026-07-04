# MeshGrid ATAK Distribution

Official **binary-only** distribution for connecting **ATAK-CIV (Android)** to the **MeshGrid** LoRa mesh network via a dedicated ESP32 node.

This repository contains **prebuilt firmware and plugin packages only**. It does **not** include source code, protocol specifications, or cryptographic implementation details.

**Product site:** [meshgrid.org](https://meshgrid.org)  
**Core node firmware & hardware wiring:** [MeshGrid-Node-Firmware](https://github.com/toygar/MeshGrid-Node-Firmware)

---

## Package contents

| File | Description |
|------|-------------|
| `MeshGrid_ESP32_ATAK_BUILD1005.bin` | ESP32 application firmware (BUILD 1005) |
| `ATAK-Plugin-MeshGrid-0.4.0-ATAK-5.5.1-civ-release.apk` | ATAK-CIV plugin (release, minified) |

### SHA-256 checksums

```
MeshGrid_ESP32_ATAK_BUILD1005.bin
  77983cc7b1eb061b6426411874246562d41469dcb8bd050b6f7f24f235680c47

ATAK-Plugin-MeshGrid-0.4.0-ATAK-5.5.1-civ-release.apk
  147d4b3cc2a6df5f47c7429514be2368a6111821e9c014a4781566464648afcb
```

---

## Requirements

### Hardware

Same node hardware as MeshGrid-Node:

- ESP32 DevKit (4 MB flash)
- EBYTE E22-900T22D LoRa module + suitable antenna
- Optional GPS module and battery sense (see [hardware docs](https://github.com/toygar/MeshGrid-Node-Firmware))

### Software

- **ATAK-CIV** **5.5.1** (CIV) on Android — plugin API must match this version
- **Python 3** + **esptool** for firmware flashing (`pip install esptool`)
- USB data cable to the ESP32

---

## 1. Flash ESP32 firmware

### Find the serial port

**macOS:** `ls /dev/cu.usb*`

**Linux:** `ls /dev/ttyUSB* /dev/ttyACM*`

**Windows:** Device Manager → COM port

### Flash (application image at `0x10000`)

Use this when the board already has a compatible MeshGrid partition layout, or after a previous MeshGrid/ATAK PlatformIO flash:

```bash
esptool.py --chip esp32 --port <PORT> --baud 460800 \
  write_flash -z 0x10000 MeshGrid_ESP32_ATAK_BUILD1005.bin
```

**Blank ESP32:** flash bootloader + partitions + app from a full MeshGrid install, or use the official MeshGrid-Node factory image first, then upgrade with the command above. See [MeshGrid-Node-Firmware installation](https://github.com/toygar/MeshGrid-Node-Firmware#firmware-installation).

### Manual bootloader mode

If flashing fails: hold **BOOT**, tap **EN**, keep **BOOT** 2–3 s, start flash, release **BOOT** when upload begins.

### Verify boot

Serial monitor **115200 baud**:

```text
[MeshGrid] boot BUILD 1005
[BLE] mesh advertising started name=MESHGRID_ATAK
```

---

## 2. Provision the ESP32 node

The node must use the **same mesh network password** as your other MeshGrid nodes.

Provisioning is done over USB serial (**115200**). Follow the MeshGrid node provisioning workflow supplied with your mesh deployment (password, optional device name, optional node allow-list).

After provisioning:

- Note the **6-digit BLE pairing PIN** provided by your provisioning process
- Note the node’s **`node=`** ID from the serial boot log (needed for roster on Core nodes)

---

## 3. Install ATAK plugin

1. Install **ATAK-CIV 5.5.1** on the Android device
2. Install the plugin APK:

   ```bash
   adb install -r ATAK-Plugin-MeshGrid-0.4.0-ATAK-5.5.1-civ-release.apk
   ```

3. ATAK → **Settings → Tool Preferences → Installed Plugins** → enable **MeshGrid**
4. Open **Tools → MeshGrid**

---

## 4. Connect and use

1. Enter your **mesh network password** in the plugin panel
2. Confirm the **BLE pairing PIN** matches your provisioned node
3. **Scan** → select **`MESHGRID_ATAK`** (or your provisioned name) → **Connect**
4. Accept Bluetooth pairing with the PIN
5. Mesh positions appear on the ATAK map; use native ATAK **GeoChat** (All Chat Rooms) for mesh messaging

**GPS source:** NODE (ESP32 GPS), PHONE, or AUTO — selectable in the plugin panel.

---

## Multi-node roster (Core / iOS meshes)

If your MeshGrid Core nodes use an **authorized-node allow-list**, every Core node must include this ATAK ESP32’s **`node=`** ID in the same roster list.

On each Core node (USB serial **115200**):

```text
roster set <id1>,<id2>,<atak_node_id>
roster
```

Replace IDs with your actual node IDs. Without this step, inbound mesh traffic from Core may work while **outbound chat from ATAK** is blocked.

---

## Troubleshooting

| Issue | Check |
|-------|--------|
| Plugin not visible in ATAK | Installed Plugins → MeshGrid **enabled**; restart ATAK |
| BLE connects but no mesh data | Same mesh password on ESP32; LoRa antenna and wiring |
| ATAK chat not reaching iOS | Core **roster** includes ATAK `node=` ID on every Core node |
| Pairing fails | Forget old Bluetooth bond; use correct 6-digit PIN |
| Wrong ATAK version | Plugin targets **ATAK-CIV 5.5.1** |

Hardware and LoRa issues: [MeshGrid-Node-Firmware troubleshooting](https://github.com/toygar/MeshGrid-Node-Firmware#troubleshooting)

---

## License and redistribution

These binaries are **proprietary**. Unauthorized copying, modification, reverse engineering, or redistribution is not permitted except as explicitly authorized by the copyright holder.

No source code, protocol documentation, or implementation details are published in this repository.

For licensing, support, or updated builds: [info@meshgrid.org](mailto:info@meshgrid.org) · [meshgrid.org](https://meshgrid.org)
