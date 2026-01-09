# 🛒 E-commerce Shopping Center 

這是一個基於 **Spring Boot 3** 與 **Vue.js 3** 構建的現代化電商購物系統。專為滿足銀行業高標準的資安與架構要求而設計。

---

## 🌟 核心特色
- **三層式架構**：嚴格區分展示層、業務層與資料層。
- **資料保護**：全面使用 **Stored Procedures** 處理核心邏輯，杜絕 SQL 注入。
- **併發控制**：建立訂單時實作 **Pessimistic Locking (悲觀鎖)**，保證零超賣。
- **安全防護**：整合 **Spring Security + JWT**，並實作 XSS 與 CSP 防禦。
- **現代化設計**：前端使用 Vite + Vue 3，提供流暢的 SPA 使用體驗。

---

## 🛠️ 技術棧
| 類別 | 技術 |
| :--- | :--- |
| **前端** | Vue 3 (Composition API), Vite, Axios |
| **後端** | Spring Boot 3.2.1, Maven |
| **資料庫** | MySQL 8.0 / MariaDB 10.4 |
| **安全性** | Spring Security, JWT, BCrypt |

---

## 🚀 快速啟動

### 1. 環境準備
請確保您的系統已安裝：
- **Java 17+**
- **Maven 3.8+**
- **Node.js 18+**
- **MySQL / MariaDB**

### 2. 資料庫初始化 (關鍵步驟)
請開啟 MySQL 工具（如 phpMyAdmin 或 Workbench），**依序**執行以下腳本：
1. `DB/ddl.sql` - 建立表格結構
2. `DB/stored_procedures.sql` - 建立業務邏輯程序
3. `DB/dml.sql` - 匯入初始商品與帳號資料

### 3. 後端啟動
前往 `ecommerce-backend/src/main/resources/application.yml` 修改資料庫連線資訊：
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/ecommerce_db
    username: root
    password: 您的密碼
```
執行啟動命令：
```bash
cd ecommerce-backend
mvn clean spring-boot:run
```
> [!NOTE]
> 後端預設運行於 Port `8080`。

### 4. 前端啟動
```bash
cd ecommerce-frontend
npm install
npm run dev
```
> [!NOTE]
> 前端預設運行於 Port `5173`。請開啟瀏覽器訪問 `http://localhost:5173`。

---

## 👤 測試帳號
| 角色 | 帳號 (MemberID) | 密碼 | 具備權限 |
| :--- | :--- | :--- | :--- |
| **管理員** | `admin` | `password123` | 商品管理、補貨、所有訂單管理 |
| **一般用戶** | `user1` | `password123` | 購物、查看個人訂單、支付 |

---

## 📁 專案架構
```text
E-commerce/
├── DB/                      # 資料庫層：DDL, DML 與 Stored Procedures
├── ecommerce-backend/       # 應用層：Spring Boot 後端程式碼
│   ├── controller/          # 展示層 (REST API)
│   ├── service/             # 業務層 (Business Logic)
│   ├── repository/          # 資料層 (JDBC / SP Call)
│   └── security/            # 安全層 (JWT / Security Auth)
└── ecommerce-frontend/      # 前端層：Vue.js 實作
    ├── src/views/           # 視圖元件
    └── src/services/        # API 整合
```

---

## �️ 資安與架構實踐
- **Transaction (交易)**：在 `sp_create_order` 中實作事務管理，確保訂單建立與庫存扣減的原子性。
- **RESTful API**：遵循 REST 風格設計語意化 API 介面。
- **DTO 封裝**：嚴格限制資料暴露，確保後端 Entity 不直接對外。
- **XSS 防護**：透過內容安全策略 (CSP) 與前端自動轉義雙重防護。

---
> [!TIP]
> **提示**：若 8080 端口被佔用，請至後端 `application.yml` 修改 `server.port`，並同步更新前端 `vite.config.js` 的代理設定。
