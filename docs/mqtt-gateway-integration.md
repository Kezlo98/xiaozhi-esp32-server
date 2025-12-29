# MQTT Gateway Deployment Tutorial

The `xiaozhi-esp32-server` project can be combined with the open-source [xiaozhi-mqtt-gateway](https://github.com/78/xiaozhi-mqtt-gateway) project with simple modifications to enable Xiaozhi hardware MQTT+UDP connection.
This tutorial is divided into three parts. You can choose the corresponding section to integrate MQTT gateway based on whether you are using full module deployment or single module deployment:
- Part One: Deploy MQTT Gateway
- Part Two: Full Module Deployment for Xiaozhi Hardware MQTT+UDP Connection
- Part Three: Single Module Deployment of xiaozhi-server for Xiaozhi Hardware MQTT+UDP Connection

## Preparation Stage
Prepare your `xiaozhi-server` `mqtt-websocket` connection address. Add the `?from=mqtt_gateway` parameter to your original `websocket address` to get the `mqtt-websocket` connection address.

1. If you are deploying from source code, your `mqtt-websocket` address is:
```
ws://127.0.0.1:8000/xiaozhi/v1/?from=mqtt_gateway
```

2. If you are using Docker deployment, your `mqtt-websocket` address is:
```
ws://your-host-machine-LAN-IP:8000/xiaozhi/v1/?from=mqtt_gateway
```

## Important Notice

If you are deploying on a server, you need to ensure that ports `1883`, `8884`, and `8007` are open to the public. Port `8884` uses the `UDP` protocol, while the others use `TCP`.

If you are deploying on a server, you need to ensure that ports `1883`, `8884`, and `8007` are open to the public. Port `8884` uses the `UDP` protocol, while the others use `TCP`.

If you are deploying on a server, you need to ensure that ports `1883`, `8884`, and `8007` are open to the public. Port `8884` uses the `UDP` protocol, while the others use `TCP`.


## Part One: Deploy MQTT Gateway

1. Clone the [modified xiaozhi-mqtt-gateway project](https://github.com/xinnan-tech/xiaozhi-mqtt-gateway.git):
```bash
git clone https://ghfast.top/https://github.com/xinnan-tech/xiaozhi-mqtt-gateway.git
cd xiaozhi-mqtt-gateway
```

2. Install dependencies:
```bash
npm install
npm install -g pm2
```

3. Configure `config.json`:
```bash
cp config/mqtt.json.example config/mqtt.json
```

4. Edit the config file config/mqtt.json, replace the `mqtt-websocket` address from the `Preparation Stage` into `chat_servers`. For example, for source code deployment of `xiaozhi-server`, the configuration is as follows:

```
{
    "production": {
        "chat_servers": [
            "ws://127.0.0.1:8000/xiaozhi/v1/?from=mqtt_gateway"
        ]
    },
    "debug": false,
    "max_mqtt_payload_size": 8192,
    "mcp_client": {
        "capabilities": {
        },
        "client_info": {
            "name": "xiaozhi-mqtt-client",
            "version": "1.0.0"
        },
        "max_tools_count": 128
    }
}
```
5. Create a `.env` file in the project root directory and set the following environment variables:
```
PUBLIC_IP=your-ip         # Server public IP
MQTT_PORT=1883            # MQTT server port
UDP_PORT=8884             # UDP server port
API_PORT=8007             # Management API port
MQTT_SIGNATURE_KEY=test   # MQTT signature key
SERVER_SECRET=Te1st12134  # Server secret, keep consistent with the control panel (server.secret) or xiaozhi-server (server.auth_key)
```
Please note the `PUBLIC_IP` configuration, ensure it matches the actual public IP. If you have a domain name, use the domain name.

`MQTT_SIGNATURE_KEY` is the key for MQTT connection authentication. It's best to set it to something complex, preferably 8 or more characters containing both uppercase and lowercase letters. This key will be used later.

- Do not use simple passwords like `123456`, `test`, etc.
- Do not use simple passwords like `123456`, `test`, etc.
- Do not use simple passwords like `123456`, `test`, etc.

`SERVER_SECRET` is used to generate websocket connection authentication information.

1. If you are using full module deployment and have set `server.auth.enabled` to `true` in the control panel's parameter management, then `SERVER_SECRET` needs to match the control panel's (`server.secret`).

2. If you are using single module deployment and have set `server.auth.enabled` to `true` in the config file, then `SERVER_SECRET` needs to match the config file's (`server.auth_key`).


6. Start MQTT Gateway
```
# Start service
pm2 start ecosystem.config.js

# View logs
pm2 logs xz-mqtt
```

When you see the following logs, the MQTT gateway has started successfully:
```
0|xz-mqtt  | 2025-09-11T12:14:48: MQTT server is listening on port 1883
0|xz-mqtt  | 2025-09-11T12:14:48: UDP server is listening on x.x.x.x:8884
```

If you need to restart the MQTT gateway, execute the following command:
```
pm2 restart xz-mqtt
```

## Part Two: Full Module Deployment for Xiaozhi Hardware MQTT+UDP Connection

Check the version number at the bottom of your control panel homepage to confirm whether your control panel version is `0.7.7` or above. If not, you need to upgrade the control panel.

1. At the top of the control panel, click `Parameter Management`, search for `server.mqtt_gateway`, click edit, and enter the `PUBLIC_IP`+`:`+`MQTT_PORT` you set in the `.env` file. Like this:
```
192.168.0.7:1883
```
2. At the top of the control panel, click `Parameter Management`, search for `server.mqtt_signature_key`, click edit, and enter the `MQTT_SIGNATURE_KEY` you set in the `.env` file.

3. At the top of the control panel, click `Parameter Management`, search for `server.udp_gateway`, click edit, and enter the `PUBLIC_IP`+`:`+`UDP_PORT` you set in the `.env` file. Like this:
```
192.168.0.7:8884
```
4. At the top of the control panel, click `Parameter Management`, search for `server.mqtt_manager_api`, click edit, and enter the `PUBLIC_IP`+`:`+`UDP_PORT` you set in the `.env` file. Like this:
```
192.168.0.7:8007
```

After completing the above configuration, you can use the curl command to verify whether your OTA address will deliver MQTT configuration. Replace `http://localhost:8002/xiaozhi/ota/` with your OTA address:
```
curl 'http://localhost:8002/xiaozhi/ota/' \
  -H 'Content-Type: application/json' \
  -H 'Client-Id: 7b94d69a-9808-4c59-9c9b-704333b38aff' \
  -H 'Device-Id: 11:22:33:44:55:66' \
  --data-raw $'{\n  "application": {\n    "version": "1.0.1",\n    "elf_sha256": "1"\n  },\n  "board": {\n    "mac": "11:22:33:44:55:66"\n  }\n}'
```

If the returned content contains `mqtt` related configuration, the configuration is successful. Like this:

```
{"server_time":{"timestamp":1757567894012,"timeZone":"Asia/Shanghai","timezone_offset":480},"activation":{"code":"460609","message":"http://xiaozhi.server.com\n460609","challenge":"11:22:33:44:55:66"},"firmware":{"version":"1.0.1","url":"http://xiaozhi.server.com:8002/xiaozhi/otaMag/download/NOT_ACTIVATED_FIRMWARE_THIS_IS_A_INVALID_URL"},"websocket":{"url":"ws://192.168.4.23:8000/xiaozhi/v1/"},"mqtt":{"endpoint":"192.168.0.7:1883","client_id":"GID_default@@@11_22_33_44_55_66@@@7b94d69a-9808-4c59-9c9b-704333b38aff","username":"eyJpcCI6IjA6MDowOjA6MDowOjA6MSJ9","password":"Y8XP9xcUhVIN9OmbCHT9ETBiYNE3l3Z07Wk46wV9PE8=","publish_topic":"device-server","subscribe_topic":"devices/p2p/11_22_33_44_55_66"}}
```

Since MQTT information needs to be delivered via the OTA address, as long as you can normally connect to the server's OTA address, just restart and wake up.

After waking up, check the mqtt-gateway logs to confirm whether there are successful connection logs.
```
pm2 logs xz-mqtt
```

## Part Three: Single Module Deployment for Xiaozhi Hardware MQTT+UDP Connection

Open your `data/.config.yaml` file, find `mqtt_gateway` under `server` and enter the `PUBLIC_IP`+`:`+`MQTT_PORT` you set in the `.env` file. Like this:
```
192.168.0.7:1883
```
Find `mqtt_signature_key` under `server` and enter the `MQTT_SIGNATURE_KEY` you set in the `.env` file.

Find `udp_gateway` under `server` and enter the `PUBLIC_IP`+`:`+`UDP_PORT` you set in the `.env` file. Like this:
```
192.168.0.7:8884
```

After completing the above configuration, you can use the curl command to verify whether your OTA address will deliver MQTT configuration. Replace `http://localhost:8002/xiaozhi/ota/` with your OTA address:
```
curl 'http://localhost:8002/xiaozhi/ota/' \
  -H 'Device-Id: 11:22:33:44:55:66' \
  --data-raw $'{\n  "application": {\n    "version": "1.0.1",\n    "elf_sha256": "1"\n  },\n  "board": {\n    "mac": "11:22:33:44:55:66"\n  }\n}'
```

If the returned content contains `mqtt` related configuration, the configuration is successful. Like this:
```
{"server_time":{"timestamp":1758781561083,"timeZone":"GMT+08:00","timezone_offset":480},"activation":{"code":"527111","message":"http://xiaozhi.server.com\n527111","challenge":"11:22:33:44:55:66"},"firmware":{"version":"1.0.1","url":"http://xiaozhi.server.com:8002/xiaozhi/otaMag/download/NOT_ACTIVATED_FIRMWARE_THIS_IS_A_INVALID_URL"},"websocket":{"url":"ws://192.168.1.15:8000/xiaozhi/v1/"},"mqtt":{"endpoint":"192.168.1.15:1883","client_id":"GID_default@@@11_22_33_44_55_66@@@11_22_33_44_55_66","username":"eyJpcCI6IjE5Mi4xNjguMS4xNSJ9","password":"fjAYs49zTJecWqJ3jBt+kqxVn/x7vkXRAc85ak/va7Y=","publish_topic":"device-server","subscribe_topic":"devices/p2p/11_22_33_44_55_66"}}
```

Since MQTT information needs to be delivered via the OTA address, as long as you can normally connect to the server's OTA address, just restart and wake up.

After waking up, check the mqtt-gateway logs to confirm whether there are successful connection logs.
```
pm2 logs xz-mqtt
```
