# IndexStreamTTS Usage Guide

## Environment Setup
### 1. Clone the Project
```bash
git clone https://github.com/Ksuriuri/index-tts-vllm.git
```
Enter the extracted directory
```bash
cd index-tts-vllm
```
Switch to the specified version (using the historical version of VLLM-0.10.2)
```bash
git checkout 224e8d5e5c8f66801845c66b30fa765328fd0be3
```

### 2. Create and Activate Conda Environment
```bash
conda create -n index-tts-vllm python=3.12
conda activate index-tts-vllm
```

### 3. Install PyTorch (Version 2.8.0 Required - Latest Version)
#### Check the maximum supported version and actually installed version of GPU driver
```bash
nvidia-smi
nvcc --version
```
#### Maximum CUDA version supported by driver
```bash
CUDA Version: 12.8
```
#### Actually installed CUDA compiler version
```bash
Cuda compilation tools, release 12.8, V12.8.89
```
#### Corresponding installation command (pytorch defaults to 12.8 driver version)
```bash
pip install torch torchvision
```
Requires pytorch version 2.8.0 (corresponding to vllm 0.10.2). For specific installation commands, please refer to: [PyTorch Official Website](https://pytorch.org/get-started/locally/)

### 4. Install Dependencies
```bash
pip install -r requirements.txt
```

### 5. Download Model Weights
### Option 1: Download Official Weights and Convert
These are official weight files. Download them to any local path. Supports IndexTTS-1.5 weights.
| HuggingFace                                                   | ModelScope                                                          |
|---------------------------------------------------------------|---------------------------------------------------------------------|
| [IndexTTS](https://huggingface.co/IndexTeam/Index-TTS)        | [IndexTTS](https://modelscope.cn/models/IndexTeam/Index-TTS)        |
| [IndexTTS-1.5](https://huggingface.co/IndexTeam/IndexTTS-1.5) | [IndexTTS-1.5](https://modelscope.cn/models/IndexTeam/IndexTTS-1.5) |

Below uses ModelScope installation method as an example.
#### Note: git needs to have lfs installed and initialized (skip if already installed)
```bash
sudo apt-get install git-lfs
git lfs install
```
Create model directory and clone the model
```bash
mkdir model_dir
cd model_dir
git clone https://www.modelscope.cn/IndexTeam/IndexTTS-1.5.git
```

#### Model Weight Conversion
```bash
bash convert_hf_format.sh /path/to/your/model_dir
```
For example: if your downloaded IndexTTS-1.5 model is stored in the model_dir directory, execute the following command:
```bash
bash convert_hf_format.sh model_dir/IndexTTS-1.5
```
This operation will convert the official model weights to a version compatible with the transformers library, saved in the vllm folder under the model weight path, for subsequent vllm library model loading.

### 6. Modify API Interface for Project Compatibility
The API return data is not compatible with the project and needs adjustment to return audio data directly
```bash
vi api_server.py
```
```bash
@app.post("/tts", responses={
    200: {"content": {"application/octet-stream": {}}},
    500: {"content": {"application/json": {}}}
})
async def tts_api(request: Request):
    try:
        data = await request.json()
        text = data["text"]
        character = data["character"]

        global tts
        sr, wav = await tts.infer_with_ref_audio_embed(character, text)

        return Response(content=wav.tobytes(), media_type="application/octet-stream")

    except Exception as ex:
        tb_str = ''.join(traceback.format_exception(type(ex), ex, ex.__traceback__))
        print(tb_str)
        return JSONResponse(
            status_code=500,
            content={
                "status": "error",
                "error": str(tb_str)
            }
        )
```

### 7. Write Shell Startup Script (Note: Run in the Corresponding Conda Environment)
```bash
vi start_api.sh
```
### Paste the following content and save with :wq
#### The path /home/system/index-tts-vllm/model_dir/IndexTTS-1.5 in the script should be modified to your actual path
```bash
# Activate conda environment
conda activate index-tts-vllm
echo "Activating project conda environment"
sleep 2
# Find the process ID occupying port 11996
PID_VLLM=$(sudo netstat -tulnp | grep 11996 | awk '{print $7}' | cut -d'/' -f1)

# Check if process ID was found
if [ -z "$PID_VLLM" ]; then
  echo "No process found occupying port 11996"
else
  echo "Found process occupying port 11996, process ID: $PID_VLLM"
  # First try normal kill, wait 2 seconds
  kill $PID_VLLM
  sleep 2
  # Check if process is still running
  if ps -p $PID_VLLM > /dev/null; then
    echo "Process still running, force terminating..."
    kill -9 $PID_VLLM
  fi
  echo "Terminated process $PID_VLLM"
fi

# Find VLLM::EngineCore related processes
GPU_PIDS=$(ps aux | grep -E "VLLM|EngineCore" | grep -v grep | awk '{print $2}')

# Check if process IDs were found
if [ -z "$GPU_PIDS" ]; then
  echo "No VLLM related processes found"
else
  echo "Found VLLM related processes, process IDs: $GPU_PIDS"
  # First try normal kill, wait 2 seconds
  kill $GPU_PIDS
  sleep 2
  # Check if process is still running
  if ps -p $GPU_PIDS > /dev/null; then
    echo "Process still running, force terminating..."
    kill -9 $GPU_PIDS
  fi
  echo "Terminated processes $GPU_PIDS"
fi

# Create tmp directory (if not exists)
mkdir -p tmp

# Run api_server.py in background, redirect logs to tmp/server.log
nohup python api_server.py --model_dir /home/system/index-tts-vllm/model_dir/IndexTTS-1.5 --port 11996 > tmp/server.log 2>&1 &
echo "api_server.py is running in background, check logs at tmp/server.log"
```
Grant script execution permission and run the script
```bash
chmod +x start_api.sh
./start_api.sh
```
Logs will be output to tmp/server.log. You can view the logs with the following command:
```bash
tail -f tmp/server.log
```
If you have enough GPU memory, you can add the startup parameter --gpu_memory_utilization in the script to adjust GPU memory usage ratio. The default value is 0.25.

## Voice Configuration
index-tts-vllm supports registering custom voices through configuration files, supporting both single voice and mixed voice configurations.
Configure custom voices in the assets/speaker.json file in the project root directory.
### Configuration Format Description
```bash
{
    "Speaker Name 1": [
        "audio_file_path_1.wav",
        "audio_file_path_2.wav"
    ],
    "Speaker Name 2": [
        "audio_file_path_3.wav"
    ]
}
```
### Note (Restart the service after configuring roles for voice registration)
After adding, you need to add the corresponding speaker in the console (for single module, replace the corresponding voice)
