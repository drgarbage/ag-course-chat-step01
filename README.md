# Class 02 Example01 — 第一個 AI 單輪對話助理

## 範例目標

建立一個最基本的單輪對話網頁助理，使用 Gemini API 的 `generateContentStream`，以串流（Stream）方式接收並顯示 AI 的回應。

## 功能清單

- ✅ 最基本的 HTML 結構（樣式由課程資源站的 `chat.css` 提供，離線時樣式會消失但功能正常）
- ✅ 雙擊 `index.html` 直接於瀏覽器執行 (`file://` 協議)，無須運行本地伺服器
- ✅ 呼叫 `generateContentStream` 取得單輪串流回應

> 本範例刻意保持極簡：沒有停止生成、Enter 送出、輸入法防護等功能，這些會在 example03 的產品級版本補齊。

## 使用方式

1. 用瀏覽器直接開啟 `index.html`
2. 將程式碼中的 `YOUR_API_KEY_HERE` 替換為您的 Gemini API Key
3. 在輸入框中輸入訊息，點「送出」按鈕
4. 觀察 API 串流回傳效果

## 程式碼結構

- `index.html` 內含極簡的 HTML 與 ESM 載入的 JavaScript 邏輯：
  1. **載入 SDK**：自 `https://esm.run/@google/generative-ai` 載入（本範例刻意使用舊版 SDK，可 file:// 直開）
  2. **調用 API**：使用 `model.generateContentStream` 實作單輪串流


## Step 1: 製作介面

新增一個 index.html 文件，貼上以下內容...

```html
<meta charset="utf-8">

<link rel="stylesheet" href="https://agcourse.web.app/stylesheets/chat.css">

<body>
  <div id="chatArea"></div>
  <div id="toolbar">
    <input id="userInput" />
    <button onclick="send()">送出</button>
  </div>
</body>
```

## Step 2: 撰寫互動程式

在上面的內容後面貼上...

```html
<script type="module">
  import { GoogleGenerativeAI } from 'https://esm.run/@google/generative-ai';

  const genAI = new GoogleGenerativeAI('貼上API_KEY');
  const model = genAI.getGenerativeModel({ model: 'gemini-2.5-flash' });

  // 輔助函式：插入對話
  function say(role, text) {
    const p = document.createElement('p');
    p.classList.add(role);
    p.textContent = text;
    chatArea.appendChild(p);
    p.scrollIntoView(false);
    return p;
  }

  window.send = async () => {
    const text = userInput.value.trim();
    if (!text) return;

    userInput.value = '';
    say('user', text);
    const message = say('ai', '思考中...');

    try {
      const result = await model.generateContentStream(text);
      message.textContent = '';

      for await (const chunk of result.stream) {
        message.textContent += chunk.text();
        message.scrollIntoView(false);
      }
    } catch (error) {
      message.textContent = '出錯: ' + error.message;
    }
  }

</script>
```