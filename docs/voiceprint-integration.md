# Voiceprint Recognition Setup Guide

This tutorial contains 3 parts:
- 1. How to deploy the voiceprint recognition service
- 2. How to configure voiceprint recognition in full module deployment
- 3. How to configure voiceprint recognition in minimal deployment

# 1. How to Deploy the Voiceprint Recognition Service

## Step 1: Download the voiceprint recognition project source code

Open [Voiceprint Recognition Project Address](https://github.com/xinnan-tech/voiceprint-api) in your browser

After opening, find a green button that says `Code`, click it, and you will see the `Download ZIP` button.

Click it to download the project source code zip file. After downloading to your computer, extract it. At this point, it may be named `voiceprint-api-main`
You need to rename it to `voiceprint-api`.

## Step 2: Create database and tables

Voiceprint recognition depends on the `mysql` database. If you have already deployed the `Control Console`, you have already installed `mysql`. You can share it.

You can try using the `telnet` command on the host machine to see if you can access the `mysql` `3306` port normally.
```
telnet 127.0.0.1 3306
```
If you can access port 3306, please ignore the following content and proceed directly to Step 3.

If you cannot access it, you need to recall how your `mysql` was installed.

If your MySQL was installed using an installation package you set up yourself, it means your `mysql` has network isolation. You may need to resolve the issue of accessing the `mysql` `3306` port first.

If your `mysql` was installed through this project's `docker-compose_all.yml`, you need to find the `docker-compose_all.yml` file you used to create the database and modify the following content

Before modification
```
  xiaozhi-esp32-server-db:
    ...
    networks:
      - default
    expose:
      - "3306:3306"
```

After modification
```
  xiaozhi-esp32-server-db:
    ...
    networks:
      - default
    ports:
      - "3306:3306"
```

Note that you need to change `expose` to `ports` under `xiaozhi-esp32-server-db`. After modification, you need to restart. Here is the command to restart MySQL:

```
# Navigate to the folder where docker-compose_all.yml is located, for example mine is xiaozhi-server
cd xiaozhi-server
docker compose -f docker-compose_all.yml down
docker compose -f docker-compose.yml up -d
```

After starting, use the `telnet` command on the host machine again to see if you can access the `mysql` `3306` port normally.
```
telnet 127.0.0.1 3306
```
Normally, this should allow access.

## Step 3: Create database and tables
If your host machine can access the MySQL database normally, create a database named `voiceprint_db` and a `voiceprints` table on MySQL.

```
CREATE DATABASE voiceprint_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE voiceprint_db;

CREATE TABLE voiceprints (
    id INT AUTO_INCREMENT PRIMARY KEY,
    speaker_id VARCHAR(255) NOT NULL UNIQUE,
    feature_vector LONGBLOB NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_speaker_id (speaker_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## Step 4: Configure database connection

Enter the `voiceprint-api` folder and create a folder named `data`.

Copy `voiceprint.yaml` from the `voiceprint-api` root directory to the `data` folder and rename it to `.voiceprint.yaml`

Next, you need to focus on configuring the database connection in `.voiceprint.yaml`.

```
mysql:
  host: "127.0.0.1"
  port: 3306
  user: "root"
  password: "your_password"
  database: "voiceprint_db"
```

Note! Since your voiceprint recognition service is deployed using Docker, `host` needs to be filled with your `LAN IP of the machine where MySQL is located`.

Note! Since your voiceprint recognition service is deployed using Docker, `host` needs to be filled with your `LAN IP of the machine where MySQL is located`.

Note! Since your voiceprint recognition service is deployed using Docker, `host` needs to be filled with your `LAN IP of the machine where MySQL is located`.

## Step 5: Start the program
This is a very simple project, it is recommended to run it with Docker. However, if you don't want to use Docker, you can refer to [this page](https://github.com/xinnan-tech/voiceprint-api/blob/main/README.md) to run from source code. Here is the Docker method:

```
# Enter the project source code root directory
cd voiceprint-api

# Clear cache
docker compose -f docker-compose.yml down
docker stop voiceprint-api
docker rm voiceprint-api
docker rmi ghcr.nju.edu.cn/xinnan-tech/voiceprint-api:latest

# Start Docker container
docker compose -f docker-compose.yml up -d
# View logs
docker logs -f voiceprint-api
```

At this point, the logs will output something like:
```
250711 INFO-🚀 Starting: Production environment service startup (Uvicorn), listening address: 0.0.0.0:8005
250711 INFO-============================================================
250711 INFO-Voiceprint API address: http://127.0.0.1:8005/voiceprint/health?key=abcd
250711 INFO-============================================================
```

Please copy the voiceprint API address:

Since you are deploying with Docker, do NOT use the above address directly!

Since you are deploying with Docker, do NOT use the above address directly!

Since you are deploying with Docker, do NOT use the above address directly!

First copy the address and save it in a draft. You need to know what your computer's LAN IP is. For example, if my computer's LAN IP is `192.168.1.25`, then
my original API address
```
http://127.0.0.1:8005/voiceprint/health?key=abcd

```
needs to be changed to
```
http://192.168.1.25:8005/voiceprint/health?key=abcd
```

After modification, please directly access the `voiceprint API address` in your browser. When the browser shows code like this, it means success.
```
{"total_voiceprints":0,"status":"healthy"}
```

Please keep the modified `voiceprint API address`, you will need it in the next step.

# 2. How to Configure Voiceprint Recognition in Full Module Deployment

## Step 1: Configure the API
First, you need to enable the voiceprint recognition feature. In the Control Console, click `Parameter Dictionary` at the top, and in the dropdown menu, click the `System Function Configuration` page. Check `Voiceprint Recognition` on the page, click `Save Configuration`. You will then see the `Voiceprint Recognition` button on the agent card when creating a new agent.

If you are using full module deployment, log in to the Control Console with an administrator account, click `Parameter Dictionary` at the top, and select the `Parameter Management` function.

Then search for the parameter `server.voice_print`, at this point its value should be `null`.
Click the modify button, paste the `voiceprint API address` from the previous step into the `Parameter Value`. Then save.

If it saves successfully, everything is working. You can go to the agent to check the effect. If it fails, it means the Control Console cannot access the voiceprint recognition service, most likely due to network firewall or not filling in the correct LAN IP.

## Step 2: Set agent memory mode

Go to your agent's role configuration and set the memory to `Local Short-term Memory`, make sure to enable `Report Text + Voice`.

## Step 3: Chat with your agent

Power on your device and chat with it using normal speech speed and tone.

## Step 4: Set voiceprint

In the Control Console, on the `Agent Management` page, there is a `Voiceprint Recognition` button on the agent panel. Click it. There is an `Add button` at the bottom. You can register a voiceprint for someone's speech.
In the popup, it is recommended to fill in the `Description` attribute, which can be the person's occupation, personality, hobbies. This helps the agent analyze and understand the speaker.

## Step 3: Chat with your agent

Power on your device and ask it: Do you know who I am? If it can answer correctly, it means the voiceprint recognition feature is working properly.

# 3. How to Configure Voiceprint Recognition in Minimal Deployment

## Step 1: Configure the API
Open the `xiaozhi-server/data/.config.yaml` file (create it if it doesn't exist), then add/modify the following content:

```
# Voiceprint recognition configuration
voiceprint:
  # Voiceprint API address
  url: your_voiceprint_api_address
  # Speaker configuration: speaker_id, name, description
  speakers:
    - "test1,Zhang San,Zhang San is a programmer"
    - "test2,Li Si,Li Si is a product manager"
    - "test3,Wang Wu,Wang Wu is a designer"
```

Paste the `voiceprint API address` from the previous step into `url`. Then save.

Add the `speakers` parameter according to your needs. Note the `speaker_id` parameter here, it will be used for voiceprint registration later.

## Step 2: Register voiceprint
If you have already started the voiceprint service, access `http://localhost:8005/voiceprint/docs` in your local browser to view the API documentation. Here we only explain how to use the voiceprint registration API.

The voiceprint registration API address is `http://localhost:8005/voiceprint/register`, the request method is POST.

The request header needs to include Bearer Token authentication, the token is the part after `?key=` in the `voiceprint API address`. For example, if my voiceprint registration address is `http://127.0.0.1:8005/voiceprint/health?key=abcd`, then my token is `abcd`.

The request body contains the speaker ID (speaker_id) and WAV audio file (file). Request example:

```
curl -X POST \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -F "speaker_id=your_speaker_id_here" \
  -F "file=@/path/to/your/file" \
  http://localhost:8005/voiceprint/register
```

Here `file` is the audio file of the speaker's speech to be registered, and `speaker_id` needs to match the `speaker_id` from Step 1's configuration. For example, if I need to register Zhang San's voiceprint, and in `.config.yaml` Zhang San's `speaker_id` is `test1`, then when I register Zhang San's voiceprint, the `speaker_id` in the request body should be `test1`, and `file` should be an audio file of Zhang San speaking.

 ## Step 3: Start the service

Start the Xiaozhi server and voiceprint service, and you can use it normally.
