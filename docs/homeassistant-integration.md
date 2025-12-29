# Xiaozhi ESP32 Open-Source Server Integration Guide with Home Assistant

[TOC]

-----

## Introduction

This document will guide you on how to integrate ESP32 devices with Home Assistant.

## Prerequisites

- Home Assistant is already installed and configured
- For this tutorial, I chose the model: Free ChatGLM, which supports function call

## Before You Start (Required)

### 1. Get the HA Network Address Information

Please access your Home Assistant network address. For example, my HA address is 192.168.4.7, and the port is the default 8123, so open in browser:

```
http://192.168.4.7:8123
```

> Manual method to find HA's IP address **(only applies when xiaozhi-esp32-server and HA are deployed on the same network device [e.g., same WiFi])**:
>
> 1. Enter Home Assistant (frontend).
>
> 2. Click **Settings** in the bottom left corner → **System** → **Network**.
>
> 3. Scroll to the bottom to the `Home Assistant URL (Home Assistant website)` section. In `Local Network`, click the `eye` button to see the current IP address (e.g., `192.168.1.10`) and network interface. Click `Copy Link` to copy directly.
>
>    ![image-20250504051716417](images/image-ha-integration-01.png)

Or, if you have already set up a directly accessible Home Assistant OAuth address, you can also access it directly in the browser:

```
http://homeassistant.local:8123
```

### 2. Log in to `Home Assistant` to Get the Developer API Key

Log in to `Home Assistant`, click `bottom left avatar -> Personal`, switch to the `Security` tab, scroll to the bottom to `Long-Lived Access Tokens` to generate an api_key, and copy and save it. All subsequent methods require this API key, and it only appears once (tip: you can save the generated QR code image to scan and extract the API key again later).

## Method 1: Xiaozhi Community-Built HA Control Function

### Feature Description

- If you need to add new devices later, this method requires manually restarting the `xiaozhi-esp32-server` to update device information **(Important)**.

- You need to ensure that you have integrated `Xiaomi Home` in Home Assistant and imported Mijia devices into `Home Assistant`.

- You need to ensure that the `xiaozhi-esp32-server console` works properly.

- My `xiaozhi-esp32-server console` and `Home Assistant` are deployed on the same machine on different ports, version is `0.3.10`

  ```
  http://192.168.4.7:8002
  ```


### Configuration Steps

#### 1. Log in to `Home Assistant` and Organize the Device List to Control

Log in to `Home Assistant`, click `Settings` in the bottom left corner, then enter `Devices & Services`, and click `Entities` at the top.

Then search for the switches you want to control in entities. After the results appear, click one of the results in the list, and a switch interface will appear.

In the switch interface, try clicking the switch to see if it turns on/off as you click. If it works, it means the connection is normal.

Then find the settings button on the switch panel, click it, and you can view the `Entity ID` of this switch.

Open a notepad and organize the data in this format:

Location + comma + Device name + comma + `Entity ID` + semicolon

For example, I'm at the office, I have a toy light, and its identifier is switch.cuco_cn_460494544_cp1_on_p_2_1, then write this entry:

```
Office,Toy Light,switch.cuco_cn_460494544_cp1_on_p_2_1;
```

Of course, I may want to control two lights in the end, my final result is:

```
Office,Toy Light,switch.cuco_cn_460494544_cp1_on_p_2_1;
Office,Desk Lamp,switch.iot_cn_831898993_socn1_on_p_2_1;
```

This string, which we call the "device list string", needs to be saved for later use.

#### 2. Log in to the `Console`

![image-20250504051716417](images/image-ha-integration-06.png)

Log in to the `Console` with an admin account. In `Agent Management`, find your agent, then click `Configure Role`.

Set intent recognition to `External LLM Intent Recognition` or `LLM Autonomous Function Call`. You will see an `Edit Functions` option on the right. Click the `Edit Functions` button, and a `Function Management` dialog will pop up.

In the `Function Management` dialog, you need to check `HomeAssistant Device Status Query` and `HomeAssistant Device Status Modification`.

After checking, click `HomeAssistant Device Status Query` in `Selected Functions`, then configure your `Home Assistant` address, API key, and device list string in `Parameter Configuration`.

After editing, click `Save Configuration`. The `Function Management` dialog will hide, then click save on the agent configuration.

After saving successfully, you can wake up the device for operation.

#### 3. Wake Up the Device for Control

Try saying to the ESP32, "Turn on XXX light"

## Method 2: Use Home Assistant's Voice Assistant as Xiaozhi's LLM Tool

### Feature Description

- This method has a significant drawback — **this method cannot use Xiaozhi's open-source ecosystem function_call plugin capabilities**, because using Home Assistant as Xiaozhi's LLM tool transfers intent recognition capabilities to Home Assistant. However, **this method allows you to experience native Home Assistant operation features, and Xiaozhi's chat capabilities remain unchanged**. If you really mind, you can use [Method 3](##Method 3: Using Home Assistant's MCP Service (Recommended)) which is also supported by Home Assistant and allows you to experience Home Assistant's features to the fullest extent.

### Configuration Steps:

#### 1. Configure Home Assistant's LLM Voice Assistant.

**You need to configure Home Assistant's voice assistant or LLM tool in advance.**

#### 2. Get Home Assistant's Voice Assistant Agent ID.

1. Enter the Home Assistant page. Click `Developer Tools` on the left.
2. In the opened `Developer Tools`, click the `Actions` tab (as shown in operation 1), find or type `conversation.process` in the `Action` dropdown and select `Conversation: Process` (as shown in operation 2).

![image-20250504043539343](images/image-ha-integration-02.png)

3. On the page, check the `Agent` option. In the now-active `Conversation Agent` dropdown, select the voice assistant name you configured in step one. As shown in the image, I configured `ZhipuAi` and selected it.

![image-20250504043854760](images/image-ha-integration-03.png)

4. After selection, click `Go to YAML mode` at the bottom left of the form.

![image-20250504043951126](images/image-ha-integration-04.png)

5. Copy the agent-id value. For example, in the image, mine is `01JP2DYMBDF7F4ZA2DMCF2AGX2` (for reference only).

![image-20250504044046466](images/image-ha-integration-05.png)

6. Switch to the xiaozhi-esp32-server open-source server's `config.yaml` file. In the LLM configuration, find Home Assistant, and set your Home Assistant network address, API key, and the agent_id you just obtained.
7. Modify the `selected_module` property in the `config.yaml` file: set `LLM` to `HomeAssistant` and `Intent` to `nointent`.
8. Restart the xiaozhi-esp32-server open-source server to use it normally.

## Method 3: Using Home Assistant's MCP Service (Recommended)

### Feature Description

- You need to integrate and install the HA integration — [Model Context Protocol Server](https://www.home-assistant.io/integrations/mcp_server/) — in Home Assistant beforehand.

- This method and Method 2 are both official solutions provided by HA. Unlike Method 2, you can normally use the open-source community plugins of the xiaozhi-esp32-server, while also allowing you to use any LLM that supports function_call.

### Configuration Steps

#### 1. Install the Home Assistant MCP Service Integration.

Official integration website — [Model Context Protocol Server](https://www.home-assistant.io/integrations/mcp_server/).

Or follow the manual steps below.

> - Go to **[Settings > Devices & Services](https://my.home-assistant.io/redirect/integrations)** in your Home Assistant page.
>
> - In the bottom right corner, select the **[Add Integration](https://my.home-assistant.io/redirect/config_flow_start?domain=mcp_server)** button.
>
> - Select **Model Context Protocol Server** from the list.
>
> - Follow the on-screen instructions to complete the setup.

#### 2. Configure Xiaozhi Open-Source Server MCP Configuration

Enter the `data` directory and find the `.mcp_server_settings.json` file.

If your `data` directory doesn't have a `.mcp_server_settings.json` file,
- Copy the `mcp_server_settings.json` file from the `xiaozhi-server` root folder to the `data` directory and rename it to `.mcp_server_settings.json`
- Or [download this file](https://github.com/xinnan-tech/xiaozhi-esp32-server/blob/main/main/xiaozhi-server/mcp_server_settings.json), download it to the `data` directory, and rename it to `.mcp_server_settings.json`


Modify the content in `"mcpServers"`:

```json
"Home Assistant": {
      "command": "mcp-proxy",
      "args": [
        "http://YOUR_HA_HOST/mcp_server/sse"
      ],
      "env": {
        "API_ACCESS_TOKEN": "YOUR_API_ACCESS_TOKEN"
      }
},
```

Notes:

1. **Replace configuration:**
   - Replace `YOUR_HA_HOST` in `args` with your HA service address. If your service address already includes https/http (e.g., `http://192.168.1.101:8123`), you only need to enter `192.168.1.101:8123`.
   - Replace `YOUR_API_ACCESS_TOKEN` in `env`'s `API_ACCESS_TOKEN` with the developer API key you obtained earlier.
2. **If you're adding this configuration inside `"mcpServers"` brackets and there's no new `mcpServers` configuration afterwards, you need to remove the trailing comma `,`**, otherwise parsing may fail.

**Final result reference (example below)**:

```json
 "mcpServers": {
    "Home Assistant": {
      "command": "mcp-proxy",
      "args": [
        "http://192.168.1.101:8123/mcp_server/sse"
      ],
      "env": {
        "API_ACCESS_TOKEN": "abcd.efghi.jkl"
      }
    }
  }
```

#### 3. Configure Xiaozhi Open-Source Server System Configuration

1. **Choose any LLM that supports function_call as Xiaozhi's LLM chat assistant (but don't choose Home Assistant as the LLM tool)**. For this tutorial, I chose the model: Free ChatGLM, which supports function call, but sometimes the calls are not very stable. If you want stability, it's recommended to set LLM to: DoubaoLLM, with the specific model_name: doubao-1-5-pro-32k-250115.

2. Switch to the xiaozhi-esp32-server open-source server's `config.yaml` file, set your LLM configuration, and adjust the `Intent` in `selected_module` to `function_call`.

3. Restart the xiaozhi-esp32-server open-source server to use it normally.
