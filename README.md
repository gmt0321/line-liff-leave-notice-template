# 用 LINE 官方帳號建立「請假通知表單」的簡易流程
<img width="1122" height="1402" alt="image" src="https://github.com/user-attachments/assets/441f2e94-8a5a-4df1-8571-0a61d0b82840" />

## 一、可以做到什麼？

學生或家長在 LINE 官方帳號中按下「我要請假」後，可以開啟一個請假表單。

填寫完成後，表單會把固定格式的請假通知傳送到與官方帳號的聊天室中，例如：

```text
【請假通知】
請假日期：2026-07-01
請假時段：第 2 節至第 4 節
學生姓名：王小明
填寫者身分：家長
請假事由：身體不適，需就醫。

請務必至校務系統完成正式請假手續。
```

官方帳號再透過關鍵字自動回覆，提醒使用者：

```text
系統已收到請假通知。

請務必至校務系統完成正式請假手續，才算完成請假。
```

這個系統的功能是「提前通知老師」，不取代學校正式請假系統。

---

# 二、系統架構

```text
LINE 官方帳號圖文選單
        ↓
LIFF 請假表單
        ↓
GitHub Pages 網頁
        ↓
liff.sendMessages()
        ↓
LINE 官方帳號聊天室
        ↓
關鍵字自動回覆
```

簡單來說：

* GitHub Pages：放請假表單網頁。
* LIFF：讓網頁可以在 LINE 中開啟。
* `liff.sendMessages()`：把表單資料送回目前的 LINE 聊天室。
* LINE 官方帳號：提供圖文選單與自動回覆。

---

# 三、開始前需要準備的項目

請先準備：

1. 一個 LINE 官方帳號。
2. 一個 GitHub 帳號。
3. 一個 LINE Developers 帳號。
4. 一份 `index.html` 表單程式。

---

# 四、建立請假表單網頁

## 步驟 1：建立 GitHub Repository

登入 GitHub，建立一個新的 repository，例如：

```text
leave-notice-liff
```

建議設為 Public，方便使用 GitHub Pages 免費發布網頁。

---

## 步驟 2：建立 `index.html`

在 repository 根目錄建立一個檔案：

```text
index.html
```

這個檔案就是請假表單。

表單可包含：

* 請假日期
* 請假方式

  * 全天
  * 半天
  * 指定節次
* 學生姓名
* 填寫者身分
* 請假事由
* 已了解正式請假規定的確認欄位

表單送出時，程式會將資料整理成固定格式文字，再送到 LINE 聊天室。

核心程式如下：

```javascript
await liff.sendMessages([
  {
    type: 'text',
    text: message
  }
]);
```

其中 `message` 是整理好的請假通知內容。

---

# 五、開啟 GitHub Pages

進入 GitHub repository 後，依序選擇：

```text
Settings
→ Pages
→ Build and deployment
→ Source：Deploy from a branch
→ Branch：main
→ Folder：/(root)
→ Save
```

稍候片刻，GitHub 會產生網站網址，例如：

```text
https://你的GitHub帳號.github.io/leave-notice-liff/
```

請記下這個網址，後面建立 LIFF App 時需要填入。

---

# 六、建立 LINE Login Channel

登入 LINE Developers Console，建立或選擇一個 Provider。

Provider 可以理解成管理同一組 LINE 專案的資料夾。

接著建立：

```text
LINE Login Channel
```

建立時可使用以下內容：

| 欄位                  | 建議內容        |
| ------------------- | ----------- |
| Channel name        | 請假通知        |
| Channel description | 學生與家長請假通知表單 |
| App type            | Web app     |
| Email address       | 管理者常用電子郵件   |

建立完成後，進入這個 LINE Login Channel。

---

# 七、建立 LIFF App

在 LINE Login Channel 中選擇：

```text
LIFF
→ Add
```

建議設定如下：

| 欄位                | 建議設定                                    |
| ----------------- | --------------------------------------- |
| LIFF app name     | 請假通知                                    |
| Size              | Full                                    |
| Endpoint URL      | GitHub Pages 網址                         |
| Scope             | `openid`、`profile`、`chat_message.write` |
| Add friend option | Off                                     |

其中 Endpoint URL 填入剛才 GitHub Pages 的網址，例如：

```text
https://你的GitHub帳號.github.io/leave-notice-liff/
```

最重要的是勾選：

```text
chat_message.write
```

這個權限讓表單可以使用 `liff.sendMessages()`，把資料送回 LINE 聊天室。

建立完成後，LINE 會產生一組：

```text
LIFF ID
```

格式大致像：

```text
1234567890-AbCdEfGh
```

請記下這組 LIFF ID。

---

# 八、把 LIFF ID 填入 `index.html`

在 `index.html` 中找到：

```javascript
const LIFF_ID = '你的LIFF_ID';
```

改成剛剛建立的 LIFF ID，例如：

```javascript
const LIFF_ID = '1234567890-AbCdEfGh';
```

初始化 LIFF 的程式應包含：

```javascript
await liff.init({
  liffId: LIFF_ID
});
```

修改後儲存並 Commit 到 GitHub。

LIFF 初始化程式必須在 Endpoint URL 或其子路徑下執行，功能才有保障。

---

# 九、建立 LIFF URL

LIFF URL 的格式是：

```text
https://liff.line.me/LIFF_ID
```

例如：

```text
https://liff.line.me/1234567890-AbCdEfGh
```

這個網址會先由 LINE 開啟 LIFF，再載入 GitHub Pages 上的表單。

---

# 十、測試表單

建議先用自己的 LINE 帳號測試。

操作方式：

1. 將自己的 LINE 帳號加入官方帳號好友。
2. 在官方帳號聊天室中開啟 LIFF URL。
3. 確認請假表單可正常開啟。
4. 填入測試資料。
5. 按下「送出請假通知」。
6. 確認聊天室中出現請假通知。

使用者從聊天室開啟 LIFF 時，`liff.sendMessages()` 會以使用者身分把訊息送到目前開啟 LIFF 的聊天室。

---

# 十一、設定圖文選單「我要請假」

回到 LINE Official Account Manager，建立或編輯圖文選單。

將「我要請假」按鈕設定為：

```text
動作：URI
連結：https://liff.line.me/你的LIFF_ID
```

使用者之後按下「我要請假」，就會直接開啟表單。

---

# 十二、完成後的使用流程

```text
學生或家長
→ 開啟 LINE 官方帳號
→ 按圖文選單「我要請假」
→ 開啟 LIFF 請假表單
→ 填寫請假資料
→ 送出請假通知
→ 請假內容出現在聊天室
→ 官方帳號自動回覆確認
→ 使用者再至校務系統完成正式請假
```

---

# 十三、正式上線前檢查清單

```text
□ GitHub Pages 網址可以正常開啟
□ index.html 已填入正確的 LIFF ID
□ LIFF App 的 Endpoint URL 已填入 GitHub Pages 網址
□ LIFF App 已勾選 chat_message.write
□ 已從官方帳號聊天室成功測試送出請假通知
□ 圖文選單「我要請假」已設定 LIFF URL
□ 關鍵字【請假通知】的自動回覆已啟用
□ 自動回覆文字已清楚提醒使用者仍須完成校務系統請假
```

---

# 十四、常見問題

## 為什麼表單送出後，訊息會出現在官方帳號聊天室？

因為表單使用：

```javascript
liff.sendMessages()
```

這個功能會把訊息送到使用者目前開啟 LIFF 的聊天室。

因此，建議讓學生或家長從官方帳號的圖文選單開啟表單。

---

## 為什麼需要 LIFF ID？

LIFF ID 是 LINE 建立 LIFF App 後產生的識別碼。

網頁透過 LIFF ID 初始化 LIFF，LINE 才知道這個網頁是哪一個 LIFF App，並提供登入、聊天室傳訊等功能。建立 LIFF App 後，LINE 會同時產生 LIFF ID 與 LIFF URL。

---

## 表單是不是正式請假？

不是。

這個表單只負責：

```text
通知導師
＋
統一請假訊息格式
＋
提醒使用者完成後續手續
```

正式請假仍須依學校規定，在校務系統中完成。

```
```

