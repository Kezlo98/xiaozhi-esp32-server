# Single Module Deployment Firmware OTA Auto-Upgrade Configuration Guide

This tutorial will guide you on how to configure firmware OTA auto-upgrade functionality in a **single module deployment** scenario, enabling automatic firmware updates for devices.

If you are already using **full module deployment**, please ignore this tutorial.

## Feature Introduction

In single module deployment, xiaozhi-server has built-in OTA firmware management functionality that can automatically detect device versions and deliver upgrade firmware. The system will automatically match and push the latest firmware version based on the device model and current version.

## Prerequisites

- You have successfully completed **single module deployment** and are running xiaozhi-server
- The device can normally connect to the server

## Step One: Prepare Firmware Files

### 1. Create Firmware Storage Directory

Firmware files need to be placed in the `data/bin/` directory. If this directory does not exist, please create it manually:

```bash
mkdir -p data/bin
```

### 2. Firmware File Naming Rules

Firmware files must follow this naming format:

```
{device_model}_{version}.bin
```

**Naming Rules Explanation:**
- `device_model`: The model name of the device, e.g., `lichuang-dev`, `bread-compact-wifi`, etc.
- `version`: Firmware version number, must start with a number, supports numbers, letters, periods, underscores, and hyphens, e.g., `1.6.6`, `2.0.0`, etc.
- File extension must be `.bin`

**Naming Examples:**
```
bread-compact-wifi_1.6.6.bin
lichuang-dev_2.0.0.bin
```

### 3. Place Firmware Files

Copy the prepared firmware files (.bin files) to the `data/bin/` directory:

Important note repeated three times: The upgrade bin file is `xiaozhi.bin`, NOT the full firmware file `merged-binary.bin`!

Important note repeated three times: The upgrade bin file is `xiaozhi.bin`, NOT the full firmware file `merged-binary.bin`!

Important note repeated three times: The upgrade bin file is `xiaozhi.bin`, NOT the full firmware file `merged-binary.bin`!

```bash
cp xiaozhi.bin data/bin/device_model_version.bin
```

For example:
```bash
cp xiaozhi.bin data/bin/bread-compact-wifi_1.6.6.bin
```

## Step Two: Configure Public Network Access Address (Only Required for Public Network Deployment)

**Note: This step only applies to single module public network deployment scenarios.**

If your xiaozhi-server is deployed on the public network (using public IP or domain name), you **must** configure the `server.vision_explain` parameter, because the OTA firmware download address will use the domain and port from this configuration.

If you are deploying on a LAN, you can skip this step.

### Why Configure This Parameter?

In single module deployment, when the system generates firmware download addresses, it uses the domain and port configured in `vision_explain` as the base address. If not configured or configured incorrectly, devices will not be able to access the firmware download address.

### Configuration Method

Open the `data/.config.yaml` file, find the `server` configuration section, and set the `vision_explain` parameter:

```yaml
server:
  vision_explain: http://your-domain-or-IP:port/mcp/vision/explain
```

**Configuration Examples:**

LAN deployment (default):
```yaml
server:
  vision_explain: http://192.168.1.100:8003/mcp/vision/explain
```

Public network domain deployment:
```yaml
server:
  vision_explain: http://yourdomain.com:8003/mcp/vision/explain
```

### Important Notes

- The domain or IP must be an address accessible by the device
- If using Docker deployment, do not use Docker internal addresses (such as 127.0.0.1 or localhost)
- If you are using nginx reverse proxy, fill in the external address and port number, not the port number this project runs on


## Frequently Asked Questions

### 1. Device Does Not Receive Firmware Update

**Possible Causes and Solutions:**

- Check if the firmware file naming follows the rules: `{model}_{version}.bin`
- Check if the firmware file is correctly placed in the `data/bin/` directory
- Check if the device model matches the model in the firmware filename
- Check if the firmware version number is higher than the device's current version
- Check server logs to confirm if OTA requests are being processed normally

### 2. Device Reports Download Address Not Accessible

**Possible Causes and Solutions:**

- Check if the domain or IP configured in `server.vision_explain` is correct
- Confirm the port number is configured correctly (default 8003)
- If deployed on public network, ensure the device can access that public address
- If using Docker deployment, ensure you are not using internal addresses (127.0.0.1)
- Check if the firewall has opened the corresponding port
- If you are using nginx reverse proxy, fill in the external address and port number, not the port number this project runs on

### 3. How to Confirm Device Current Version

Check the OTA request logs, which will display the version number reported by the device:

```
[ota_handler] - Device AA:BB:CC:DD:EE:FF firmware is up to date: 1.6.6
```

### 4. Firmware File Not Taking Effect After Placement

The system has a 30-second cache time (default), you can:
- Wait 30 seconds before having the device initiate an OTA request
- Restart the xiaozhi-server service
- Adjust the `firmware_cache_ttl` configuration to a shorter time
