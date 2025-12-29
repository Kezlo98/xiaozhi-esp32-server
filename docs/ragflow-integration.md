# RAGFlow Integration Guide

This tutorial consists of two parts:

- Part 1: How to deploy RAGFlow
- Part 2: How to configure the RAGFlow API in the Control Console

If you are familiar with RAGFlow and have already deployed it, you can skip the first part and go directly to the second part. However, if you want guidance on deploying RAGFlow so that it can share `mysql` and `redis` basic services with `xiaozhi-esp32-server` to reduce resource costs, you need to start from the first part.

# Part 1: How to Deploy RAGFlow
## Step 1: Verify MySQL and Redis availability

RAGFlow depends on the `mysql` database. If you have already deployed the `Control Console`, you have already installed `mysql`. You can share it.

You can try using the `telnet` command on the host machine to see if you can access the `mysql` `3306` port normally.
``` shell
telnet 127.0.0.1 3306

telnet 127.0.0.1 6379
```
If you can access ports `3306` and `6379`, please ignore the following content and proceed directly to Step 2.

If you cannot access them, you need to recall how your `mysql` was installed.

If your MySQL was installed using an installation package you set up yourself, it means your `mysql` has network isolation. You may need to resolve the issue of accessing the `mysql` `3306` port first.

If your `mysql` was installed through this project's `docker-compose_all.yml`, you need to find the `docker-compose_all.yml` file you used to create the database and modify the following content

Before modification
``` yaml
  xiaozhi-esp32-server-db:
    ...
    networks:
      - default
    expose:
      - "3306:3306"
  xiaozhi-esp32-server-redis:
    ...
    expose:
      - 6379
```

After modification
``` yaml
  xiaozhi-esp32-server-db:
    ...
    networks:
      - default
    ports:
      - "3306:3306"
  xiaozhi-esp32-server-redis:
    ...
    ports:
      - "6379:6379"
```

Note that you need to change `expose` to `ports` under both `xiaozhi-esp32-server-db` and `xiaozhi-esp32-server-redis`. After modification, you need to restart. Here is the command to restart MySQL:

``` shell
# Navigate to the folder where docker-compose_all.yml is located, for example mine is xiaozhi-server
cd xiaozhi-server
docker compose -f docker-compose_all.yml down
docker compose -f docker-compose.yml up -d
```

After starting, use the `telnet` command on the host machine again to see if you can access the `mysql` `3306` port normally.
``` shell
telnet 127.0.0.1 3306

telnet 127.0.0.1 6379
```
Normally, this should allow access.

## Step 2: Create database and tables
If your host machine can access the MySQL database normally, create a database named `rag_flow` and a user `rag_flow` with password `infini_rag_flow` on MySQL.

``` sql
-- Create database
CREATE DATABASE IF NOT EXISTS rag_flow CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Create user and grant privileges
CREATE USER IF NOT EXISTS 'rag_flow'@'%' IDENTIFIED BY 'infini_rag_flow';
GRANT ALL PRIVILEGES ON rag_flow.* TO 'rag_flow'@'%';

-- Flush privileges
FLUSH PRIVILEGES;
```

## Step 3: Download the RAGFlow project

You need to find a folder on your computer to store the RAGFlow project. For example, I use the `/home/system/xiaozhi` folder.

You can use the `git` command to download the RAGFlow project to this folder. This tutorial uses version `v0.22.0` for installation and deployment.
```
git clone https://ghfast.top/https://github.com/infiniflow/ragflow.git
cd ragflow
git checkout v0.22.0
```
After downloading, enter the `docker` folder.
``` shell
cd docker
```
Modify the `docker-compose.yml` file in the `ragflow/docker` folder, remove the `depends_on` configuration from both `ragflow-cpu` and `ragflow-gpu` services to decouple the `ragflow-cpu` service's dependency on `mysql`.

Before modification:
``` yaml
  ragflow-cpu:
    depends_on:
      mysql:
        condition: service_healthy
    profiles:
      - cpu
  ...
  ragflow-gpu:
    depends_on:
      mysql:
        condition: service_healthy
    profiles:
      - gpu
```
After modification:
``` yaml
  ragflow-cpu:
    profiles:
      - cpu
  ...
  ragflow-gpu:
    profiles:
      - gpu
```

Next, modify the `docker-compose-base.yml` file in the `ragflow/docker` folder to remove the `mysql` and `redis` configurations.

For example, before deletion:
``` yaml
services:
  minio:
    image: quay.io/minio/minio:RELEASE.2025-06-13T11-33-47Z
    ...
  mysql:
    image: mysql:8.0
    ...
  redis:
    image: redis:6.2-alpine
    ...
```

After deletion
``` yaml
services:
  minio:
    image: quay.io/minio/minio:RELEASE.2025-06-13T11-33-47Z
    ...
```
## Step 4: Modify environment variable configuration

Edit the `.env` file in the `ragflow/docker` folder, find the following configurations, search and modify them one by one! Search and modify them one by one!

For the modifications to the `.env` file below, 60% of people will overlook the `MYSQL_USER` configuration causing RAGFlow startup failure, therefore, it needs to be emphasized three times:

Emphasis #1: If your `.env` file does not have a `MYSQL_USER` configuration, please add this item to the configuration file!

Emphasis #2: If your `.env` file does not have a `MYSQL_USER` configuration, please add this item to the configuration file!

Emphasis #3: If your `.env` file does not have a `MYSQL_USER` configuration, please add this item to the configuration file!

``` env
# Port settings
SVR_WEB_HTTP_PORT=8008           # HTTP port
SVR_WEB_HTTPS_PORT=8009          # HTTPS port
# MySQL configuration - modify to your local MySQL information
MYSQL_HOST=host.docker.internal  # Use host.docker.internal to let container access host services
MYSQL_PORT=3306                  # Local MySQL port
MYSQL_USER=rag_flow              # The username created above, add this item if it doesn't exist
MYSQL_PASSWORD=infini_rag_flow   # The password set above
MYSQL_DBNAME=rag_flow            # Database name

# Redis configuration - modify to your local Redis information
REDIS_HOST=host.docker.internal  # Use host.docker.internal to let container access host services
REDIS_PORT=6379                  # Local Redis port
REDIS_PASSWORD=                  # If your Redis has no password, fill it like this, otherwise fill in the password
```

Note, if your Redis has no password, you also need to modify `service_conf.yaml.template` in the `ragflow/docker` folder, replacing `infini_rag_flow` with an empty string.

Before modification
``` shell
redis:
  db: 1
  password: '${REDIS_PASSWORD:-infini_rag_flow}'
  host: '${REDIS_HOST:-redis}:6379'
```
After modification
``` shell
redis:
  db: 1
  password: '${REDIS_PASSWORD:-}'
  host: '${REDIS_HOST:-redis}:6379'
```

## Step 5: Start the RAGFlow service
Execute command:
``` shell
docker-compose -f docker-compose.yml up -d
```
After successful execution, you can use the `docker logs -n 20 -f docker-ragflow-cpu-1` command to view the logs of the `docker-ragflow-cpu-1` service.

If there are no errors in the logs, it means the RAGFlow service started successfully.

# Step 5: Register an account
You can access `http://127.0.0.1:8008` in your browser, click `Sign Up` to register an account.

After successful registration, you can click `Sign In` to log in to the RAGFlow service. If you want to disable the registration service of RAGFlow and prevent others from registering accounts, you can set the `REGISTER_ENABLED` configuration item to `0` in the `.env` file in the `ragflow/docker` folder.

``` dotenv
REGISTER_ENABLED=0
```
After modification, restart the RAGFlow service.
``` shell
docker-compose -f docker-compose.yml down
docker-compose -f docker-compose.yml up -d
```

# Step 6: Configure the RAGFlow service models
You can access `http://127.0.0.1:8008` in your browser, click `Sign In` to log in to the RAGFlow service. Click the `Avatar` in the upper right corner of the page to enter the settings page.
First, in the left navigation bar, click `Model Providers` to enter the model configuration page. Under the `Available Models` search box on the right, select `LLM`, select your model provider from the list, click `Add`, and enter your key;
Then, select `TEXT EMBEDDING`, select your model provider from the list, click `Add`, and enter your key.
Finally, refresh the page, click LLM and Embedding in the `Set Default Model` list respectively, and select the model you use. Please confirm that your key has the corresponding service enabled, for example, if the Embedding model I use is from xxx provider, I need to go to this provider's official website to check if this model requires purchasing a resource package to use.


# Part 2: Configure RAGFlow Service

# Step 1: Log in to RAGFlow service
You can access `http://127.0.0.1:8008` in your browser, click `Sign In` to log in to the RAGFlow service.

Then click the `Avatar` in the upper right corner to enter the settings page. In the left navigation bar, click the `API` function, then click the "API Key" button. A popup appears.

In the popup, click the "Create new Key" button to generate an API Key. Copy this `API Key`, you will use it later.

# Step 2: Configure in Control Console
Make sure your Control Console version is `0.8.7` or above. Log in to the Control Console with the super administrator account.

First, you need to enable the Knowledge Base feature. In the top navigation bar, click `Parameter Dictionary`, in the dropdown menu, click the `System Function Configuration` page. Check `Knowledge Base` on the page, click `Save Configuration`. You will then see the `Knowledge Base` feature in the navigation bar.

In the top navigation bar, click `Model Configuration`, in the left navigation bar, click `Knowledge Base`. Find `RAG_RAGFlow` in the list, click the `Edit` button.

In `Service Address`, fill in `http://your_ragflow_service_LAN_IP:8008`, for example, if my RAGFlow service LAN IP is `192.168.1.100`, then I would fill in `http://192.168.1.100:8008`.

In `API Key`, fill in the `API Key` you copied earlier.

Finally, click the save button.

# Step 2: Create a Knowledge Base
Log in to the Control Console with the super administrator account. In the top navigation bar, click `Knowledge Base`, in the bottom left corner of the list, click the `Add` button. Fill in a name and description for the knowledge base. Click save.

To improve the LLM's understanding and recall capability for the knowledge base, it is recommended to fill in a meaningful name and description when creating a knowledge base. For example, if you want to create a knowledge base about `Company Introduction`, the knowledge base name could be `Company Introduction`, and the description could be `Information about the company such as basic company info, services, contact phone, address, etc.`.

After saving, you can see this knowledge base in the knowledge base list. Click the `View` button of the knowledge base you just created to enter the knowledge base details page.

On the knowledge base details page, click the `Add` button in the bottom left corner to upload documents to the knowledge base.

After uploading, you can see the uploaded documents on the knowledge base details page. At this point, you can click the `Parse` button on a document to parse the document.

After parsing is complete, you can view the parsed chunk information. On the knowledge base details page, click the `Recall Test` button to test the knowledge base's recall/retrieval functionality.

# Step 3: Let Xiaozhi use the RAGFlow knowledge base
Log in to the Control Console. In the top navigation bar, click `Agents`, find the agent you want to configure, and click the `Configure Role` button.

On the left side of Intent Recognition, click the `Edit Functions` button, a popup will appear. In the popup, select the knowledge base you want to add. Save.
