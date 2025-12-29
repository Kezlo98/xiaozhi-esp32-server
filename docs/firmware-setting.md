# Configuring Custom Server with Pre-compiled Firmware

## Step 1: Confirm Version
Flash the pre-compiled [firmware version 1.6.1 or above](https://github.com/78/xiaozhi-esp32/releases)

## Step 2: Prepare Your OTA Address
If you followed the tutorial using Full Module Deployment, you should have an OTA address.

Open your OTA address in a browser. For example, my OTA address is:
```
https://2662r3426b.vicp.fun/xiaozhi/ota/
```

If it displays "OTA interface running normally, websocket cluster count: X", proceed to the next step.

If it displays "OTA interface not running properly", you probably haven't configured the `Websocket` address in the `Control Panel`. Then:

- 1. Log in to the Control Panel as super administrator

- 2. Click `Parameter Management` in the top menu

- 3. Find the `server.websocket` item in the list and enter your `Websocket` address. For example, mine is:

```
wss://2662r3426b.vicp.fun/xiaozhi/v1/
```

After configuration, refresh your OTA interface address in the browser to see if it's working properly. If it still doesn't work, verify that Websocket is running properly and that the Websocket address is configured.

## Step 3: Enter Network Configuration Mode
Enter the device's network configuration mode. At the top of the page, click "Advanced Options", enter your server's `ota` address, and click save. Restart the device.
![Reference - OTA Address Setting](../docs/images/firmware-setting-ota.png)

## Step 4: Wake Up Xiaozhi and Check Log Output

Wake up Xiaozhi and check if the logs are outputting normally.


## FAQ
Here are some common questions for reference:

[1. Why does Xiaozhi recognize my speech as Korean, Japanese, or English?](./FAQ.md)

[2. Why does "TTS task error, file not found" appear?](./FAQ.md)

[3. TTS frequently fails or times out](./FAQ.md)

[4. Can connect to self-hosted server via WiFi, but cannot connect in 4G mode](./FAQ.md)

[5. How to improve Xiaozhi's dialogue response speed?](./FAQ.md)

[6. I speak slowly, and Xiaozhi keeps interrupting during pauses](./FAQ.md)

[7. I want to control lights, air conditioning, remote power on/off through Xiaozhi](./FAQ.md)
