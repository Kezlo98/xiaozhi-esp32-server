# get_news_from_newsnow Plugin News Source Configuration Guide

## Overview

The `get_news_from_newsnow` plugin now supports dynamic configuration of news sources through the Web management interface, no longer requiring code modifications. Users can configure different news sources for each agent in the control panel.

## Configuration Methods

### 1. Configure via Web Management Interface (Recommended)

1. Log in to the control panel
2. Go to the "Role Configuration" page
3. Select the agent you want to configure
4. Click the "Edit Functions" button
5. Find the "NewsNow News Aggregation" plugin in the right parameter configuration area
6. Enter semicolon-separated Chinese names in the "News Source Configuration" field

### 2. Configuration File Method

Configure in `config.yaml`:

```yaml
plugins:
  get_news_from_newsnow:
    url: "https://newsnow.busiyi.world/api/s?id="
    news_sources: "The Paper;Baidu Hot Search;Cailian Press;Weibo;Douyin"
```

## News Source Configuration Format

News source configuration uses semicolon-separated Chinese names, format:

```
ChineseName1;ChineseName2;ChineseName3
```

### Configuration Example

```
The Paper;Baidu Hot Search;Cailian Press;Weibo;Douyin;Zhihu;36Kr
```

## Supported News Sources

The plugin supports the following news source Chinese names:

- The Paper (Pengpai Xinwen)
- Baidu Hot Search (Baidu Resou)
- Cailian Press (Cailianshe)
- Weibo
- Douyin
- Zhihu
- 36Kr
- Wallstreet News (Huaerjie Jianwen)
- IT Home (IT Zhijia)
- Toutiao (Jinri Toutiao)
- Hupu
- Bilibili (Bilibibili)
- Kuaishou
- Xueqiu
- Gelonghui
- Fabu Finance (Fabu Caijing)
- Jin10 Data (Jinshi Shuju)
- Nowcoder (Niuke)
- SSPAI (Shaoshu Pai)
- Juejin (Xitu Juejin)
- ifeng (Fenghuang Wang)
- Chongbuluo
- Zaobao (Lianhe Zaobao)
- Coolapk (Ku'an)
- PCBeta (Yuanjing Luntan)
- Reference News (Cankao Xiaoxi)
- Sputnik News (Weixing Tongxun She)
- Baidu Tieba
- Reliable News (Kaopu Xinwen)
- And more...

## Default Configuration

If no news sources are configured, the plugin will use the following default configuration:

```
The Paper;Baidu Hot Search;Cailian Press
```

## Usage Instructions

1. **Configure News Sources**: Set the Chinese names of news sources in the Web interface or config file, separated by semicolons
2. **Call Plugin**: Users can say "broadcast news" or "get news"
3. **Specify News Source**: Users can say "broadcast The Paper news" or "get Baidu Hot Search"
4. **Get Details**: Users can say "tell me more about this news"

## How It Works

1. The plugin accepts Chinese names as parameters (e.g., "The Paper")
2. Based on the configured news source list, it converts Chinese names to corresponding English IDs (e.g., "thepaper")
3. Uses the English ID to call the API to get news data
4. Returns news content to the user

## Notes

1. Configured Chinese names must exactly match the names defined in CHANNEL_MAP
2. Configuration changes require restarting the service or reloading the configuration
3. If configured news sources are invalid, the plugin will automatically use default news sources
4. Use English semicolons (;) to separate multiple news sources, do not use Chinese semicolons (；)
