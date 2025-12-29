# Full Module Source Code Deployment Auto-Upgrade Method

This tutorial is designed for enthusiasts who deploy full module source code, explaining how to automatically pull source code, compile, and start port services using automated commands for the most efficient system upgrades.

The project's test platform `https://2662r3426b.vicp.fun` has been using this method since it opened, with good results.

You can refer to the video tutorial by Bilibili creator `Bile Labs`: [《Xiaozhi Server xiaozhi-server Auto-Update and Latest Version MCP Configuration Step-by-Step Tutorial》](https://www.bilibili.com/video/BV15H37zHE7Q)

# Prerequisites
- Your computer/server runs a Linux operating system
- You have already run through the entire process successfully
- You like to keep up with the latest features, but find manual deployment tedious and want an automatic update method

The second condition must be met because some files, JDK, Node.js environment, Conda environment, etc. mentioned in this tutorial are only available after you've run through the entire process. If you haven't, you may not understand what I'm referring to when I mention certain files.

# Tutorial Results
- Solve the problem of not being able to pull the latest project source code in China
- Automatically pull code and compile frontend files
- Automatically pull code and compile Java files, automatically kill port 8002, automatically start port 8002
- Automatically pull Python code, automatically kill port 8000, automatically start port 8000

# Step 1: Choose Your Project Directory

For example, I planned my project directory as follows. This is a newly created empty directory. If you don't want any errors, you can follow the same setup:
```
/home/system/xiaozhi
```

# Step 2: Clone This Project
First, execute the following command to pull the source code. This command works for servers and computers in China without needing a VPN:

```
cd /home/system/xiaozhi
git clone https://ghproxy.net/https://github.com/xinnan-tech/xiaozhi-esp32-server.git
```

After execution, your project directory will have a new folder `xiaozhi-esp32-server`, which is the project source code.

# Step 3: Copy Basic Files

If you've already run through the entire process, you should be familiar with the funasr model file `xiaozhi-server/models/SenseVoiceSmall/model.pt` and your private configuration file `xiaozhi-server/data/.config.yaml`.

Now you need to copy the `model.pt` file to the new directory. You can do it like this:
```
# Create required directories
mkdir -p /home/system/xiaozhi/xiaozhi-esp32-server/main/xiaozhi-server/data/

cp your-original-.config.yaml-full-path /home/system/xiaozhi/xiaozhi-esp32-server/main/xiaozhi-server/data/.config.yaml
cp your-original-model.pt-full-path /home/system/xiaozhi/xiaozhi-esp32-server/main/xiaozhi-server/models/SenseVoiceSmall/model.pt
```

# Step 4: Create Three Auto-Compile Files

## 4.1 Auto-compile manager-web Module
In the `/home/system/xiaozhi/` directory, create a file named `update_8001.sh` with the following content:

```
cd /home/system/xiaozhi/xiaozhi-esp32-server
git fetch --all
git reset --hard
git pull origin main


cd /home/system/xiaozhi/xiaozhi-esp32-server/main/manager-web
npm install
npm run build
rm -rf /home/system/xiaozhi/manager-web
mv /home/system/xiaozhi/xiaozhi-esp32-server/main/manager-web/dist /home/system/xiaozhi/manager-web
```

After saving, execute the permission command:
```
chmod 777 update_8001.sh
```
After execution, continue below.

## 4.2 Auto-compile and Run manager-api Module
In the `/home/system/xiaozhi/` directory, create a file named `update_8002.sh` with the following content:

```
cd /home/system/xiaozhi/xiaozhi-esp32-server
git pull origin main


cd /home/system/xiaozhi/xiaozhi-esp32-server/main/manager-api
rm -rf target
mvn clean package -Dmaven.test.skip=true
cd /home/system/xiaozhi/

# Find the process ID occupying port 8002
PID=$(sudo netstat -tulnp | grep 8002 | awk '{print $7}' | cut -d'/' -f1)

rm -rf /home/system/xiaozhi/xiaozhi-esp32-api.jar
mv /home/system/xiaozhi/xiaozhi-esp32-server/main/manager-api/target/xiaozhi-esp32-api.jar /home/system/xiaozhi/xiaozhi-esp32-api.jar

# Check if process ID was found
if [ -z "$PID" ]; then
  echo "No process found occupying port 8002"
else
  echo "Found process occupying port 8002, process ID: $PID"
  # Kill the process
  kill -9 $PID
  kill -9 $PID
  echo "Killed process $PID"
fi

nohup java -jar xiaozhi-esp32-api.jar --spring.profiles.active=dev &

tail tail -f nohup.out
```

After saving, execute the permission command:
```
chmod 777 update_8002.sh
```
After execution, continue below.

## 4.3 Auto-compile and Run Python Project
In the `/home/system/xiaozhi/` directory, create a file named `update_8000.sh` with the following content:

```
cd /home/system/xiaozhi/xiaozhi-esp32-server
git pull origin main

# Find the process ID occupying port 8000
PID=$(sudo netstat -tulnp | grep 8000 | awk '{print $7}' | cut -d'/' -f1)

# Check if process ID was found
if [ -z "$PID" ]; then
  echo "No process found occupying port 8000"
else
  echo "Found process occupying port 8000, process ID: $PID"
  # Kill the process
  kill -9 $PID
  kill -9 $PID
  echo "Killed process $PID"
fi
cd main/xiaozhi-server
# Initialize conda environment
source ~/.bashrc
conda activate xiaozhi-esp32-server
pip install -r requirements.txt
nohup python app.py >/dev/null &
tail -f /home/system/xiaozhi/xiaozhi-esp32-server/main/xiaozhi-server/tmp/server.log
```

After saving, execute the permission command:
```
chmod 777 update_8000.sh
```
After execution, continue below.

# Daily Updates

After setting up all the above scripts, for daily updates, you just need to execute the following commands in sequence for automatic updates and startup:

```
cd /home/system/xiaozhi
# Update and start Java program
./update_8001.sh
# Update web program
./update_8002.sh
# Update and start Python program
./update_8000.sh


# To view Java logs later, execute the following command
tail -f nohup.out
# To view Python logs later, execute the following command
tail -f /home/system/xiaozhi/xiaozhi-esp32-server/main/xiaozhi-server/tmp/server.log
```

# Notes
The test platform `https://2662r3426b.vicp.fun` uses nginx as a reverse proxy. For detailed nginx.conf configuration, [refer to here](https://github.com/xinnan-tech/xiaozhi-esp32-server/issues/791)

## FAQ

### 1. Why don't I see port 8001?
Answer: Port 8001 is used in the development environment for running the frontend. If you're deploying on a server, it's not recommended to use `npm run serve` to start the frontend on port 8001. Instead, compile it into HTML files like this tutorial shows, and then use nginx to manage access.

### 2. Do I need to manually update SQL statements each time I update?
Answer: No, because the project uses **Liquibase** to manage database versions, which automatically executes new SQL scripts.
