# RESTful API 串接

## 大綱

1. 主線任務說明
2. 實作練習
    1. 建立 API 設定
    2. 建立登入頁面
    3. 建立狀態管理
    4. 實作登入功能
    5. 建立產品管理頁面
    6. 實作產品資料載入

## 1. 主線任務說明

第二週開始要接觸到課程的 API，是非常重要的一個章節，這份 API 會一直跟到直播班結束，同學們可用此份 API 建立屬於自己的電商資料，請務必在本週的時間內完成。

請**使用 vite** 完成以下需求：

- 使用者可以從登入頁面登入，並轉到後台商品頁面
- 使用者若無登入直接進入商品頁面，會被導回登入頁面
- 使用者可以查看產品列表
- 使用者可以點擊單一產品，查看詳細資訊

### **API 申請說明**

這個 API 與『 Vue 直播班』是同一個資料庫，因此若已經有帳號了，也可以登入該帳號並**申請一個新的 API 路徑**就好。

若是還沒有帳號，就註冊一個來使用就可以哩～

**課程 API 相關網址：**

- [註冊連結、測試管理平台](https://ec-course-api.hexschool.io/)
- [API 文件](https://hexschool.github.io/ec-courses-api-swaggerDoc/)

登入頁面 API

- [登入串接 POST API](https://hexschool.github.io/ec-courses-api-swaggerDoc/#/%E7%99%BB%E5%85%A5%E5%8F%8A%E9%A9%97%E8%AD%89/post_v2_admin_signin)
- [驗證登入串接 POST API](https://hexschool.github.io/ec-courses-api-swaggerDoc/#/%E7%99%BB%E5%85%A5%E5%8F%8A%E9%A9%97%E8%AD%89/post_v2_api_user_check)
產品頁面 API

- [取得產品資料串接 GET API](https://hexschool.github.io/ec-courses-api-swaggerDoc/#/%E7%AE%A1%E7%90%86%E6%8E%A7%E5%88%B6%E5%8F%B0%20-%20%E7%94%A2%E5%93%81%20(Products)/get_v2_api__api_path__admin_products)

頁面模板

- [登入到產品頁面的頁面模板](https://codepen.io/hexschool/pen/zxOzORN)（因這週尚未教到路由，所以會先用 三元運算子 切換畫面呈現）

作業須符合此[作業規範](https://hackmd.io/XbKPYiE9Ru6G0sAfB5PBJw)

每週主線任務範例：https://github.com/hexschool/react-training-chapter-2025

## 專案建立

### 使用 Vite 建立專案

```bash
# 建立新專案
npm create vite@latest

# 進入專案目錄
cd week2

# 安裝依賴
npm install axios bootstrap@5.3.8

# 啟動開發伺服器
npm run dev
```

###

### 專案結構

```
week2/
├── src/
│   ├── App.jsx        # 主元件
│   ├── main.jsx       # 應用程式入口
│   └── assets/
│       └── style.css  # 樣式
├── index.html         # HTML 模板
├── package.json       # 專案設定
└── vite.config.js     # Vite 設定
```

### 流程

```jsx
1. 使用者輸入帳密 → 呼叫登入 API
2. 登入成功 → 取得 token
3. 將 token 存到 Cookie
4. 將 token 放進 axios Authorization header
5. 使用 token 取得產品資料
6. 根據登入狀態切換畫面
```

## 2. 實作步驟

### 建立 .env 檔案

- 變數名稱一定要以 `VITE_` 開頭
- .env 不要上傳到 GitHub

```bash
VITE_API_BASE=https://ec-course-api.hexschool.io/v2
VITE_API_PATH=你的API路徑
```

### 載入 bootstrap 的 css 與 js

```jsx
// main.jsx
import 'bootstrap/dist/css/bootstrap.min.css';
import 'bootstrap/dist/js/bootstrap.bundle.min.js';
```

### 建立 API 設定

```jsx
import axios from "axios";

// API 設定
const API_BASE = import.meta.env.VITE_API_BASE;
const API_PATH = import.meta.env.VITE_API_PATH;
```

### 引入 style

```jsx
// App.jsx
import "./assets/style.css";
```

### 建立登入頁面

切換完全 `isAuth` 狀態控制：

- false → 顯示登入頁
- true → 顯示產品管理頁
- [forms](https://getbootstrap.com/docs/5.3/forms/floating-labels/)

```jsx
// 登入表單畫面
{!isAuth ? (
  <div className="container login">
    <div className="row justify-content-center">
      <h1 className="h3 mb-3 font-weight-normal">請先登入</h1>
      <div className="col-8">
        <form id="form" className="form-signin" onSubmit={handleSubmit}>
          <div className="form-floating mb-3">
            <input
              type="email"
              className="form-control"
              id="username"
              name="username"
              placeholder="name@example.com"
              value={formData.username}
              onChange={handleInputChange}
              required
              autoFocus
            />
            <label htmlFor="username">Email address</label>
          </div>
          <div className="form-floating">
            <input
              type="password"
              className="form-control"
              id="password"
              name="password"
              placeholder="Password"
              value={formData.password}
              onChange={handleInputChange}
              required
            />
            <label htmlFor="password">Password</label>
          </div>
          <button
            className="btn btn-lg btn-primary w-100 mt-3"
            type="submit"
          >
            登入
          </button>
        </form>
      </div>
    </div>
    <p className="mt-5 mb-3 text-muted">&copy; 2025~∞ - 六角學院</p>
  </div>
) : (
  // 登入後的產品管理頁面 (同第一週)
)}
```

### 建立狀態管理

```jsx
function App() {
  // 表單資料狀態(儲存登入表單輸入)
  const [formData, setFormData] = useState({
    username: "",
    password: "",
  });
  // 登入狀態管理(控制顯示登入或產品頁）
  const [isAuth, setIsAuth] = useState(false);
  // 產品資料狀態
  const [products, setProducts] = useState([]);
  // 目前選中的產品
  const [tempProduct, setTempProduct] = useState(null);
}
```

### 表單輸入處理

```jsx
// 表單輸入處理
const handleInputChange = (e) => {
  const { name, value } = e.target;
  setFormData((prevData) => ({
    ...prevData, // 保留原有屬性
    [name]: value, // 更新特定屬性
  }));
};
```

### Cookie 操作

[設定 cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie#example_1_simple_usage)、[讀取 cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie#example_2_get_a_sample_cookie_named_test2)

```jsx
// 設定 Cookie
document.cookie = `hexToken=${token};expires=${new Date(expired)};`;

// 讀取 Cookie
const token = document.cookie
  .split("; ")
  .find((row) => row.startsWith("hexToken="))
  ?.split("=")[1];
```

### 實作登入功能

在 API 中，我們通常會在 `Authorization` Header 傳送 Token

常見格式如下：

```jsx
Authorization: Bearer xxxxxxx.yyyyyyy.zzzzzzz
```

課程使用的 API（Swagger）不需要加 Bearer，請直接傳送 `Token` 字串即可，否則會驗證失敗

**設定 Authorization Header**

[axios 全域設置](https://axios-http.com/zhtw/docs/config_defaults)

登入成功後，請將 Token 設定到 axios 的預設 Header，之後所有 API 請求都會自動帶上 Token

```jsx
// 修改實體建立時所指派的預設配置
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
```

補充：axios 也可以透過 create 建立實體並設定預設 Header

```jsx
// 建立實體時指派預設配置
const instance = axios.create({
  baseURL: 'https://api.example.com'
});
```

- 登入 api `${API_BASE}/admin/signin`
    
    ```jsx
    // 登入提交處理
    const handleSubmit = async (e) => {
      e.preventDefault();
    
      try {
        // 發送登入請求
        const response = await axios.post(`${API_BASE}/admin/signin`, formData);
        const { token, expired } = response.data;
    
        // 儲存 Token 到 Cookie
        document.cookie = `hexToken=${token};expires=${new Date(expired)};`;
    
        // 設定 axios 預設 header
        axios.defaults.headers.common.Authorization = `${token}`;
    
        // 載入產品資料
        getData();
    
        // 更新登入狀態
        setIsAuth(true);
      } catch (error) {
        alert("登入失敗: " + error.response.data.message);
      }
    };
    ```
    
- 驗證 token api `${API_BASE}/api/user/check`
    
    ```jsx
    // 檢查登入狀態
    const checkLogin = async () => {
      try {
        // 從 Cookie 取得 Token
        const token = document.cookie
          .split("; ")
          .find((row) => row.startsWith("hexToken="))
          ?.split("=")[1];
    
        console.log("目前 Token：", token);
    
        if (token) {
          axios.defaults.headers.common.Authorization = token;
    
          // 驗證 Token 是否有效
          const res = await axios.post(`${API_BASE}/api/user/check`);
          console.log("Token 驗證結果：", res.data);
        }
      } catch (error) {
        console.error("Token 驗證失敗：", error.response?.data);
      }
    };
    ```
    
- 驗證
    
    ```jsx
    {/* 功能按鈕 */}
    <button
      className="btn btn-danger mb-5"
      type="button"
      onClick={checkLogin}
    >
      確認是否登入
    </button>
    
    ```
    
    補充: 實務上會在頁面載入時自動檢查登入狀態
    
    ```jsx
    useEffect(() => {
      checkLogin();
    }, []);
    ```
    

登入時設定 token，是給「本次操作」使用

checkLogin 是為了「重新整理頁面後」重新取回 token

### 建立產品管理頁面

整合第一週作業

產品 API 需要 token，因此必須在「登入成功後」才能呼叫

```jsx
// 產品管理頁面 (同第一週)
{
  isAuth && (
    <div className="container">
      <div className="row mt-5">
        <div className="col-md-6">
				  {/* 產品列表區塊 (同第一週) */}
        </div>
        <div className="col-md-6">{/* 產品詳情區塊 (同第一週) */}</div>
      </div>
    </div>
  );
}
```

### 實作產品資料載入

登入後取得產品資料

取得產品資料 api `${API_BASE}/api/${API_PATH}/admin/products`

```jsx
// 取得產品資料
const getData = async () => {
  try {
    const response = await axios.get(
      `${API_BASE}/api/${API_PATH}/admin/products`
    );
    console.log("產品資料：", response.data);
    setProducts(response.data.products);
  } catch (err) {
    console.error("取得產品失敗：", err.response?.data?.message);
  }
};
```