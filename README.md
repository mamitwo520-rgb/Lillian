## 完整流程說明：LINE Bot 自動推播圖片

---

### 第一階段：LINE Developers 設定

**Step 1：註冊 LINE Developers 帳號**
1. 前往 [developers.line.biz](https://developers.line.biz/)
2. 點右上角 **「Log in」**
3. 用你的 LINE 帳號登入

**Step 2：建立 Provider**
1. 登入後點 **「Create a new provider」**
2. 填入 Provider 名稱（例如：我的餐廳）
3. 點 **「Create」**

**Step 3：建立 Messaging API Channel**
1. 點 **「Create a new channel」**
2. 選 **「Messaging API」**
3. 填入以下資料：
   - Channel name：Bot 的名稱
   - Channel description：簡短說明
   - Category：選適合的類別
4. 勾選同意條款 → 點 **「Create」**

**Step 4：取得 Channel Access Token**
1. 進入剛建立的 Channel
2. 點上方 **「Messaging API」** 頁籤
3. 滾到最下面找到 **「Channel access token」**
4. 複製並妥善保存這個 Token

---

### 第二階段：GitHub 設定（存放圖片）

**Step 5：註冊 GitHub 帳號**
1. 前往 [github.com](https://github.com)
2. 點 **「Sign up」** 註冊帳號
3. 填入 Email、密碼、使用者名稱

**Step 6：建立 Repository**
1. 登入後點右上角 **「+」→「New repository」**
2. 填入 Repository 名稱（例如：MyImages）
3. 選 **「Public」**（公開，LINE 才能存取）
4. 點 **「Create repository」**

**Step 7：上傳圖片**
1. 進入 Repository
2. 點 **「Add file」→「Upload files」**
3. 把圖片拖曳進去，命名建議用英文（例如：1.jpg、2.jpg）
4. 點 **「Commit changes」**

**Step 8：取得圖片 Raw URL**

每張圖片的網址格式是：
```
https://raw.githubusercontent.com/你的帳號/Repository名稱/main/圖片檔名.jpg
```

---

### 第三階段：Google Sheets 設定

**Step 9：建立 Google Sheets**

建立一個新的試算表，格式如下：

| A（num） | B（url） | C (text 說明文字)
|---------|---------|---------|
| 1 | https://raw.githubusercontent.com/.../1.jpg | 這是第一張的圖片說明 |
| | https://raw.githubusercontent.com/.../2.jpg | 這是第二張的圖片說明 |
| | https://raw.githubusercontent.com/.../3.jpg | 這是第三張的圖片說明 |

> A1 填 `1`，代表下次要推播第幾張圖片，B 欄每列放一張圖片的 Raw URL，C 欄每列放一行想要詳細說明的文字

---

### 第四階段：Make 設定

**Step 10：註冊 Make 帳號**
1. 前往 [make.com](https://make.com)
2. 點 **「Get started free」** 註冊
3. 用 Google 帳號登入最方便

**Step 11：建立 Scenario**
1. 點左側 **「Scenarios」→「Create a new scenario」**
2. 點畫面中間的 **「+」** 開始新增模組

---

**模組 0：Schedule（定時觸發）**
1. 搜尋 **「Schedule」** 選擇它
2. 設定觸發頻率，例如每天幾點推播

---

**模組 1：Google Sheets - Get a Cell（讀取序號）**
1. 點 **「+」** 新增模組，搜尋 **「Google Sheets」**
2. 選 **「Get a Cell」**
3. 連線到你的 Google 帳號
4. 設定：
   - Search Method：**Search by path**
   - Drive：**My Drive**
   - Spreadsheet Name：你的試算表名稱
   - Sheet Name：**Sheet1**
   - Cell：**`A1`**

---

**模組 2：Google Sheets - Get a Cell（讀取 URL）**
1. 再新增一個 **「Google Sheets - Get a Cell」**
2. 設定相同的試算表，但 Cell 填：
   - **`B{{1.value}}`**（1 是模組1的編號，依實際編號調整）

---

**模組 3：Google Sheets - Get a Cell（讀取 TEXT）**
1. 再新增一個 **「Google Sheets - Get a Cell」**
2. 設定相同的試算表，但 Cell 填：
   - **`C{{1.value}}`**（1 是模組1的編號，依實際編號調整）
---

**模組 4：LINE - Send a Broadcast Message**
1. 新增 **「LINE」** 模組
2. 選 **「Send a Broadcast Message」**
3. Connection：點 **「Add」**，填入 **Channel Access Token**
4. 新增 Item，Type 選 **「Image」**
5. Original Content URL 填：
```
{{replace(replace(2.value; "github.com"; "raw.githubusercontent.com"); "/blob"; "")}}
```
6. Preview Image URL 填一樣的公式

> 注意：如果你的 Google Sheets 已經直接放 Raw URL，就直接填 `{{2.value}}` 就好

---

**模組 5：LINE - Send a Broadcast Message**
1. 新增 **「LINE」** 模組
2. 選 **「Send a Broadcast Message」**
3. Connection：點 **「Add」**，填入 **Channel Access Token**
4. 新增 Item，Type 選 **「TEXT」**
5. Original Content URL 填：
```
{{replace(replace(3.value; "github.com"; "raw.githubusercontent.com"); "/blob"; "")}}
```
6. Preview Image URL 填一樣的公式

> 注意：如果你的 Google Sheets 已經直接放 Raw URL，就直接填 `{{3.value}}` 就好

---

**模組 6：Google Sheets - Update a Cell（更新序號）**
1. 新增 **「Google Sheets - Update a Cell」**
2. 設定相同的試算表
3. Cell 填：**`A1`**
4. Value 填：
```
{{1.value * 1 + 1}}
```
5. 如果圖片有 10 張，想循環播放則填：
```
{{if(1.value * 1 >= 10; 1; 2.value * 1 + 1)}}
```

---

### 第五階段：測試執行

**Step 12：手動測試**
1. 點右下角 **「Run once」**
2. 確認每個模組都是綠色勾勾 ✅
3. 去 LINE 確認有收到圖片

**Step 13：開啟排程**
1. 確認測試成功後
2. 把左下角的開關切成**開啟**
3. Make 就會依照你設定的時間自動推播！

---

### 整體流程圖

```
[Schedule 定時觸發]
        ↓
[Google Sheets: 讀取 A1 序號]
        ↓
[Google Sheets: 讀取 B{n} 的圖片 URL]
        ↓
[LINE: Broadcast 圖片給所有好友]
        ↓
[Google Sheets: A1 序號 +1]
```

---