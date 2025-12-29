# ESP32 Firmware Compilation

## Step 1: Prepare Your OTA Address

If you are using version 0.3.12 or later of this project, whether using Simple Server Deployment or Full Module Deployment, you will have an OTA address.

Since the OTA address configuration differs between Simple Server Deployment and Full Module Deployment, please choose the appropriate method below:

### If You Are Using Simple Server Deployment
Open your OTA address in a browser. For example, my OTA address is:
```
http://192.168.1.25:8003/xiaozhi/ota/
```
If it displays "OTA interface running normally, the websocket address sent to devices is: ws://xxx:8000/xiaozhi/v1/"

You can use the project's built-in `test_page.html` to test whether you can connect to the websocket address shown on the OTA page.

If you cannot access it, you need to modify the `server.websocket` address in the configuration file `.config.yaml`, restart, and test again until `test_page.html` works properly.

After success, proceed to Step 2.

### If You Are Using Full Module Deployment
Open your OTA address in a browser. For example, my OTA address is:
```
http://192.168.1.25:8002/xiaozhi/ota/
```

If it displays "OTA interface running normally, websocket cluster count: X", proceed to Step 2.

If it displays "OTA interface not running properly", you probably haven't configured the `Websocket` address in the `Control Panel`. Then:

- 1. Log in to the Control Panel as super administrator

- 2. Click `Parameter Management` in the top menu

- 3. Find the `server.websocket` item in the list and enter your `Websocket` address. For example, mine is:

```
ws://192.168.1.25:8000/xiaozhi/v1/
```

After configuration, refresh your OTA interface address in the browser to see if it's working properly. If it still doesn't work, verify that Websocket is running properly and that the Websocket address is configured.

## Step 2: Configure Environment
First, follow this tutorial to configure your project environment: [Setting up ESP IDF 5.3.2 Development Environment on Windows and Compiling Xiaozhi](https://icnynnzcwou8.feishu.cn/wiki/JEYDwTTALi5s2zkGlFGcDiRknXf)

## Step 3: Open Configuration File
After setting up the build environment, download the xiaozhi-esp32 project source code.

Download the [xiaozhi-esp32 project source code](https://github.com/78/xiaozhi-esp32) from here.

After downloading, open the `xiaozhi-esp32/main/Kconfig.projbuild` file.

## Step 4: Modify OTA Address

Find the `default` value of `OTA_URL` and change `https://api.tenclass.net/xiaozhi/ota/`
   to your own address. For example, if my interface address is `http://192.168.1.25:8002/xiaozhi/ota/`, I would change it to that.

Before modification:
```
config OTA_URL
    string "Default OTA URL"
    default "https://api.tenclass.net/xiaozhi/ota/"
    help
        The application will access this URL to check for new firmwares and server address.
```
After modification:
```
config OTA_URL
    string "Default OTA URL"
    default "http://192.168.1.25:8002/xiaozhi/ota/"
    help
        The application will access this URL to check for new firmwares and server address.
```

## Step 5: Set Build Parameters

Set the build parameters:

```
# Navigate to the xiaozhi-esp32 root directory in terminal
cd xiaozhi-esp32
# For example, if I'm using an esp32s3 board, I set the build target to esp32s3. If your board is a different model, replace it with the corresponding model
idf.py set-target esp32s3
# Enter menu configuration
idf.py menuconfig
```

After entering menu configuration, go to `Xiaozhi Assistant` and set `BOARD_TYPE` to your specific board model.
Save and exit, then return to the terminal command line.

## Step 6: Compile Firmware

```
idf.py build
```

## Step 7: Package Bin Firmware

```
cd scripts
python release.py
```

After the packaging command completes, the firmware file `merged-binary.bin` will be generated in the `build` directory under the project root.
This `merged-binary.bin` is the firmware file to be flashed to the hardware.

Note: If you get a "zip" related error after executing the second command, you can ignore this error. As long as the firmware file `merged-binary.bin` is generated in the `build` directory, it won't affect you significantly. Please continue.

## Step 8: Flash Firmware
Connect the ESP32 device to your computer and open the following URL in Chrome browser:

```
https://espressif.github.io/esp-launchpad/
```

Open this tutorial: [Flash Tool/Web-based Firmware Flashing (Without IDF Development Environment)](https://ccnphfhqs21z.feishu.cn/wiki/Zpz4wXBtdimBrLk25WdcXzxcnNS).
Navigate to: `Method 2: ESP-Launchpad Browser Web Flashing`, starting from `3. Flash Firmware/Download to Development Board`, and follow the tutorial.

After successful flashing and network connection, wake up Xiaozhi using the wake word and monitor the console output from the server.

## FAQ
Here are some common questions for reference:

[1. Why does Xiaozhi recognize my speech as Korean, Japanese, or English?](./FAQ.md)

[2. Why does "TTS task error, file not found" appear?](./FAQ.md)

[3. TTS frequently fails or times out](./FAQ.md)

[4. Can connect to self-hosted server via WiFi, but cannot connect in 4G mode](./FAQ.md)

[5. How to improve Xiaozhi's dialogue response speed?](./FAQ.md)

[6. I speak slowly, and Xiaozhi keeps interrupting during pauses](./FAQ.md)

[7. I want to control lights, air conditioning, remote power on/off through Xiaozhi](./FAQ.md)
