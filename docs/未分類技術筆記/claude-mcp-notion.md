---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
此篇文章是用來記錄如何拿到 claude code 如何串接 notion mcp
:::



## 懶人包

要串接 notion mcp，要先需要拿到 notion 的 Token，拿到後執行一段指令，安裝 notion mcp，就可以使用 claude code 跟 notion 互動了


### 1. 取得 token

首先點擊這個連結 - https://www.notion.so/profile/integrations/，進到 notion 新增 app 的地方，可以根據此 YT 看要怎麼新增：https://www.billprin.com/articles/notion-mcp-claude (1:00s 開始)。
新增好後，會得到 `Internal Integration Secret`
1. Internal Integration Secret: ntn_xxxxxxx 



### 2. 到  ~/.config/claude-code 設定 notionMCP


1. 進入 mcp json 檔案位置

```shell
$ cd ~/.config/claude-code
```

2. 開啟 json 檔案

```shell
$ vi .mcp.json
```

3. 新增設定，前面那一段是 slack 的，不用理他
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
    },
    "notionMCP": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.notion.com/mcp"]
    }
  }
}
```

### 3. 確認是否連結成功

```shell
$ claude mcp list

----
slack: npx -y @modelcontextprotocol/server-slack - ✓ Connected
....
```

這樣輸入後，會彈出另外一個視窗，上面就是設定當初 notion 的 token，最後點下連結，應該會顯示這段訊息。  
`Authorization successful! You may close this window and return to the CLI.`  




### 參考連結
1. notion integration : https://www.notion.so/profile/integrations/
2. Claude Desktop + MCP + Notion API 完整設定指南：https://claude.ai/public/artifacts/3831d5d6-ae84-4708-b261-9afa64f7538d
3. 










