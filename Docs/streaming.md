# 為 `any-chat-completions-mcp` 伺服器添加流式傳輸支援

## 簡介

`any-chat-completions-mcp` 是一個 MCP 伺服器，它允許將 Claude 與任何 OpenAI SDK 相容的 Chat Completion API 整合。本文件描述如何為 `any-chat-completions-mcp` 伺服器添加流式傳輸支援。

## 準備工作

1.  確認已安裝 Node.js 和 npm。
2.  確認已獲取 OpenAI API 金鑰。
3.  確認已安裝 `@modelcontextprotocol/sdk` 和 `openai` 庫。

```bash
npm install @modelcontextprotocol/sdk openai
```

## 修改程式碼

1.  修改 `build/index.js` 文件，以啟用流式傳輸。

```javascript
 84 |             const client = new OpenAI({
 85 |                 apiKey: AI_CHAT_KEY,
 86 |                 baseURL: AI_CHAT_BASE_URL,
 87 |             });
 88 |             const chatCompletion = await client.chat.completions.create({
 89 |                 messages: [{ role: 'user', content: content }],
 90 |                 model: AI_CHAT_MODEL,
 91 |                 stream: true, // 啟用流式傳輸
 92 |             });
 93 |
 94 |             const contentArray = [];
 95 |             for await (const chunk of chatCompletion) {
 96 |               contentArray.push({
 97 |                 type: "text",
 98 |                 text: chunk.choices[0]?.delta?.content || ""
 99 |               });
100 |             }
101 |
102 |           return {
103 |               content: contentArray
104 |           };
```

2.  重新構建伺服器。

```bash
npm run build
```

## 測試

1.  啟動 `any-chat-completions-mcp` 伺服器。
2.  使用 MCP 客戶端連接到伺服器。
3.  發送聊天訊息到伺服器。
4.  確認伺服器可以流式地返回聊天完成結果。
5.  為了確保執行 AI 知道數據傳送何時完成，可以在 `build/index.js` 文件中添加以下程式碼，在 `content` 陣列的末尾添加一個包含結束標記 `[DONE]` 的物件：

```javascript
101 |
102 |           contentArray.push({
103 |             type: "text",
104 |             text: '[DONE]'
105 |           });
```