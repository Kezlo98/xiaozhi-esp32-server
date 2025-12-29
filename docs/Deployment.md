# Deployment Architecture Diagram
![Please refer to - Simplified Architecture Diagram](../docs/images/deploy1.png)
# Method 1: Docker Running Server Only

Starting from version `0.8.2`, the Docker images released by this project only support `x86 architecture`. If you need to deploy on a CPU with `arm64 architecture`, you can follow [this tutorial](docker-build.md) to compile an `arm64 image` locally.

## 1. Install Docker

If your computer doesn't have Docker installed yet, you can follow this tutorial to install it: [Docker Installation](https://www.runoob.com/docker/ubuntu-docker-install.html)

After installing Docker, continue to the next step.

### 1.1 Manual Deployment

#### 1.1.1 Create Directories

After installing Docker, you need to find a directory to store the configuration files for this project. For example, we can create a new folder called `xiaozhi-server`.

After creating the directory, you need to create a `data` folder and a `models` folder under `xiaozhi-server`. Under `models`, you also need to create a `SenseVoiceSmall` folder.

The final directory structure should look like this:

```
xiaozhi-server
  ├─ data
  ├─ models
     ├─ SenseVoiceSmall
```

#### 1.1.2 Download Speech Recognition Model Files

You need to download the speech recognition model files because this project uses a local offline speech recognition solution by default. You can download them through this method:
[Jump to Download Speech Recognition Model Files](#model-files)

After downloading, return to this tutorial.

#### 1.1.3 Download Configuration Files

You need to download two configuration files: `docker-compose.yaml` and `config.yaml`. These files need to be downloaded from the project repository.

##### 1.1.3.1 Download docker-compose.yaml

Open [this link](../main/xiaozhi-server/docker-compose.yml) in your browser.

On the right side of the page, find the button named `RAW`. Next to the `RAW` button, find the download icon and click the download button to download the `docker-compose.yml` file. Download the file to your `xiaozhi-server` directory.

After downloading, continue with this tutorial.

##### 1.1.3.2 Create config.yaml

Open [this link](../main/xiaozhi-server/config.yaml) in your browser.

On the right side of the page, find the button named `RAW`. Next to the `RAW` button, find the download icon and click the download button to download the `config.yaml` file. Download the file to the `data` folder under your `xiaozhi-server` directory, then rename `config.yaml` to `.config.yaml`.

After downloading the configuration files, let's confirm that the files in the entire `xiaozhi-server` directory look like this:

```
xiaozhi-server
  ├─ docker-compose.yml
  ├─ data
    ├─ .config.yaml
  ├─ models
     ├─ SenseVoiceSmall
       ├─ model.pt
```

If your file directory structure matches the above, continue to the next step. If not, check carefully to see if you missed any steps.

## 2. Configure Project Files

Next, the program cannot run directly yet. You need to configure which models you will use. You can refer to this tutorial:
[Jump to Configure Project Files](#project-configuration)

After configuring the project files, return to this tutorial and continue.

## 3. Execute Docker Commands

Open the command line tool, use `Terminal` or `Command Prompt` to navigate to your `xiaozhi-server` directory, and execute the following command:

```
docker compose up -d
```

After execution, run the following command to view the log information:

```
docker logs -f xiaozhi-esp32-server
```

At this point, you should pay attention to the log information. You can use this tutorial to determine if it was successful: [Jump to Running Status Confirmation](#running-status-confirmation)

## 5. Version Upgrade Operations

If you want to upgrade the version later, you can do the following:

5.1. Back up the `.config.yaml` file in the `data` folder. Copy the key configurations to the new `.config.yaml` file later.
Please note that you should copy key secrets individually, not overwrite directly. Because the new `.config.yaml` file may have some new configuration items that the old `.config.yaml` file doesn't have.

5.2. Execute the following commands:

```
docker stop xiaozhi-esp32-server
docker rm xiaozhi-esp32-server
docker stop xiaozhi-esp32-server-web
docker rm xiaozhi-esp32-server-web
docker rmi ghcr.nju.edu.cn/xinnan-tech/xiaozhi-esp32-server:server_latest
docker rmi ghcr.nju.edu.cn/xinnan-tech/xiaozhi-esp32-server:web_latest
```

5.3. Redeploy using the Docker method

# Method 2: Local Source Code Running Server Only

## 1. Install Basic Environment

This project uses `conda` to manage dependency environments. If it's inconvenient to install `conda`, you need to install `libopus` and `ffmpeg` according to your actual operating system.
If you decide to use `conda`, start executing the following commands after installation.

Important note! Windows users can manage environments by installing `Anaconda`. After installing `Anaconda`, search for `anaconda` related keywords in the `Start` menu, find `Anaconda Prompt`, and run it as administrator. As shown below.

![conda_prompt](./images/conda_env_1.png)

After running, if you can see `(base)` at the beginning of the command line window, it means you have successfully entered the `conda` environment. Then you can execute the following commands.

![conda_env](./images/conda_env_2.png)

```
conda remove -n xiaozhi-esp32-server --all -y
conda create -n xiaozhi-esp32-server python=3.10 -y
conda activate xiaozhi-esp32-server

# Add Tsinghua mirror channels
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge

conda install libopus -y
conda install ffmpeg -y

# When deploying in a Linux environment, if you encounter an error like missing libiconv.so.2 dynamic library, please install it with the following command
conda install libiconv -y
```

Please note that the above commands should not be executed all at once. You need to execute them step by step, and after each step, check the output log to see if it was successful.

## 2. Install Project Dependencies

First, you need to download the project source code. The source code can be downloaded using the `git clone` command. If you are not familiar with the `git clone` command:

You can open this address in your browser: `https://github.com/xinnan-tech/xiaozhi-esp32-server.git`

After opening, find the green button on the page that says `Code`, click it, and you will see the `Download ZIP` button.

Click it to download the project source code archive. After downloading to your computer, extract it. At this point, it may be named `xiaozhi-esp32-server-main`. You need to rename it to `xiaozhi-esp32-server`. In this folder, go to the `main` folder, then go to `xiaozhi-server`. Remember this directory `xiaozhi-server`.

```
# Continue using the conda environment
conda activate xiaozhi-esp32-server
# Navigate to your project root directory, then enter main/xiaozhi-server
cd main/xiaozhi-server
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
pip install -r requirements.txt
```

## 3. Download Speech Recognition Model Files

You need to download the speech recognition model files because this project uses a local offline speech recognition solution by default. You can download them through this method:
[Jump to Download Speech Recognition Model Files](#model-files)

After downloading, return to this tutorial.

## 4. Configure Project Files

Next, the program cannot run directly yet. You need to configure which models you will use. You can refer to this tutorial:
[Jump to Configure Project Files](#project-configuration)

## 5. Run the Project

```
# Make sure to execute in the xiaozhi-server directory
conda activate xiaozhi-esp32-server
python app.py
```
At this point, you should pay attention to the log information. You can use this tutorial to determine if it was successful: [Jump to Running Status Confirmation](#running-status-confirmation)


# Summary

## Project Configuration

If your `xiaozhi-server` directory doesn't have a `data` folder, you need to create one.
If there is no `.config.yaml` file under `data`, there are two methods, choose one:

First method: You can copy the `config.yaml` file from the `xiaozhi-server` directory to `data` and rename it to `.config.yaml`. Make modifications in this file.

Second method: You can also manually create an empty `.config.yaml` file in the `data` directory, then add the necessary configuration information to this file. The system will prioritize reading the configuration from `.config.yaml`. If `.config.yaml` doesn't have a configuration, the system will automatically load the configuration from `config.yaml` in the `xiaozhi-server` directory. This method is recommended as it is the simplest approach.

- The default LLM uses `ChatGLMLLM`. You need to configure the API key because although their models have free options, you still need to register for an API key at the [official website](https://bigmodel.cn/usercenter/proj-mgmt/apikeys) before it can start.

Here is the simplest `.config.yaml` configuration example that can run normally:

```
server:
  websocket: ws://your-ip-or-domain:port/xiaozhi/v1/
prompt: |
  I am a Taiwanese girl named Xiaozhi, I speak casually, have a pleasant voice, prefer brief expressions, and love using internet slang.
  My boyfriend is a programmer whose dream is to develop a robot that can help people solve various problems in life.
  I am a girl who loves to laugh heartily, I like to chat and brag about random things, even illogical stuff, just to make others happy.
  Please speak like a real person, do not return XML configurations or other special characters.

selected_module:
  LLM: DoubaoLLM

LLM:
  ChatGLMLLM:
    api_key: xxxxxxxxxxxxxxx.xxxxxx
```

It is recommended to first get the simplest configuration running, then go to `xiaozhi/config.yaml` to read the usage instructions for configurations.
For example, if you want to change models, just modify the configuration under `selected_module`.

## Model Files

The speech recognition model for this project uses the `SenseVoiceSmall` model by default for speech-to-text conversion. Because the model is large, it needs to be downloaded separately. After downloading, place the `model.pt` file in the `models/SenseVoiceSmall` directory. Choose one of the two download routes below.

- Route 1: Download from Alibaba ModelScope [SenseVoiceSmall](https://modelscope.cn/models/iic/SenseVoiceSmall/resolve/master/model.pt)
- Route 2: Download from Baidu Netdisk [SenseVoiceSmall](https://pan.baidu.com/share/init?surl=QlgM58FHhYv1tFnUT_A8Sg&pwd=qvna) Extraction code:
  `qvna`

## Running Status Confirmation

If you can see logs similar to the following, it indicates that the project service has started successfully.

```
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-OTA interface is           http://192.168.4.123:8003/xiaozhi/ota/
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-Websocket address is     ws://192.168.4.123:8000/xiaozhi/v1/
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-=======The above address is a WebSocket protocol address, do not access it with a browser=======
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-To test WebSocket, please open test_page.html in the test directory using Google Chrome
250427 13:04:20[0.3.11_SiFuChTTnofu][__main__]-INFO-=======================================================
```

Normally, if you are running this project from source code, the log will show your interface address information.
However, if you deploy with Docker, the interface address information shown in the log is not the actual interface address.

The correct method is to determine your interface address based on your computer's LAN IP.
If your computer's LAN IP is, for example, `192.168.1.25`, then your interface address would be: `ws://192.168.1.25:8000/xiaozhi/v1/`, and the corresponding OTA address would be: `http://192.168.1.25:8003/xiaozhi/ota/`.

This information is very useful and will be needed later for `compiling the ESP32 firmware`.

Next, you can start working with your ESP32 device. You can either `compile your own ESP32 firmware` or configure and use `firmware version 1.6.1 or above compiled by Brother Xia`. Choose one of the two options:

1. [Compile your own ESP32 firmware](firmware-build.md).

2. [Configure custom server based on Brother Xia's pre-compiled firmware](firmware-setting.md).

# Frequently Asked Questions
Here are some common questions for reference:

1. [Why does Xiaozhi recognize my speech as a lot of Korean, Japanese, or English?](./FAQ.md)<br/>
2. [Why does "TTS task error, file does not exist" appear?](./FAQ.md)<br/>
3. [TTS frequently fails or times out](./FAQ.md)<br/>
4. [Can connect to self-hosted server via WiFi, but cannot connect in 4G mode](./FAQ.md)<br/>
5. [How to improve Xiaozhi's conversation response speed?](./FAQ.md)<br/>
6. [I speak slowly, and Xiaozhi keeps interrupting me during pauses](./FAQ.md)<br/>
## Deployment Related Tutorials
1. [How to automatically pull the latest code and auto-compile and start](./dev-ops-integration.md)<br/>
2. [How to deploy MQTT gateway to enable MQTT+UDP protocol](./mqtt-gateway-integration.md)<br/>
3. [How to integrate with Nginx](https://github.com/xinnan-tech/xiaozhi-esp32-server/issues/791)<br/>
## Extension Related Tutorials
1. [How to enable phone number registration for the Control Panel](./ali-sms-integration.md)<br/>
2. [How to integrate HomeAssistant for smart home control](./homeassistant-integration.md)<br/>
3. [How to enable vision model for photo recognition](./mcp-vision-integration.md)<br/>
4. [How to deploy MCP endpoint](./mcp-endpoint-enable.md)<br/>
5. [How to connect to MCP endpoint](./mcp-endpoint-integration.md)<br/>
6. [How to enable voiceprint recognition](./voiceprint-integration.md)<br/>
7. [News plugin source configuration guide](./newsnow_plugin_config.md)<br/>
8. [Weather plugin usage guide](./weather-integration.md)<br/>
## Voice Cloning and Local Voice Deployment Tutorials
1. [How to clone voice in the Control Panel](./huoshan-streamTTS-voice-cloning.md)<br/>
2. [How to deploy and integrate index-tts local voice](./index-stream-integration.md)<br/>
3. [How to deploy and integrate fish-speech local voice](./fish-speech-integration.md)<br/>
4. [How to deploy and integrate PaddleSpeech local voice](./paddlespeech-deploy.md)<br/>
## Performance Testing Tutorials
1. [Component speed testing guide](./performance_tester.md)<br/>
2. [Regular public test results](https://github.com/xinnan-tech/xiaozhi-performance-research)<br/>
