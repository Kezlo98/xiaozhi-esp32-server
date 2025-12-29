# Frequently Asked Questions

### 1. Why does Xiaozhi recognize my speech as a lot of Korean, Japanese, or English?

Suggestion: Check whether the `models/SenseVoiceSmall` directory already has the `model.pt` file. If not, you need to download it. See here: [Download Speech Recognition Model Files](Deployment.md#model-files)

### 2. Why does "TTS task error, file does not exist" appear?

Suggestion: Check whether you have correctly installed the `libopus` and `ffmpeg` libraries using `conda`.

If not installed, install them:

```
conda install conda-forge::libopus
conda install conda-forge::ffmpeg
```

### 3. TTS frequently fails or times out

Suggestion: If `EdgeTTS` frequently fails, first check whether you are using a proxy (VPN). If you are, try disabling the proxy and try again;
If you are using Volcano Engine's Doubao TTS, and it frequently fails, it is recommended to use the paid version because the test version only supports 2 concurrent connections.

### 4. Can connect to self-hosted server via WiFi, but cannot connect in 4G mode

Reason: Brother Xia's firmware requires a secure connection for 4G mode.

Solution: Currently there are two ways to solve this. Choose one:

1. Modify the code. Refer to this video for the solution: https://www.bilibili.com/video/BV18MfTYoE85

2. Use nginx to configure SSL certificate. Refer to the tutorial: https://icnt94i5ctj4.feishu.cn/docx/GnYOdMNJOoRCljx1ctecsj9cnRe

### 5. How to improve Xiaozhi's conversation response speed?

This project's default configuration is a low-cost solution. It is recommended that beginners first use the default free models to solve the "can it run" problem, then optimize for "running faster".
To improve response speed, you can try replacing various components. Since version `0.5.2`, the project supports streaming configuration, which improves response speed by approximately `2.5 seconds` compared to earlier versions, significantly improving user experience.

| Module Name | Free Entry-Level Settings | Streaming Configuration |
|:---:|:---:|:---:|
| ASR (Speech Recognition) | FunASR (Local) | XunfeiStreamASR (iFlytek Streaming) |
| LLM (Large Model) | glm-4-flash (Zhipu) | qwen-flash (Alibaba Bailian) |
| VLLM (Vision Large Model) | glm-4v-flash (Zhipu) | qwen2.5-vl-3b-instructh (Alibaba Bailian) |
| TTS (Speech Synthesis) | LinkeraiTTS (Lingxi Streaming) | HuoshanDoubleStreamTTS (Volcano Streaming) |
| Intent (Intent Recognition) | function_call (Function Call) | function_call (Function Call) |
| Memory (Memory Function) | mem_local_short (Local Short-term Memory) | mem_local_short (Local Short-term Memory) |

If you care about the latency of each component, please refer to the [Xiaozhi Component Performance Test Report](https://github.com/xinnan-tech/xiaozhi-performance-research), and you can test in your own environment using the test methods in the report.

### 6. I speak slowly, and Xiaozhi keeps interrupting me during pauses

Suggestion: Find the following section in the configuration file and increase the value of `min_silence_duration_ms` (for example, change it to `1000`):

```yaml
VAD:
  SileroVAD:
    threshold: 0.5
    model_dir: models/snakers4_silero-vad
    min_silence_duration_ms: 700  # If you have long pauses when speaking, increase this value
```

### 7. Deployment Related Tutorials
1. [How to perform simplified deployment](./Deployment.md)<br/>
2. [How to perform full module deployment](./Deployment_all.md)<br/>
3. [How to deploy MQTT gateway to enable MQTT+UDP protocol](./mqtt-gateway-integration.md)<br/>
4. [How to automatically pull the latest code and auto-compile and start](./dev-ops-integration.md)<br/>
5. [How to integrate with Nginx](https://github.com/xinnan-tech/xiaozhi-esp32-server/issues/791)<br/>

### 9. Firmware Compilation Related Tutorials
1. [How to compile your own Xiaozhi firmware](./firmware-build.md)<br/>
2. [How to modify OTA address based on Brother Xia's pre-compiled firmware](./firmware-setting.md)<br/>
3. [How to configure firmware OTA auto-upgrade for single module deployment](./ota-upgrade-guide.md)<br/>

### 10. Extension Related Tutorials
1. [How to enable phone number registration for the Control Panel](./ali-sms-integration.md)<br/>
2. [How to integrate HomeAssistant for smart home control](./homeassistant-integration.md)<br/>
3. [How to enable vision model for photo recognition](./mcp-vision-integration.md)<br/>
4. [How to deploy MCP endpoint](./mcp-endpoint-enable.md)<br/>
5. [How to connect to MCP endpoint](./mcp-endpoint-integration.md)<br/>
6. [How MCP methods get device information](./mcp-get-device-info.md)<br/>
7. [How to enable voiceprint recognition](./voiceprint-integration.md)<br/>
8. [News plugin source configuration guide](./newsnow_plugin_config.md)<br/>
9. [Knowledge base ragflow integration guide](./ragflow-integration.md)<br/>
10. [How to deploy context provider](./context-provider-integration.md)<br/>

### 11. Voice Cloning and Local Voice Deployment Tutorials
1. [How to clone voice in the Control Panel](./huoshan-streamTTS-voice-cloning.md)<br/>
2. [How to deploy and integrate index-tts local voice](./index-stream-integration.md)<br/>
3. [How to deploy and integrate fish-speech local voice](./fish-speech-integration.md)<br/>
4. [How to deploy and integrate PaddleSpeech local voice](./paddlespeech-deploy.md)<br/>

### 12. Performance Testing Tutorials
1. [Component speed testing guide](./performance_tester.md)<br/>
2. [Regular public test results](https://github.com/xinnan-tech/xiaozhi-performance-research)<br/>

### 13. For more questions, feel free to contact us for feedback

You can submit your questions in [issues](https://github.com/xinnan-tech/xiaozhi-esp32-server/issues).
