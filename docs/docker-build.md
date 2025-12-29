# Building Docker Images Locally

This project now uses GitHub Actions for automatic Docker image building. This document is for those who need to build Docker images locally.

1. Install Docker
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
2. Build Docker Images
```
# Navigate to the project root directory
# Build server
docker build -t xiaozhi-esp32-server:server_latest -f ./Dockerfile-server .
# Build web
docker build -t xiaozhi-esp32-server:web_latest -f ./Dockerfile-web .

# After building, you can use docker-compose to start the project
# You need to modify docker-compose.yml to use your own built image versions
cd main/xiaozhi-server
docker compose up -d
```
