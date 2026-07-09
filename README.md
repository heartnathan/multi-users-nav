# Navigation Site

基於 Cloudflare Workers + D1 資料庫的網址導航站，支援多使用者、分類管理、主題切換。

[Original Repo](https://github.com/zz80900/navigation)

## 功能特性

- 多使用者支援：每個使用者獨立管理自己的分類和連結
- 超級管理員：user_id=1 為超級管理員，可管理所有使用者
- 使用者管理：新增/刪除/停用使用者、重設密碼
- 分類管理：建立/編輯/刪除分類，支援排序
- 連結管理：按分類篩選、15 條分頁、拖曳排序，建立/編輯/刪除并支援自訂圖示
- 分類管理：支援拖曳重新排序，左側列表醒目提示記憶
- 管理後台體驗：左側分類醒目提示記憶、行內操作懸浮顯示、分頁狀態與空態提示
- 主題切換：支援亮色/暗色主題，跟隨系統或手動切換
- 搜尋功能：即時搜尋連結名稱和URL
- 響應式設計：適配桌面端和行動端

## 技術棧

- **後端**: Cloudflare Workers (JavaScript)
- **資料庫**: Cloudflare D1 (SQLite)
- **前端**: 純 HTML/CSS/JavaScript
- **認證**: JWT (Web Crypto API)

## 專案結構

```
nav/
├── wrangler.toml              # Workers 設定
├── package.json               # 專案相依
├── db.sql                     # 資料庫初始化脚本
├── src/
│   ├── index.js               # Worker 入口，路由分發
│   ├── auth.js                # JWT 認證邏輯
│   ├── api/
│   │   ├── public.js          # 公開 API
│   │   ├── admin.js           # 管理 API
│   │   └── auth.js            # 登入 API
│   └── utils/
│       ├── response.js        # 統一回應格式
│       └── crypto.js          # 密碼加密
├── public/
│   ├── index.html             # 导航页
│   ├── admin.html             # 管理页
│   ├── login.html             # 登入页
│   ├── favicon.svg            # 網站圖示
│   ├── default_logo_icon.svg  # 預設連結圖示
│   ├── css/
│   │   ├── style.css          # 导航页樣式
│   │   └── admin.css          # 管理页樣式
│   └── js/
│       ├── app.js             # 导航页邏輯
│       ├── admin.js           # 管理页邏輯
│       └── login.js           # 登入邏輯
└── README.md
```

## 部署步驟

### 前置要求

- Node.js 18+
- Cloudflare 帳號
- Wrangler CLI

### 命令列部署

#### 1. 安裝相依

```bash
npm install
```

#### 2. 登入 Cloudflare

```bash
npx wrangler login
```

#### 3. 建立 D1 資料庫

```bash
npx wrangler d1 create nav-db
```

執行后会輸出資料庫 ID，将其更新到 `wrangler.toml`:

```toml
[[d1_databases]]
binding = "DB"
database_name = "nav-db"
database_id = "你的資料庫ID"
```

#### 4. 初始化資料庫

```bash
# 遠端資料庫
npx wrangler d1 execute nav-db --remote --file=db.sql

# 或本機資料庫（用于開發）
npx wrangler d1 execute nav-db --local --file=db.sql
```

#### 5. 設定 JWT 金鑰

在 Cloudflare Dashboard 或使用 wrangler 设置环境变量：

```bash
npx wrangler secret put JWT_SECRET
# 輸入一个隨機字串作为金鑰
```

#### 6. 本機開發

```bash
npx wrangler dev
```

存取 http://localhost:8787

#### 7. 部署到 Cloudflare

```bash
npx wrangler deploy
```

## 預設帳號

- 用户名: `admin`
- 密碼: `admin123`

**重要**: 部署后请立即修改預設密碼！

## API 說明

### 公開 API

| 方法 | 路徑 | 說明 |
|------|------|------|
| GET | /api/links | 取得目前用户的連結（需登入） |
| GET | /api/categories | 取得目前用户的分類（需登入） |

### 認證 API

| 方法 | 路徑 | 說明 |
|------|------|------|
| POST | /api/auth/login | 用户登入 |
| GET | /api/auth/check | 驗證 Token |

### 管理 API（需認證）

| 方法 | 路徑 | 說明 |
|------|------|------|
| GET | /api/admin/categories | 取得分類列表 |
| PUT | /api/admin/categories/reorder | 拖曳調整分類排序 |
| POST | /api/admin/categories | 建立分類 |
| PUT | /api/admin/categories/:id | 更新分類 |
| DELETE | /api/admin/categories/:id | 刪除分類 |
| GET | /api/admin/links | 取得連結列表，支援 `category_id`、`page`、`per_page` 查询 |
| POST | /api/admin/links | 建立連結 |
| PUT | /api/admin/links/:id | 更新連結 |
| DELETE | /api/admin/links/:id | 刪除連結 |
| PUT | /api/admin/links/reorder | 按分類拖曳排序并持久化 sort_order |
| PUT | /api/admin/password | 修改密碼 |

### 用户管理 API（僅超級管理員）

| 方法 | 路徑 | 說明 |
|------|------|------|
| GET | /api/admin/users | 取得用户列表 |
| POST | /api/admin/users | 建立用户 |
| PUT | /api/admin/users/:id | 更新用户（狀態/密碼） |
| DELETE | /api/admin/users/:id | 刪除用户 |



## 使用者權限

| 功能 | 超級管理員(id=1) | 一般使用者 |
|------|-----------------|---------|
| 檢視用户列表 | Y | N |
| 新增用户 | Y | N |
| 停用/啟用用户 | Y | N |
| 重設用户密碼 | Y | N |
| 刪除用户 | Y | N |
| 修改自己密碼 | Y | Y |
| 管理自己的分類/連結 | Y | Y |

## License

MIT
