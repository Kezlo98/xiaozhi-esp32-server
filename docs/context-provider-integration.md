# Context Provider Tutorial

## Overview

`Context Provider` is a way to add [data sources] to the system prompt context for Xiaozhi.

`Context Provider` retrieves data from external systems at the moment Xiaozhi is awakened, and dynamically injects it into the LLM's system prompt.
This allows Xiaozhi to perceive the state of certain things in the world when awakened.

It has fundamental differences from MCP and Memory: `Context Provider` forces Xiaozhi to perceive world data; `Memory (Mem)` lets it know what was discussed previously; `MCP (function call)` is used when a specific capability/knowledge needs to be invoked.

Through this feature, at the moment Xiaozhi awakens, it can "perceive":
- Human health sensor status (body temperature, blood pressure, blood oxygen levels, etc.)
- Real-time business system data (server load, to-do items, stock information, etc.)
- Any text information obtainable via HTTP API

**Note**: This feature only allows Xiaozhi to perceive the state of things when awakened. If you want Xiaozhi to get real-time status after waking up, it's recommended to combine this feature with MCP tool calls.

## How It Works

1. **Configure Sources**: Users configure one or more HTTP API addresses.
2. **Trigger Request**: When the system builds the Prompt and finds the `{{ dynamic_context }}` placeholder in the template, it requests all configured APIs.
3. **Automatic Injection**: The system automatically formats the API response data as a Markdown list and replaces the `{{ dynamic_context }}` placeholder.

## API Specification

For Xiaozhi to correctly parse the data, your API needs to meet the following specifications:

- **Request Method**: `GET`
- **Request Headers**: The system will automatically add a `device-id` field to the Request Header.
- **Response Format**: Must return JSON format with `code` and `data` fields.

### Response Examples

**Case 1: Return Key-Value Pairs**
```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "Living Room Temperature": "26℃",
    "Living Room Humidity": "45%",
    "Front Door Status": "Closed"
  }
}
```
*Injection Result:*
```markdown
<context>
- **Living Room Temperature:** 26℃
- **Living Room Humidity:** 45%
- **Front Door Status:** Closed
</context>
```

**Case 2: Return List**
```json
{
  "code": 0,
  "data": [
    "You have 10 pending tasks",
    "Current car speed is 100km/h"
  ]
}
```
*Injection Result:*
```markdown
<context>
- You have 10 pending tasks
- Current car speed is 100km/h
</context>
```

## Configuration Guide

### Method 1: Console Configuration (Full Module Deployment)

1. Log in to the console and go to the **Role Configuration** page.
2. Find the **Context Provider** configuration item (click the "Edit Sources" button).
3. Click **Add** and enter your API address.
4. If the API requires authentication, you can add `Authorization` or other headers in the **Request Headers** section.
5. Save the configuration.

### Method 2: Configuration File (Single Module Deployment)

Edit the `xiaozhi-server/data/.config.yaml` file and add the `context_providers` configuration section:

```yaml
# Context Provider Configuration
context_providers:
  - url: "http://api.example.com/data"
    headers:
      Authorization: "Bearer your-token"
  - url: "http://another-api.com/data"
```

## Enabling the Feature

By default, the system's prompt template file (`data/.agent-base-prompt.txt`) already contains the `{{ dynamic_context }}` placeholder, so you don't need to add it manually.

**Example:**

```markdown
<context>
[Important! The following information is provided in real-time, no need to call tools to query, please use directly:]
- **Device ID:** {{device_id}}
- **Current Time:** {{current_time}}
...
{{ dynamic_context }}
</context>
```

**Note**: If you don't need this feature, you can choose to **not configure any context providers**, or **remove** the `{{ dynamic_context }}` placeholder from the prompt template file.

## Appendix: Mock Test Server Example

For your convenience in testing and development, we provide a simple Python Mock Server script. You can run this script to simulate API endpoints locally.

**mock_api_server.py**

```python
import http.server
import socketserver
import json
from urllib.parse import urlparse, parse_qs

# Set port number
PORT = 8081

class MockRequestHandler(http.server.SimpleHTTPRequestHandler):
    def do_GET(self):
        # Parse path and parameters
        parsed_path = urlparse(self.path)
        path = parsed_path.path
        query = parse_qs(parsed_path.query)

        response_data = {}
        status_code = 200

        print(f"Received request: {path}, Parameters: {query}")

        # Case 1: Simulate health data (returns Dict)
        # Path parameter style: /health
        # device_id obtained from Header
        if path == "/health":
            device_id = self.headers.get("device-id", "unknown_device")
            print(f"device_id: {device_id}")
            response_data = {
                "code": 0,
                "msg": "success",
                "data": {
                    "Test Device ID": device_id,
                    "Heart Rate": "80 bpm",
                    "Blood Pressure": "120/80 mmHg",
                    "Status": "Good"
                }
            }

        # Case 2: Simulate news list (returns List)
        # No parameters: /news/list
        elif path == "/news/list":
            response_data = {
                "code": 0,
                "msg": "success",
                "data": [
                    "Top News: Python 3.14 Released",
                    "Tech News: AI Assistants Changing Lives",
                    "Local News: Heavy Rain Tomorrow, Don't Forget Your Umbrella"
                ]
            }

        # Case 3: Simulate weather briefing (returns String)
        # No parameters: /weather/simple
        elif path == "/weather/simple":
            response_data = {
                "code": 0,
                "msg": "success",
                "data": "Today sunny turning cloudy, temperature 20-25 degrees, excellent air quality, suitable for going out."
            }

        # Case 4: Simulate device details (Query parameter style)
        # Parameter style: /device/info
        # device_id obtained from Header
        elif path == "/device/info":
            device_id = self.headers.get("device-id", "unknown_device")
            response_data = {
                "code": 0,
                "msg": "success",
                "data": {
                    "Query Method": "Header Parameter",
                    "Device ID": device_id,
                    "Battery": "85%",
                    "Firmware": "v2.0.1"
                }
            }

        # Case 5: 404 Not Found
        else:
            status_code = 404
            response_data = {"error": "Endpoint does not exist"}

        # Send response
        self.send_response(status_code)
        self.send_header('Content-type', 'application/json; charset=utf-8')
        self.end_headers()
        self.wfile.write(json.dumps(response_data, ensure_ascii=False).encode('utf-8'))

# Start server
# Allow address reuse to prevent errors on quick restart
socketserver.TCPServer.allow_reuse_address = True
with socketserver.TCPServer(("", PORT), MockRequestHandler) as httpd:
    print(f"==================================================")
    print(f"Mock API Server Started: http://localhost:{PORT}")
    print(f"Available Endpoints:")
    print(f"1. [Dict] http://localhost:{PORT}/health")
    print(f"2. [List] http://localhost:{PORT}/news/list")
    print(f"3. [Text] http://localhost:{PORT}/weather/simple")
    print(f"4. [Param] http://localhost:{PORT}/device/info")
    print(f"==================================================")
    try:
        httpd.serve_forever()
    except KeyboardInterrupt:
        print("\nServer stopped")
```
