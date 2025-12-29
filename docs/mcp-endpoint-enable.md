# MCP Endpoint Deployment Guide

This tutorial contains 3 parts:
- 1. How to deploy the MCP endpoint service
- 2. How to configure the MCP endpoint for full module deployment
- 3. How to configure the MCP endpoint for single module deployment

# 1. How to Deploy the MCP Endpoint Service

## Step 1: Download the MCP endpoint project source code

Open [MCP endpoint project repository](https://github.com/xinnan-tech/mcp-endpoint-server) in your browser.

Once opened, find the green button labeled `Code`, click it, and you'll see the `Download ZIP` button.

Click it to download the project source code archive. After downloading to your computer, extract it. At this point, it might be named `mcp-endpoint-server-main`.
You need to rename it to `mcp-endpoint-server`.

## Step 2: Start the program
This is a simple project, and it's recommended to run it with Docker. However, if you don't want to use Docker, you can refer to [this page](https://github.com/xinnan-tech/mcp-endpoint-server/blob/main/README_dev.md) to run from source code. Below is the Docker method:

```
# Enter the project source root directory
cd mcp-endpoint-server

# Clear cache
docker compose -f docker-compose.yml down
docker stop mcp-endpoint-server
docker rm mcp-endpoint-server
docker rmi ghcr.nju.edu.cn/xinnan-tech/mcp-endpoint-server:latest

# Start docker container
docker compose -f docker-compose.yml up -d
# View logs
docker logs -f mcp-endpoint-server
```

At this point, the logs will output something similar to the following:
```
250705 INFO-=====The following addresses are for Smart Console/Single Module MCP endpoint====
250705 INFO-Smart Console MCP parameter configuration: http://172.22.0.2:8004/mcp_endpoint/health?key=abc
250705 INFO-Single module deployment MCP endpoint: ws://172.22.0.2:8004/mcp_endpoint/mcp/?token=def
250705 INFO-=====Please choose based on your specific deployment, do not share with anyone======
```

Please copy both endpoint addresses:

Since you're deploying with Docker, do NOT use the above addresses directly!

Since you're deploying with Docker, do NOT use the above addresses directly!

Since you're deploying with Docker, do NOT use the above addresses directly!

First, copy the addresses to a draft. You need to know your computer's local network IP. For example, my computer's local IP is `192.168.1.25`, so:
Originally my endpoint addresses:
```
Smart Console MCP parameter configuration: http://172.22.0.2:8004/mcp_endpoint/health?key=abc
Single module deployment MCP endpoint: ws://172.22.0.2:8004/mcp_endpoint/mcp/?token=def
```
Should be changed to:
```
Smart Console MCP parameter configuration: http://192.168.1.25:8004/mcp_endpoint/health?key=abc
Single module deployment MCP endpoint: ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=def
```

After making changes, use your browser to directly access the `Smart Console MCP parameter configuration` URL. When the browser displays code similar to this, it means success:
```
{"result":{"status":"success","connections":{"tool_connections":0,"robot_connections":0,"total_connections":0}},"error":null,"id":null,"jsonrpc":"2.0"}
```

Please keep both `endpoint addresses` handy, you'll need them in the next step.

# 2. How to Configure MCP Endpoint for Full Module Deployment
First, you need to enable the MCP endpoint feature. In the Smart Console, click `Parameter Dictionary` at the top, then click `System Feature Configuration` in the dropdown menu. On the page, check `MCP Endpoint` and click `Save Configuration`. On the `Role Configuration` page, click the `Edit Features` button to see the `MCP endpoint` feature.

If you have full module deployment, use the administrator account to log in to the Smart Console, click `Parameter Dictionary` at the top, and select `Parameter Management`.

Then search for parameter `server.mcp_endpoint`. At this point, its value should be `null`.
Click the modify button, paste the `Smart Console MCP parameter configuration` URL from the previous step into `Parameter Value`, then save.

If it saves successfully, everything is working, and you can go to the agent to see the effect. If it doesn't work, it means the Smart Console cannot access the MCP endpoint, most likely due to network firewall issues or incorrect local network IP.

# 3. How to Configure MCP Endpoint for Single Module Deployment

If you have single module deployment, find your configuration file `data/.config.yaml`.
Search for `mcp_endpoint` in the configuration file. If not found, add the `mcp_endpoint` configuration. For example:
```
server:
  websocket: ws://your-ip-or-domain:port/xiaozhi/v1/
  http_port: 8002
log:
  log_level: INFO

# There may be more configurations here..

mcp_endpoint: your-endpoint-websocket-address
```
Now, paste the `Single module deployment MCP endpoint` URL obtained from `How to Deploy the MCP Endpoint Service` into `mcp_endpoint`. Like this:

```
server:
  websocket: ws://your-ip-or-domain:port/xiaozhi/v1/
  http_port: 8002
log:
  log_level: INFO

# There may be more configurations here

mcp_endpoint: ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=def
```

After configuration, starting the single module will output the following logs:
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

As shown above, if the output shows `MCP endpoint is` with `ws://192.168.1.25:8004/mcp_endpoint/mcp/?token=abc`, the configuration is successful.

