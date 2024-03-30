---
title: "一次搞懂 Fetch"
date: 2022-07-01
categories: "Development"
---

## 目錄

- [In A Nutshell](#in-a-nutshell)
  - [Fetch 是什麼，用來做什麼？](#fetch-是什麼用來做什麼)
  - [Fetch 的語法](#fetch-的語法)
  - [Fetch 的回傳值](#fetch-的回傳值)
- [Fetch 教學](#fetch-教學)
  - [使用 `.then` 串接 Fetch 函式](#使用-then-串接-fetch-函式)
    - [使用 Fetch 發送 GET 請求](#使用-fetch-發送-get-請求)
    - [【完整版】使用 Fetch 發送 GET 請求](#完整版使用-fetch-發送-get-請求)
  - [使用 `async await` 串接 Fetch 函式](#使用-async-await-串接-fetch-函式)
    - [使用 `async await` 與 Fetch 發送 GET 請求](#使用-async-await-與-fetch-發送-get-請求)
    - [【完整版】使用 `async await` 與 Fetch 發送 GET 請求](#完整版使用-async-await-與-fetch-發送-get-請求)

# In A Nutshell

## Fetch 是什麼，用來做什麼？

- `fetch` 可以串接 API ，也就是**取得**來自伺服器或**傳送**至伺服器資料（JSON、blob 等）。
- `fetch` 常常和 `.then` 或 `async await` 寫法一同出現，因此本篇會一起介紹。

## Fetch 的語法

```jsx
fetch(resource)
fetch(resource, init)
```

- `**resource**`
    - **你想要取得／傳送資訊的路徑，通常就是一個 URL** （但也可以是本機的相對路徑）
- **`init` (optional)**
    - 你想要的額外設定。預設留空代表你要做出 GET Request ；**若想改為 POST Request 的話需要加入以下設定**（整個東西要用物件包起來 `{}` ）。：
        - `method`: Request method ，像是 GET 或 POST。
        - `headers`: 加在 request 的 headers ，整個東西要用物件包起來 `{}` 。
        - `body`: 你想要傳送 POST 的內容（沒錯，GET 不能含有 `body` ）
        - `mode`: Request mode ，像是 `cors`、`no-cors`、`same-origin`
        - `credentials`: Controls what browsers do with credentials ([cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies), [HTTP authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication) entries, and TLS client certificates).
        - [⋯⋯以及更多](https://developer.mozilla.org/en-US/docs/Web/API/fetch#browser_compatibility)。

## Fetch 的回傳值

- `fetch` 會回傳一個 [Promise](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Promise) ，需要用 [`.then`](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) 或是 [`async`](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Statements/async_function) [`await`](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Operators/await) 方法串接。
    - 快速理解 Promise ：**`Promise`** 物件代表一個即將完成、或失敗的非同步操作，以及它所產生的值。我們可以設置一個 Promise 物件嘗試成功（resovle）和失敗（rejected）時不同的行為。
- **除非網路錯誤、資源路徑（URL）輸入錯誤或其他中斷 request 的情況，不然 fetch 的 request 一律都是回傳 resolve （成功）！就算我們 fetch 的資源有 error status 404 或 500 ，也會被 fetch 視為 resolve ！**

# Fetch 教學

## 使用 `.then` 串接 Fetch 函式

我們先練習使用 `.then` 串接 Fetch 函式。

假設我們要從 [`http://foo.com`](http://foo.com) 取得一個 JSON 資料，我們的 `fetch` 方法就會是 GET，資料解析方式會是 `json()` （因為網路上傳來的 JSON 資料無法直接在 JavaScript 中使用，你需要先解析）

從 `fetch` 開始到成功取得資料本身共有 3 步驟：

1. 使用 `fetch` 取得資源 URL，這時回傳的值是一個 promise，無法直接使用。
2. 為了等待 promise resolve ，我們要接上一個 `.then` ，並在裡面使用 `json()` 來把取得的 response 解析成 JavaScript 可以理解的 JSON 物件。注意！這個步驟也要花時間，也會回傳一個 promise，因此我們需要在後方再接上一個 `.then` 。
3. 我們將解析完成的 JSON 物件取做 `data` ，並將他 console.log 出來。完成。

### 使用 Fetch 發送 GET 請求

```jsx
fetch('http://foo.com') // 取得來源；因為我們是 GET Request 所以不用額外參數
.then(response => response.json())
.then(data => console.log(data)) 
```

然而，這段程式碼沒有導入任何抓錯功能，且就像前面說的，若 `fetch` 傳回的 status code 為 400 或 500 ，程式碼也會像沒有出錯般繼續執行。

---

### 【完整版】使用 Fetch 發送 GET 請求

為了避免這個問題，我們接下來導入能夠抓錯的程式碼。

- `if / else` 流程控制讓我們能抓住 URL status 回傳錯誤（如 404、500 等）的可能性。
- `catch(err => console.error(err))` **能夠幫我們抓到 fetch 回傳 promise 的錯誤，包含找不到的 URL，或串接期間任何一行程式碼出現的問題**。

```jsx
fetch('http://foo.com')
.then(response => {
	if (!response.ok){
		console.log('ERROR!');
	}
	else{
		console.log('SUCCESS');
		return response.json()
	}
})
.then(data => console.log(data))
.catch(err => console.error(err));
```

些下來我們要來介紹另一種做法，叫做 `async await` 。這種方法（似乎）會讓你的 code 變得比較乾淨、易讀。

---

## 使用 `async await` 串接 Fetch 函式

要將我們前面的例子改為使用到 `async await` 的方法，有以下幾個變化：

- 將整個 `fetch` API 整合到一個 function。
- 用 `await` 語法取代 `.then` 串接。
- 將每個回傳值都儲存為一個變數。

我們先來看看最基礎、不加錯誤處理的程式碼：

### 使用 `async await` 與 Fetch 發送 GET 請求

```jsx
async function getInfo(){ // 👈這個 function 非同步，因此要在宣告時加上 "async"！
	const result = await fetch('http://foo.com'); // 👈 這個函式會回傳一個 promise ，因此要在前加上 "await"！
	const data = await result.json(); // 👈 這個函式會回傳一個 promise ，因此要在前加上 "await"！
	console.log(data);
}

getInfo() // 別忘了呼叫他才會執行喔：）
```

### 【完整版】使用 `async await` 與 Fetch 發送 GET 請求

```jsx
async function getInfo(){
	const result = await fetch('http://foo.com');
	if (!result.ok){
		console.log('ERROR');
	} else {
		const data = result.json();
		console.log(data);
	}
}

getInfo()
```

**那麼現在要怎麼 `catch` 連線問題或是 URL 錯誤呢？很簡單，我們在呼叫函式時在後面接上 `.catch()` 就行。**

```jsx
getInfo().catch(err => console.error(err)); // 會接住執行函式期間的錯誤，以及 URL 和網路錯誤。
```