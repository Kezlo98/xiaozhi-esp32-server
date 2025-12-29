# MCP Endpoint Integration Guide

This tutorial uses Xiage's open-source MCP calculator feature as an example to introduce how to integrate your custom MCP service into your endpoint.

The prerequisite for this tutorial is that your `xiaozhi-server` has already enabled the MCP endpoint feature. If you haven't enabled it yet, you can first follow [this tutorial](./mcp-endpoint-enable.md) to enable it.

# How to Integrate a Simple MCP Feature for Your Agent, Such as Calculator

### If You Have Full Module Deployment
If you have full module deployment, you can enter the Smart Console, go to Agent Management, click `Configure Role`, and on the right side of `Intent Recognition`, there is an `Edit Features` button.

Click this button. In the popup page, at the bottom, there will be `MCP Endpoint`. Normally, it will display this agent's `MCP Endpoint Address`. Next, we'll extend a calculator feature based on MCP technology for this agent.

This `MCP Endpoint Address` is very important, you'll use it later.

### If You Have Single Module Deployment
If you have single module deployment and you have already configured the MCP endpoint address in the configuration file, then normally, when starting the single module deployment, the following logs will be output:
```
250705[__main__]-INFO-Initializing component: vad success SileroVAD
250705[__main__]-INFO-Initializing component: asr success FunASRServer
250705[__main__]-INFO-OTA endpoint is          http://192.168.1.25:8002/xiaozhi/ota/
250705[__main__]-INFO-Vision analysis endpoint is     http://192.168.1.25:8002/mcp/vision/explain
250705[__main__]-INFO-MCP endpoint is        ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=abc
250705[__main__]-INFO-Websocket address is    ws://192.168.1.25:8000/xiaozhi/v1/
250705[__main__]-INFO-=======The above addresses are websocket protocol addresses, do not access via browser=======
250705[__main__]-INFO-To test websocket, open test_page.html in the test directory with Chrome browser
250705[__main__]-INFO-=============================================================
```

As shown above, the `ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=abc` in the `MCP endpoint is` output is your `MCP Endpoint Address`.

This `MCP Endpoint Address` is very important, you'll use it later.

## Step 1: Download Xiage's MCP Calculator Project Code

Open Xiage's [calculator project](https://github.com/78/mcp-calculator) in your browser.

Once opened, find the green button labeled `Code`, click it, and you'll see the `Download ZIP` button.

Click it to download the project source code archive. After downloading to your computer, extract it. At this point, it might be named `mcp-calculatorr-main`.
You need to rename it to `mcp-calculator`. Next, we'll use the command line to enter the project directory and install dependencies.


```bash
# Enter project directory
cd mcp-calculator

conda remove -n mcp-calculator --all -y
conda create -n mcp-calculator python=3.10 -y
conda activate mcp-calculator

pip install -r requirements.txt
```

## Step 2: Start

Before starting, copy the MCP endpoint address from your agent in the Smart Console.

For example, my agent's MCP address is:
```
ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=abc
```

Enter the command:

```bash
export MCP_ENDPOINT=ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=abc
```

After entering, start the program:

```bash
python mcp_pipe.py calculator.py
```

### If You Have Smart Console Deployment
If you have Smart Console deployment, after starting, enter the Smart Console again, click refresh MCP connection status, and you'll see your extended feature list.

### If You Have Single Module Deployment
If you have single module deployment, when the device connects, logs similar to the following will be output, indicating success:

```
250705 -INFO-Initializing MCP endpoint: wss://2662r3426b.vicp.fun/mcp_e
250705 -INFO-Sending MCP endpoint initialization message
250705 -INFO-MCP endpoint connected successfully
250705 -INFO-MCP endpoint initialized successfully
250705 -INFO-Unified tool handler initialization complete
250705 -INFO-MCP endpoint server info: name=Calculator, version=1.9.4
250705 -INFO-Number of tools supported by MCP endpoint: 1
250705 -INFO-All MCP endpoint tools retrieved, client ready
250705 -INFO-Tool cache refreshed
250705 -INFO-Current supported function list: [ 'get_time', 'get_lunar', 'play_music', 'get_weather', 'handle_exit_intent', 'calculator']
```
If it includes `'calculator'`, it means the device will be able to call the calculator tool based on intent recognition.
