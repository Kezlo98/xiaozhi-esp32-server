# Deployment Architecture Diagram
![Please refer to - Full Module Installation Architecture Diagram](../docs/images/deploy2.png)
# Method 1: Docker Running All Modules
Starting from version `0.8.2`, the Docker images released by this project only support `x86 architecture`. If you need to deploy on a CPU with `arm64 architecture`, you can follow [this tutorial](docker-build.md) to compile an `arm64 image` locally.

## 1. Install Docker

If your computer doesn't have Docker installed yet, you can follow this tutorial to install it: [Docker Installation](https://www.runoob.com/docker/ubuntu-docker-install.html)

There are two ways to install all modules with Docker. You can use the [lazy script](./Deployment_all.md#11-lazy-script) (author [@VanillaNahida](https://github.com/VanillaNahida))
The script will automatically download the required files and configuration files for you, or you can use [manual deployment](./Deployment_all.md#12-manual-deployment) to build from scratch.



### 1.1 Lazy Script
Simple deployment, you can refer to the [video tutorial](https://www.bilibili.com/video/BV17bbvzHExd/). Text version tutorial is as follows:
> [!NOTE]
> Currently only supports one-click deployment on Ubuntu servers. Other systems have not been tested and may have some strange bugs.

Use an SSH tool to connect to the server and execute the following script with root privileges:
```bash
sudo bash -c "$(wget -qO- https://ghfast.top/https://raw.githubusercontent.com/xinnan-tech/xiaozhi-esp32-server/main/docker-setup.sh)"
```

The script will automatically complete the following operations:
> 1. Install Docker
> 2. Configure image sources
> 3. Download/pull images
> 4. Download speech recognition model files
> 5. Guide server configuration
>

After execution, complete simple configuration, then refer to [4. Run Program](#4-run-program) and the 3 most important things mentioned in [5. Restart xiaozhi-esp32-server](#5-restart-xiaozhi-esp32-server). Complete these 3 configurations and you're ready to use.

### 1.2 Manual Deployment

#### 1.2.1 Create Directories

After installation, you need to find a directory to store the configuration files for this project. For example, we can create a new folder called `xiaozhi-server`.

After creating the directory, you need to create a `data` folder and a `models` folder under `xiaozhi-server`. Under `models`, you also need to create a `SenseVoiceSmall` folder.

The final directory structure should look like this:

```
xiaozhi-server
  ├─ data
  ├─ models
     ├─ SenseVoiceSmall
```

#### 1.2.2 Download Speech Recognition Model Files

The speech recognition model for this project uses the `SenseVoiceSmall` model by default for speech-to-text conversion. Because the model is large, it needs to be downloaded separately. After downloading, place the `model.pt` file in the `models/SenseVoiceSmall` directory. Choose one of the two download routes below.

- Route 1: Download from Alibaba ModelScope [SenseVoiceSmall](https://modelscope.cn/models/iic/SenseVoiceSmall/resolve/master/model.pt)
- Route 2: Download from Baidu Netdisk [SenseVoiceSmall](https://pan.baidu.com/share/init?surl=QlgM58FHhYv1tFnUT_A8Sg&pwd=qvna) Extraction code:
  `qvna`


#### 1.2.3 Download Configuration Files

You need to download two configuration files: `docker-compose_all.yaml` and `config_from_api.yaml`. These files need to be downloaded from the project repository.

##### 1.2.3.1 Download docker-compose_all.yaml

Open [this link](../main/xiaozhi-server/docker-compose_all.yml) in your browser.

On the right side of the page, find the button named `RAW`. Next to the `RAW` button, find the download icon and click the download button to download the `docker-compose_all.yml` file. Download the file to your `xiaozhi-server` directory.

Or directly execute `wget https://raw.githubusercontent.com/xinnan-tech/xiaozhi-esp32-server/refs/heads/main/main/xiaozhi-server/docker-compose_all.yml` to download.

After downloading, continue with this tutorial.

##### 1.2.3.2 Download config_from_api.yaml

Open [this link](../main/xiaozhi-server/config_from_api.yaml) in your browser.

On the right side of the page, find the button named `RAW`. Next to the `RAW` button, find the download icon and click the download button to download the `config_from_api.yaml` file. Download the file to the `data` folder under your `xiaozhi-server` directory, then rename `config_from_api.yaml` to `.config.yaml`.

Or directly execute `wget https://raw.githubusercontent.com/xinnan-tech/xiaozhi-esp32-server/refs/heads/main/main/xiaozhi-server/config_from_api.yaml` to download and save.

After downloading the configuration files, let's confirm that the files in the entire `xiaozhi-server` directory look like this:

```
xiaozhi-server
  ├─ docker-compose_all.yml
  ├─ data
    ├─ .config.yaml
  ├─ models
     ├─ SenseVoiceSmall
       ├─ model.pt
```

If your file directory structure matches the above, continue to the next step. If not, check carefully to see if you missed any steps.

## 2. Backup Data

If you have successfully run the Control Panel before and have your API key information saved there, please copy the important data from the Control Panel first. Because during the upgrade process, the original data may be overwritten.

## 3. Remove Old Version Images and Containers
Next, open the command line tool, use `Terminal` or `Command Prompt` to navigate to your `xiaozhi-server` directory, and execute the following commands:

```
docker compose -f docker-compose_all.yml down

docker stop xiaozhi-esp32-server
docker rm xiaozhi-esp32-server

docker stop xiaozhi-esp32-server-web
docker rm xiaozhi-esp32-server-web

docker stop xiaozhi-esp32-server-db
docker rm xiaozhi-esp32-server-db

docker stop xiaozhi-esp32-server-redis
docker rm xiaozhi-esp32-server-redis

docker rmi ghcr.nju.edu.cn/xinnan-tech/xiaozhi-esp32-server:server_latest
docker rmi ghcr.nju.edu.cn/xinnan-tech/xiaozhi-esp32-server:web_latest
```

## 4. Run Program
Execute the following command to start the new version containers:

```
docker compose -f docker-compose_all.yml up -d
```

After execution, run the following command to view the log information:

```
docker logs -f xiaozhi-esp32-server-web
```

When you see the output log, it means your `Control Panel` has started successfully.

```
2025-xx-xx 22:11:12.445 [main] INFO  c.a.d.s.b.a.DruidDataSourceAutoConfigure - Init DruidDataSource
2025-xx-xx 21:28:53.873 [main] INFO  xiaozhi.AdminApplication - Started AdminApplication in 16.057 seconds (process running for 17.941)
http://localhost:8002/xiaozhi/doc.html
```

Please note that at this moment only the `Control Panel` is running. If port 8000 `xiaozhi-esp32-server` reports an error, ignore it for now.

At this point, you need to use a browser to open the `Control Panel`, link: http://127.0.0.1:8002, and register the first user. The first user is the super administrator, and subsequent users are regular users. Regular users can only bind devices and configure agents; super administrators can perform model management, user management, parameter configuration, and other functions.

Next, there are three important things to do:

### The First Important Thing

Using the super administrator account, log in to the Control Panel, find `Parameter Management` in the top menu, find the first row of data in the list where the parameter code is `server.secret`, and copy its `Parameter Value`.

A note about `server.secret`: this `Parameter Value` is very important. Its purpose is to allow our `Server` to connect to `manager-api`. `server.secret` is a randomly generated key that is automatically created each time you deploy the manager module from scratch.

After copying the `Parameter Value`, open the `.config.yaml` file in the `data` directory under `xiaozhi-server`. At this point, your configuration file content should look like this:

```
manager-api:
  url:  http://127.0.0.1:8002/xiaozhi
  secret: your-server.secret-value
```
1. Copy the `server.secret` `Parameter Value` you just copied from the `Control Panel` to the `secret` field in the `.config.yaml` file.

2. Because you are deploying with Docker, change `url` to `http://xiaozhi-esp32-server-web:8002/xiaozhi`

3. Because you are deploying with Docker, change `url` to `http://xiaozhi-esp32-server-web:8002/xiaozhi`

4. Because you are deploying with Docker, change `url` to `http://xiaozhi-esp32-server-web:8002/xiaozhi`

It should look something like this:
```
manager-api:
  url: http://xiaozhi-esp32-server-web:8002/xiaozhi
  secret: 12345678-xxxx-xxxx-xxxx-123456789000
```

After saving, continue to do the second important thing.

### The Second Important Thing

Using the super administrator account, log in to the Control Panel, find `Model Configuration` in the top menu, then click `Large Language Model` in the left sidebar, find the first row of data `Zhipu AI`, and click the `Modify` button.
After the modification dialog appears, fill in the `Zhipu AI` API key you registered in the `API Key` field. Then click Save.

## 5. Restart xiaozhi-esp32-server

Next, open the command line tool, use `Terminal` or `Command Prompt` and enter:
```
docker restart xiaozhi-esp32-server
docker logs -f xiaozhi-esp32-server
```
If you can see logs similar to the following, it indicates that the Server has started successfully.

```
25-02-23 12:01:09[core.websocket_server] - INFO - Websocket address is      ws://xxx.xx.xx.xx:8000/xiaozhi/v1/
25-02-23 12:01:09[core.websocket_server] - INFO - =======The above address is a WebSocket protocol address, do not access it with a browser=======
25-02-23 12:01:09[core.websocket_server] - INFO - To test WebSocket, please open test_page.html in the test directory using Google Chrome
25-02-23 12:01:09[core.websocket_server] - INFO - =======================================================
```

Since you are doing a full module deployment, you have two important interfaces that need to be written to the ESP32.

OTA Interface:
```
http://your-host-machine-lan-ip:8002/xiaozhi/ota/
```

Websocket Interface:
```
ws://your-host-machine-ip:8000/xiaozhi/v1/
```

### The Third Important Thing

Using the super administrator account, log in to the Control Panel, find `Parameter Management` in the top menu, find the parameter code `server.websocket`, and enter your `Websocket Interface`.

Using the super administrator account, log in to the Control Panel, find `Parameter Management` in the top menu, find the parameter code `server.ota`, and enter your `OTA Interface`.

Next, you can start working with your ESP32 device. You can either `compile your own ESP32 firmware` or configure and use `firmware version 1.6.1 or above compiled by Brother Xia`. Choose one of the two options:

1. [Compile your own ESP32 firmware](firmware-build.md).

2. [Configure custom server based on Brother Xia's pre-compiled firmware](firmware-setting.md).


# Method 2: Local Source Code Running All Modules

## 1. Install MySQL Database

If you already have MySQL installed on your machine, you can directly create a database named `xiaozhi_esp32_server` in the database.

```sql
CREATE DATABASE xiaozhi_esp32_server CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

If you don't have MySQL yet, you can install MySQL through Docker:

```
docker run --name xiaozhi-esp32-server-db -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -e MYSQL_DATABASE=xiaozhi_esp32_server -e MYSQL_INITDB_ARGS="--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci" -e TZ=Asia/Shanghai -d mysql:latest
```

## 2. Install Redis

If you don't have Redis yet, you can install Redis through Docker:

```
docker run --name xiaozhi-esp32-server-redis -d -p 6379:6379 redis
```

## 3. Run manager-api Program

3.1 Install JDK21 and set JDK environment variables

3.2 Install Maven and set Maven environment variables

3.3 Use VSCode programming tool and install Java environment related plugins

3.4 Use VSCode programming tool to load the manager-api module

In `src/main/resources/application-dev.yml`, configure the database connection information:

```
spring:
  datasource:
    username: root
    password: 123456
```
In `src/main/resources/application-dev.yml`, configure the Redis connection information:
```
spring:
    data:
      redis:
        host: localhost
        port: 6379
        password:
        database: 0
```

3.5 Run the main program

This project is a SpringBoot project. The startup method is:
Open `Application.java` and run the `Main` method to start.

```
Path:
src/main/java/xiaozhi/AdminApplication.java
```

When you see the output log, it means your `manager-api` has started successfully.

```
2025-xx-xx 22:11:12.445 [main] INFO  c.a.d.s.b.a.DruidDataSourceAutoConfigure - Init DruidDataSource
2025-xx-xx 21:28:53.873 [main] INFO  xiaozhi.AdminApplication - Started AdminApplication in 16.057 seconds (process running for 17.941)
http://localhost:8002/xiaozhi/doc.html
```

## 4. Run manager-web Program

4.1 Install Node.js

4.2 Use VSCode programming tool to load the manager-web module

Use terminal commands to enter the manager-web directory:

```
npm install
```
Then start:
```
npm run serve
```

Please note, if your manager-api interface is not at `http://localhost:8002`, please modify the path in `main/manager-web/.env.development` during development.

After running successfully, you need to use a browser to open the `Control Panel`, link: http://127.0.0.1:8001, and register the first user. The first user is the super administrator, and subsequent users are regular users. Regular users can only bind devices and configure agents; super administrators can perform model management, user management, parameter configuration, and other functions.


Important: After successful registration, using the super administrator account, log in to the Control Panel, find `Model Configuration` in the top menu, then click `Large Language Model` in the left sidebar, find the first row of data `Zhipu AI`, and click the `Modify` button.
After the modification dialog appears, fill in the `Zhipu AI` API key you registered in the `API Key` field. Then click Save.

Important: After successful registration, using the super administrator account, log in to the Control Panel, find `Model Configuration` in the top menu, then click `Large Language Model` in the left sidebar, find the first row of data `Zhipu AI`, and click the `Modify` button.
After the modification dialog appears, fill in the `Zhipu AI` API key you registered in the `API Key` field. Then click Save.

Important: After successful registration, using the super administrator account, log in to the Control Panel, find `Model Configuration` in the top menu, then click `Large Language Model` in the left sidebar, find the first row of data `Zhipu AI`, and click the `Modify` button.
After the modification dialog appears, fill in the `Zhipu AI` API key you registered in the `API Key` field. Then click Save.

## 5. Install Python Environment

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

## 6. Install Project Dependencies

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

### 7. Download Speech Recognition Model Files

The speech recognition model for this project uses the `SenseVoiceSmall` model by default for speech-to-text conversion. Because the model is large, it needs to be downloaded separately. After downloading, place the `model.pt` file in the `models/SenseVoiceSmall` directory. Choose one of the two download routes below.

- Route 1: Download from Alibaba ModelScope [SenseVoiceSmall](https://modelscope.cn/models/iic/SenseVoiceSmall/resolve/master/model.pt)
- Route 2: Download from Baidu Netdisk [SenseVoiceSmall](https://pan.baidu.com/share/init?surl=QlgM58FHhYv1tFnUT_A8Sg&pwd=qvna) Extraction code:
  `qvna`

## 8. Configure Project Files

Using the super administrator account, log in to the Control Panel, find `Parameter Management` in the top menu, find the first row of data in the list where the parameter code is `server.secret`, and copy its `Parameter Value`.

A note about `server.secret`: this `Parameter Value` is very important. Its purpose is to allow our `Server` to connect to `manager-api`. `server.secret` is a randomly generated key that is automatically created each time you deploy the manager module from scratch.

If your `xiaozhi-server` directory doesn't have a `data` folder, you need to create one.
If there is no `.config.yaml` file under `data`, you can copy the `config_from_api.yaml` file from the `xiaozhi-server` directory to `data` and rename it to `.config.yaml`.

After copying the `Parameter Value`, open the `.config.yaml` file in the `data` directory under `xiaozhi-server`. At this point, your configuration file content should look like this:

```
manager-api:
  url: http://127.0.0.1:8002/xiaozhi
  secret: your-server.secret-value
```

Copy the `server.secret` `Parameter Value` you just copied from the `Control Panel` to the `secret` field in the `.config.yaml` file.

It should look something like this:
```
manager-api:
  url: http://127.0.0.1:8002/xiaozhi
  secret: 12345678-xxxx-xxxx-xxxx-123456789000
```

## 5. Run the Project

```
# Make sure to execute in the xiaozhi-server directory
conda activate xiaozhi-esp32-server
python app.py
```

If you can see logs similar to the following, it indicates that the project service has started successfully.

```
25-02-23 12:01:09[core.websocket_server] - INFO - Server is running at ws://xxx.xx.xx.xx:8000/xiaozhi/v1/
25-02-23 12:01:09[core.websocket_server] - INFO - =======The above address is a WebSocket protocol address, do not access it with a browser=======
25-02-23 12:01:09[core.websocket_server] - INFO - To test WebSocket, please open test_page.html in the test directory using Google Chrome
25-02-23 12:01:09[core.websocket_server] - INFO - =======================================================
```

Since you are doing a full module deployment, you have two important interfaces.

OTA Interface:
```
http://your-computer-lan-ip:8002/xiaozhi/ota/
```

Websocket Interface:
```
ws://your-computer-lan-ip:8000/xiaozhi/v1/
```

Please make sure to write the above two interface addresses to the Control Panel: they will affect WebSocket address distribution and automatic upgrade functionality.

1. Using the super administrator account, log in to the Control Panel, find `Parameter Management` in the top menu, find the parameter code `server.websocket`, and enter your `Websocket Interface`.

2. Using the super administrator account, log in to the Control Panel, find `Parameter Management` in the top menu, find the parameter code `server.ota`, and enter your `OTA Interface`.


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
