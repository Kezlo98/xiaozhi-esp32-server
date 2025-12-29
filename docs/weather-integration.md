# Weather Plugin User Guide

## Overview

The weather plugin `get_weather` is one of the core features of the Xiaozhi ESP32 voice assistant, supporting voice queries for weather information across China. The plugin is based on the QWeather API, providing real-time weather and 7-day weather forecast functionality.

## API Key Application Guide

### 1. Register a QWeather Account

1. Visit [QWeather Console](https://console.qweather.com/)
2. Register an account and complete email verification
3. Log in to the console

### 2. Create an Application to Get API Key

1. After entering the console, click ["Project Management"](https://console.qweather.com/project?lang=zh) on the right -> "Create Project"
2. Fill in the project information:
   - **Project Name**: e.g., "Xiaozhi Voice Assistant"
3. Click Save
4. After the project is created, click "Create Credential" in the project
5. Fill in the credential information:
    - **Credential Name**: e.g., "Xiaozhi Voice Assistant"
    - **Authentication Method**: Select "API Key"
6. Click Save
7. Copy the `API Key` from the credential, this is the first key configuration information

### 3. Get API Host

1. In the console, click ["Settings"](https://console.qweather.com/setting?lang=zh) -> "API Host"
2. View the exclusive `API Host` address assigned to you, this is the second key configuration information

The above operations will yield two important configuration pieces: `API Key` and `API Host`

## Configuration Methods (Choose One)

### Method 1: If you deployed with Control Console (Recommended)

1. Log in to the Control Console
2. Go to the "Role Configuration" page
3. Select the agent to configure
4. Click the "Edit Functions" button
5. Find the "Weather Query" plugin in the parameter configuration area on the right
6. Check "Weather Query"
7. Paste the first key configuration `API Key` into `Weather Plugin API Key`
8. Paste the second key configuration `API Host` into `Developer API Host`
9. Save the configuration, then save the agent configuration

### Method 2: If you only deployed the single module xiaozhi-server

In `data/.config.yaml` configure:

1. Paste the first key configuration `API Key` into `api_key`
2. Paste the second key configuration `API Host` into `api_host`
3. Fill in your city into `default_location`, e.g., `Guangzhou`

```yaml
plugins:
  get_weather:
    api_key: "your_qweather_api_key"
    api_host: "your_qweather_api_host"
    default_location: "your_default_query_city"
```

