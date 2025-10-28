---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
此篇文章是用來記錄如何拿到 claude code 如何串接 slack mcp
:::



## 懶人包

要串接 slack mcp，要先需要拿到 slack app 的 teamID、appToken，拿到後執行一段指令，安裝 slack mcp，就可以使用 claude code 跟 slack 互動了


### 1. 取得 slack token、teamID

首先點擊這個連結 - https://api.slack.com/apps，進到 slack 新增 app 的地方，可以根據此 YT 看要怎麼新增：https://www.youtube.com/watch?v=76HJ6f5ucKs (3:40s 開始)。
新增好後，會得到這兩個資量
1. Bot User OAuth Token: xoxb-XXXXXXXXXXXX (這個 token 在 `OAuth & Permissions`)
2. teamID: TXXXXXXXX (ID 可以在線上版的 slack 找到，剛剛影片的 5:30s 有說到)

### 2. 執行安裝程式
```shell
$ claude mcp add slack npx -y @modelcontextprotocol/server-slack
```

輸入完上面的資料後，會新增 mcp 的檔案，位置在 `/Users/yee0526/.config/claude-code/.mcp.json`，更通用的路徑是 `~/.config/claude-code/.mcp.json`。


### 3. 把 token 和 teamID 加進去

```shell
$ export SLACK_BOT_TOKEN="xoxb-你的實際token"
$ export SLACK_TEAM_ID="T你的實際team-id"
```

### 4. 手動測試看 mcp 有沒有連結成功

```shell
$ SLACK_BOT_TOKEN="你的token" SLACK_TEAM_ID="你的team-id" npx -y @modelcontextprotocol/server-slack
```

### 5. 執行 claude mcp list
```shell
$ claude mcp list

-----
應該會顯示:
slack: npx -y @modelcontextprotocol/server-slack - ✓ Connected
```


### 6. 進到 .mcp.json 看內容

1. 進入 mcp json 檔案位置

```shell
$ cd ~/.config/claude-code
```

2. 開啟 json 檔案

```shell
$ vi .mcp.json
```

可以看到剛剛設定的 token 和 teamId，如果想要更換連結的 slack 帳號，也可以在這邊修改

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-......",
        "SLACK_TEAM_ID": "T......."
      }
    }
  }
}
```


### 7. 進入 claude 確認
```shell
$ claude

----
我現在有 claude slack mcp 連結嗎？
--
claude 會跟說明是否有成功連結
```


### 8. 把 mcp bot 加進指定的頻道

確認好是否有連結後，可以把機器人加入指定的頻道中，這樣 mcp bot 才可以在此頻道發訊息

```md
/invite @Test-mcp-bot(這個是我 app 的名稱)
```


### 9. 進到 claude 指定頻道輸入資料

```shell
$ claude

------
針對某個頻道輸入：我是 mcp 機器人
```

這樣應該就可以看到 slack 那邊有 mcp 機器人回應







### 參考連結
1. Claude Code - Where is my MCP configuration stored: https://www.reddit.com/r/ClaudeAI/comments/1lm7fbc/claude_code_where_is_my_mcp_configuration_stored/
2. 方格子 (Claude Code 添加 MCP 伺服器)：https://vocus.cc/article/68a32baffd897800016553f7
3. 










