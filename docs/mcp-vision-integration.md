# Vision Model Integration Guide
This tutorial is divided into two parts:
- Part 1: Enabling vision model when running xiaozhi-server in single module mode
- Part 2: How to enable vision model when running in full module mode

Before enabling the vision model, you need to prepare three things:
- You need a device with a camera, and this device has implemented the camera calling function in Xiage's repository. For example, `LiChuang ESP32-S3 Development Board`
- Your device firmware version is upgraded to 1.6.6 or above
- You have successfully run the basic conversation module

## Enabling Vision Model in Single Module Mode

### Step 1: Confirm Network
The vision model will use port 8003 by default.

If you're running with Docker, please confirm whether your `docker-compose.yml` has exposed port `8003`. If not, update to the latest `docker-compose.yml` file.

If you're running from source, confirm whether the firewall allows port `8003`.

### Step 2: Select Your Vision Model
Open your `data/.config.yaml` file and set your `selected_module.VLLM` to a vision model. Currently, we support vision models with `openai` type interfaces. `ChatGLMVLLM` is one such model compatible with `openai`.

```
selected_module:
  VAD: ..
  ASR: ..
  LLM: ..
  VLLM: ChatGLMVLLM
  TTS: ..
  Memory: ..
  Intent: ..
```

Assuming we use `ChatGLMVLLM` as the vision model, we first need to log in to the [Zhipu AI](https://bigmodel.cn/usercenter/proj-mgmt/apikeys) website to apply for an API key. If you have already applied for a key before, you can reuse it.

In your configuration file, add this configuration. If it already exists, just set your api_key.

```
VLLM:
  ChatGLMVLLM:
    api_key: your_api_key
```

### Step 3: Start xiaozhi-server Service
If you're running from source, enter the command to start:
```
python app.py
```
If you're running with Docker, restart the container:
```
docker restart xiaozhi-esp32-server
```

After starting, the following logs will be output:

```
2025-06-01 **** - OTA endpoint is           http://192.168.4.7:8003/xiaozhi/ota/
2025-06-01 **** - Vision analysis endpoint is        http://192.168.4.7:8003/mcp/vision/explain
2025-06-01 **** - Websocket address is       ws://192.168.4.7:8000/xiaozhi/v1/
2025-06-01 **** - =======The above addresses are websocket protocol addresses, do not access via browser=======
2025-06-01 **** - To test websocket, open test_page.html in the test directory with Chrome browser
2025-06-01 **** - =============================================================
```

After starting, use your browser to open the `Vision analysis endpoint` link from the logs. See what it outputs. If you're on Linux without a browser, you can execute this command:
```
curl -i your_vision_analysis_endpoint
```

Normally, it will display:
```
MCP Vision interface is running normally, vision explain endpoint address is: http://xxxx:8003/mcp/vision/explain
```

Please note, if you're deploying on public network or with Docker, you must modify this configuration in your `data/.config.yaml`:
```
server:
  vision_explain: http://your-ip-or-domain:port/mcp/vision/explain
```

Why? Because the vision explain endpoint needs to be sent to the device. If your address is a local network address or a Docker internal address, the device cannot access it.

Assuming your public IP is `111.111.111.111`, then `vision_explain` should be configured like this:

```
server:
  vision_explain: http://111.111.111.111:8003/mcp/vision/explain
```

If your MCP Vision interface is running normally and you've successfully opened the delivered `vision explain endpoint address` in your browser, please continue to the next step.

### Step 4: Wake Device to Enable

Say to the device "Please turn on the camera and tell me what you see"

Pay attention to the xiaozhi-server log output to see if there are any errors.


## How to Enable Vision Model in Full Module Mode

### Step 1: Confirm Network
The vision model will use port 8003 by default.

If you're running with Docker, please confirm whether your `docker-compose_all.yml` has mapped port `8003`. If not, update to the latest `docker-compose_all.yml` file.

If you're running from source, confirm whether the firewall allows port `8003`.

### Step 2: Confirm Your Configuration File

Open your `data/.config.yaml` file and confirm whether your configuration file structure matches `data/config_from_api.yaml`. If not, or if items are missing, please complete them.

### Step 3: Configure Vision Model API Key

We first need to log in to the [Zhipu AI](https://bigmodel.cn/usercenter/proj-mgmt/apikeys) website to apply for an API key. If you have already applied for a key before, you can reuse it.

Log in to the `Smart Console`, click `Model Configuration` in the top menu, click `Vision Large Language Model` in the left sidebar, find `VLLM_ChatGLMVLLM`, click the modify button, enter your API key in the `API Key` field in the popup, and click save.

After saving successfully, go to the agent you want to test, click `Configure Role`, and in the opened content, check whether `Vision Large Language Model (VLLM)` has selected the vision model you just configured. Click save.

### Step 3: Start xiaozhi-server Module
If you're running from source, enter the command to start:
```
python app.py
```
If you're running with Docker, restart the container:
```
docker restart xiaozhi-esp32-server
```

After starting, the following logs will be output:

```
2025-06-01 **** - Vision analysis endpoint is        http://192.168.4.7:8003/mcp/vision/explain
2025-06-01 **** - Websocket address is       ws://192.168.4.7:8000/xiaozhi/v1/
2025-06-01 **** - =======The above addresses are websocket protocol addresses, do not access via browser=======
2025-06-01 **** - To test websocket, open test_page.html in the test directory with Chrome browser
2025-06-01 **** - =============================================================
```

After starting, use your browser to open the `Vision analysis endpoint` link from the logs. See what it outputs. If you're on Linux without a browser, you can execute this command:
```
curl -i your_vision_analysis_endpoint
```

Normally, it will display:
```
MCP Vision interface is running normally, vision explain endpoint address is: http://xxxx:8003/mcp/vision/explain
```

Please note, if you're deploying on public network or with Docker, you must modify this configuration in your `data/.config.yaml`:
```
server:
  vision_explain: http://your-ip-or-domain:port/mcp/vision/explain
```

Why? Because the vision explain endpoint needs to be sent to the device. If your address is a local network address or a Docker internal address, the device cannot access it.

Assuming your public IP is `111.111.111.111`, then `vision_explain` should be configured like this:

```
server:
  vision_explain: http://111.111.111.111:8003/mcp/vision/explain
```

If your MCP Vision interface is running normally and you've successfully opened the delivered `vision explain endpoint address` in your browser, please continue to the next step.

### Step 4: Wake Device to Enable

Say to the device "Please turn on the camera and tell me what you see"

Pay attention to the xiaozhi-server log output to see if there are any errors.

