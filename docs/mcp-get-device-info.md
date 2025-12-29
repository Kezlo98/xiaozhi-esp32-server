# How to Get Device Information in MCP Methods

This tutorial will guide you on how to use MCP methods to get device information.

Step 1: Customize your `agent-base-prompt.txt` file

Copy the `agent-base-prompt.txt` file from the xiaozhi-server directory to your `data` directory and rename it to `.agent-base-prompt.txt`.

Step 2: Modify the `data/.agent-base-prompt.txt` file, find the `<context>` tag, and add the following code content inside the tag:
```
- **Device ID:** {{device_id}}
```

After adding, your `data/.agent-base-prompt.txt` file's `<context>` tag content should look approximately like this:
```
<context>
[IMPORTANT! The following information is provided in real-time, no need to call tools to query, please use directly:]
- **Device ID:** {{device_id}}
- **Current Time:** {{current_time}}
- **Today's Date:** {{today_date}} ({{today_weekday}})
- **Today's Lunar Date:** {{lunar_date}}
- **User's City:** {{local_address}}
- **Local 7-Day Weather Forecast:** {{weather_info}}
</context>
```

Step 3: Modify the `data/.config.yaml` file, find the `agent-base-prompt` configuration. Before modification:
```
prompt_template: agent-base-prompt.txt
```
Change to:
```
prompt_template: data/.agent-base-prompt.txt
```

Step 4: Restart your xiaozhi-server service.

Step 5: Add a parameter named `device_id`, type `string`, description `Device ID` to your MCP method.

Step 6: Re-wake Xiaozhi and have it call the MCP method to check if your MCP method can retrieve the `Device ID`.

