---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
此篇文章是用來記錄如何拿到 claude 付費 token 後，在 terminal 直接使用 claude
:::



## 懶人包

### 安裝 claude code

```shell
$ npm install -g @anthropic-ai/claude-code
```

```shell
$ claude
```

這樣輸入完後，claude 會給要你輸入一些選項，然後最後會要你登入，彈出登入視窗，如果是透過付費的 token，就可以跳過登入的步驟，使用該 token 來付費使用


### 生成 anthropic_key.sh
1. 打開 terminal
2. cd ~/.claude/
3. echo "echo \"$ANTHROPIC_API_KEY\"" > ~/.claude/anthropic_key.sh
4. chmod +x ~/.claude/anthropic_key.sh

ps1. 直接把 `$ANTHROPIC_API_KEY` 換成付費的 token
ps2. 如果今天沒有這個 token，會需要登入


### 編輯 settings.json

1. 新增檔案：`vi ~/.claude/settings.json`
2. 新增內容
```json
{	
	"apiKeyHelper": "~/.claude/anthropic_key.sh" --> #多加這行
}
```


### 再次登入

上面這樣新增好後，就可以再次輸入 `claude`，然後就會發現不用登入，就可以使用 claude

















