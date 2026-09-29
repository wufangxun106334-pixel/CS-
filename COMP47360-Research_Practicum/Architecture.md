# ClearPath 系统架构

> 更新日期: 2026-07-17
> 基于项目实际代码结构

---

## 一、项目概述

**定位**：面向曼哈顿行动不便人群的**实时无障碍与医疗服务导航平台**（Waze-style）。

**平台形态**：React Native Mobile (Expo) + React Web (Vite) + Flask REST API + ML 预测模块 + Gemini AI 多语言 Chatbot

---

## 二、整体架构

```
┌──────────────────────────────────────────────────────────────┐
│                        前端 (Frontend)                        │
│  ┌─────────────────────┐   ┌─────────────────────────────┐  │
│  │ Mobile (Expo SDK 56) │   │ Web (React 19 + Vite 8)     │  │
│  │ React Native 0.85    │   │ React Router 7              │  │
│  │ Expo Router          │   │ MapLibre GL 5               │  │
│  │ react-native-maps    │   │ Recharts 3                  │  │
│  │ i18next 6语言        │   │ CSS                          │  │
│  └──────────┬──────────┘   └──────────────┬──────────────┘  │
└─────────────┼─────────────────────────────┼─────────────────┘
              │          HTTP/REST           │
              ▼                              ▼
┌──────────────────────────────────────────────────────────────┐
│                     后端 (Flask API)                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ app.py → 13 Blueprints (Flask)                       │   │
│  │ auth/venues/reports/insights/chatbot/medical/        │   │
│  │ routes/realtime/integrations/translate/user/         │   │
│  │ health/app_state                                      │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │ JWT 认证  │ │ 响应缓存  │ │ SOS 缓冲 │ │ Celery   │      │
│  │ (PyJWT)  │ │ (Redis)  │ │ (deque)  │ │ (Tasks)  │      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
└─────────────┬──────────────────┬───────────────────────────┘
              │                  │
     ┌────────▼────────┐  ┌─────▼──────┐
     │   MySQL 8.4     │  │  Redis 7   │
     │ (Tablespace加密) │  │ (Broker +  │
     │  10+3 表 Schema │  │  Blacklist │
     └─────────────────┘  │  + Cache)  │
                          └────────────┘
```

---

## 三、前端架构

### 3.1 Mobile 端 (`frontend/mobile/`)

| 层级 | 技术 | 说明 |
|------|------|------|
| 框架 | React Native 0.85 + Expo SDK 56 | 跨平台 iOS/Android |
| 路由 | Expo Router (~56.2) | 文件系统路由 |
| 地图 | react-native-maps 1.27 | 原生地图渲染 |
| 国际化 | i18next + react-i18next | 6 种语言 (de/en/es/fr/it/zh) |
| 动画 | react-native-reanimated 4.3 | 高性能动画 |
| 安全存储 | expo-secure-store | Token 持久化 |
| 定位 | expo-location | GPS 定位 |
| 语音 | expo-speech | TTS 语音输出 |

**目录结构**:
```
src/
├── app/                    # Expo Router 文件系统路由
│   ├── _layout.tsx         #   根 Stack Navigator
│   ├── index.tsx           #   入口
│   ├── welcome.tsx         #   欢迎页
│   ├── auth-gateway.tsx    #   认证网关（登录/游客）
│   ├── login.tsx           #   登录
│   ├── (tabs)/             #   底部 Tab 导航
│   │   ├── _layout.tsx     #   Tab 配置
│   │   ├── map.tsx         #   地图（核心）
│   │   ├── assistant.tsx   #   AI 助手
│   │   ├── show-staff.tsx  #   Show Staff 卡片
│   │   ├── profile.tsx     #   个人中心
│   │   └── more.tsx        #   更多
│   ├── edit-profile.tsx
│   ├── medical-id.tsx      #   医疗 ID
│   ├── settings.tsx
│   ├── sos.tsx             #   SOS 紧急
│   ├── location.tsx
│   ├── language.tsx
│   ├── legal.tsx
│   └── profile-guest.tsx
├── components/             # 共享 UI 组件
│   ├── VenueBottomSheet.tsx
│   ├── ReportBottomSheet.tsx
│   ├── FilterModal.tsx
│   ├── MapSearchBar.tsx
│   ├── RouteOptionsModal.tsx
│   ├── FloatingActionButtons.tsx
│   ├── CategoryChips.tsx
│   └── ui/
├── services/               # API 服务层
│   ├── api.ts              #   HTTP 客户端
│   ├── authService.ts      #   认证
│   ├── location.ts         #   定位
│   ├── medical.ts          #   医疗档案
│   └── profile.ts          #   用户资料
├── hooks/                  # useColorScheme, useTheme
├── constants/              # colours, typography, theme
├── types/                  # TypeScript 类型
└── locales/                # 6 语言翻译文件
```

### 3.2 Web 端 (`frontend/web/`)

| 层级 | 技术 | 说明 |
|------|------|------|
| 构建 | Vite 8 | 开发服务器 + 构建 |
| 框架 | React 19.2 | SPA |
| 路由 | React Router DOM 7.17 | 客户端路由 |
| 地图 | MapLibre GL 5.24 | 矢量地图渲染 |
| 图表 | Recharts 3.8 | Insights 仪表盘 |
| 样式 | 纯 CSS (页面级 .css + tokens.css) | 无 CSS-in-JS |
| 测试 | Jest 30 + Testing Library | 单元测试 |

**目录结构**:
```
src/
├── main.jsx                # 入口: createRoot → <App/>
├── App.jsx                 # BrowserRouter + Header + Routes
├── pages/                  # 10 个页面
│   ├── Login.jsx
│   ├── LiveHelpMap.jsx     #   实时地图（核心）
│   ├── InsightsDashboard.jsx  # 数据洞察
│   ├── Profile.jsx
│   ├── EditProfile.jsx
│   ├── MedicalCard.jsx
│   ├── Favourites.jsx
│   ├── Settings.jsx
│   ├── About.jsx
│   └── UserGuide.jsx
├── components/
│   └── BusynessChart.jsx
├── services/
│   ├── apiClient.js        #   fetch 封装 (JWT + API Key)
│   ├── authService.js      #   登录/注册
│   ├── tokenStorage.js     #   localStorage Token
│   └── *Api.js             #   各模块 API
├── styles/tokens.css
└── utils/sessionCleanup.js
```

**路由结构**:
```
/                → Login
/map             → LiveHelpMap
/insights        → InsightsDashboard
/about           → About
/guide           → UserGuide
/profile         → Profile
/profile/edit    → EditProfile
/medical-card    → MedicalCard
/settings        → Settings
/favourites      → Favourites
```

**API 代理**: Vite dev server 将 `/api` 代理到 `http://127.0.0.1:5000`

---

## 四、后端架构 (`backend/`)

### 4.1 技术栈

| 类别 | 技术 | 版本 |
|------|------|------|
| Web 框架 | Flask | 3.0 |
| 语言 | Python | 3.11+ |
| 数据库 | MySQL 8.4 + PyMySQL + DBUtils 连接池 |
| 缓存/消息 | Redis 7 (Celery broker + JWT 黑名单 + 响应缓存) |
| 任务队列 | Celery 5.4 (Beat Scheduler) |
| 认证 | PyJWT (HS256) + Werkzeug 密码哈希 |
| 加密 | Fernet (AES-128-CBC + HMAC) 双层加密 |
| 包管理 | Poetry |
| 外部 API | Gemini, Google Maps Directions, BestTime |

### 4.2 目录结构

```
backend/src/
├── main.py                 # 启动入口
├── app.py                  # Flask 工厂: create_app() → 13 Blueprint
├── settings.py             # @dataclass Settings (.env 加载)
│
├── db.py                   # PyMySQL 连接池 (DBUtils PooledDB)
│   ├── get_connection()    #   获取连接
│   ├── db_cursor()         #   只读游标
│   ├── db_transaction()    #   事务 (auto commit/rollback)
│   └── verify_tablespace_encryption()  # Tier 1 加密校检
│
├── auth.py                 # 认证 (src 级共享)
│   ├── require_api_key     #   X-API-Key 装饰器
│   ├── require_bearer_auth #   JWT Bearer 装饰器
│   ├── issue_access_token  #   HS256 签发 (1h TTL)
│   └── SESSIONS            #   内存会话表
│
├── token_blacklist.py      # Redis JWT 黑名单（登出撤销）
├── response_cache.py       # Redis 响应缓存（fail-open）
├── sos_buffer.py           # SOS 事件 deque 缓冲
│
├── gemini_client.py        # Gemini REST 客户端
│   ├── embed_text()        #   gemini-embedding-001
│   └── generate_structured_reply()  # gemini-2.5-flash
│
├── google_maps_client.py   # Directions API 客户端
├── medical_crypto.py       # Fernet 医疗档案加解密
├── district_leaderboard.py # 区域聚合 (纯函数)
├── mock_data.py            # 离线/降级 Mock
│
├── celery_app.py           # Celery 配置 (Redis + Beat)
├── tasks.py                # expire_stale_reports (每5分钟)
│
└── api/                    # 13 个 Blueprint
    ├── health.py           #   GET  /api/v1/health
    ├── auth.py             #   POST /api/v1/auth/{register,login,logout}
    ├── user.py             #   GET/PUT/DELETE /api/v1/user/profile
    ├── venues.py           #   GET  /api/v1/venues (双语badge, 繁忙度)
    ├── routes.py           #   POST /api/v1/routes/directions
    ├── reports.py          #   POST/GET /api/v1/reports (众包+确认)
    ├── insights.py         #   GET  /api/v1/insights (区域聚合, 排行榜)
    ├── chatbot.py          #   POST /api/v1/chatbot (RAG + Gemini)
    ├── medical.py          #   GET/PUT /api/v1/medical/profile (加密)
    ├── realtime.py         #   SSE  /api/v1/realtime/stream
    ├── integrations.py     #   POST /api/v1/integrations/sos
    ├── translate.py        #   POST /api/v1/translate
    └── app_state.py        #   GET  /api/v1/app/state
```

### 4.3 核心设计

**认证双通道**: 每个请求经 `X-API-Key`（应用级） + `Bearer JWT`（用户级）两层验证。

**数据库连接池**: DBUtils `PooledDB`，懒加载，兼容无 MySQL 环境（CI/Mock）。

**双层加密（医疗数据）**:
- Tier 1: MySQL InnoDB tablespace 加密（磁盘级）→ `verify_tablespace_encryption()` 启动时 fails-closed 校验
- Tier 2: Fernet 对称加密（应用级）→ 密钥仅存环境变量，数据库只存密文

**SOS 实时流**: `integrations.py` 接收 SOS → `sos_buffer.py` (deque) → `realtime.py` SSE 推送客户端

**降级策略**: 所有外部依赖 (Gemini, Google Maps, Redis) 采用 fail-open 模式，不可用时回退 Mock。

**后台任务**: Celery Beat 每 5 分钟执行 `expire_stale_reports`，将 2 小时未确认的报告标记为 expired。

---

## 五、Data+ML 架构 (`Data+ML/`)

### 5.1 技术栈

| 类别 | 技术 |
|------|------|
| 数据库 | MySQL 8.4 |
| ETL | Python (pymysql, pandas, numpy) |
| ML | scikit-learn 1.4 |
| 弱标签 | Google Popular Times (SerpAPI) |
| 向量化 | Gemini Embedding API |
| 调度 | Celery Beat (Telemetry Worker) |
| 可视化 | matplotlib + Jupyter Notebook |

### 5.2 数据生命周期 (4 阶段)

```
Phase 1: ETL          Phase 2: DQR         Phase 3: ML
6.2-6.5_DB            6.8-6.12_DB          6.22-6.27 → 7.13-7.18
│                     │                    │
│ 9 外部数据源         │ 数据质量审查        │ 模型训练 + 预测
│ → venues + profiles │ → 清洗 + GPS修复   │ → busyness_scores
└─────────────────────┴────────────────────┴──────────────┘
                                      │
                                      ▼
                              Phase 4: Serving
                              后端 API + 前端
```

### 5.3 Phase 1: ETL

**位置**: `Data+ML/test/6.2-6.5_DB/clearpath_db/`

```
clearpath_db/
├── db.py              # PyMySQL 连接
├── config.py          # 连接配置
├── schema.py          # DDL 建表
├── migrations.py      # Schema 迁移
├── etl/
│   ├── restrooms.py   #   公共卫生间 (NYC OpenData x2)
│   ├── healthcare.py  #   医疗机构 (OSM + NYS Health)
│   ├── aed.py         #   AED (NYC OpenData)
│   └── ramps.py       #   坡道 (NYC OpenData)
├── sources.py         # 数据源追踪
├── venue_language.py  # 语言标签
├── dedup.py           # 去重
├── validation.py      # 校验
└── weather.py         # 天气
```

**9 个保留数据源**:

| 数据源 | 目标表 |
|--------|--------|
| NYC Public Restrooms | venues + restroom_profiles |
| Directory of Toilets in Parks | venues + restroom_profiles |
| OSM Healthcare POI | venues + healthcare_profiles |
| NYS Health Facility Info | venues + healthcare_profiles |
| NYC AED Inventory | venues + emergency_assets |
| Pedestrian Ramp Locations | pedestrian_ramps |
| Google Maps API | external_context_cache |
| Weather API | external_context_cache |
| User Reports (内部) | user_reports + report_confirmations |

### 5.4 Phase 2: DQR

**位置**: `Data+ML/test/6.8-6.12_DB/dqr/`

数据质量审查管道: `dqr_plan.md` → `dqr_checks.py` → `dqr_cleaning_pipeline.ipynb` → `fix_restroom_gps.py`

### 5.5 Phase 3: ML 预测

**迭代**: `6.28-7.3/` (v1) → `7.6-7.11/` (v2 实验) → `7.13-7.18/` (当前权威)

**当前权威代码** (`7.13-7.18/src/`):

```
├── serpapi_popular_times_snapshot.py   # Popular Times 采集
├── serpapi_popular_times_labels.py     # Weak label 生成
├── forecast_v2_pattern.py              # 12h 预测模型
├── score_utils.py                      # 评分工具
├── db_utils.py                         # DB 工具
├── analytics_kpi_formulas.py           # KPI 公式
├── district_aggregation.py             # 区域聚合
├── fastest_hubs.py                     # 最快枢纽排序
└── rag_knowledge_base.py               # RAG 知识库
```

**ML 设计原则**:
- 弱监督学习: Google Popular Times 作为 proxy label
- 输出: predicted_score (0-100) + predicted_level (quiet/moderate/busy/no_data)
- 特征: 星期、小时、天气、历史模式
- 模型: scikit-learn

### 5.6 Phase 4: Telemetry

**位置**: `Data+ML/test/6.15-6.20/src/`

实时遥测系统（Docker profile 隔离启动）:
- `busyness_ingestion.py` — 繁忙度 ETL
- `live_capacity_telemetry.py` — 实时容量采集
- `telemetry_worker.py` — Worker 进程
- `venue_coverage.py` — 空间覆盖分析

### 5.7 数据库 Schema (10 核心表 + 3 扩展)

```
venues (主表)
├── venue_source_links     # 数据溯源
├── restroom_profiles      # 卫生间详情
├── healthcare_profiles    # 医疗机构详情
├── emergency_assets       # AED 设备
├── pedestrian_ramps       # 坡道
├── user_reports           # 用户报告
│   └── report_confirmations  # 报告确认
├── busyness_scores        # 繁忙度
├── external_context_cache # 外部 API 缓存
├── venue_accessibility    # 无障碍设施
├── venue_language         # 语言支持
└── venue_warnings         # 警告信息
```

---

## 六、基础设施 (Docker Compose)

```yaml
services:
  mysql:        # MySQL 8.4 (端口 3306)
  redis:        # Redis 7 (端口 6379)
  phpmyadmin:   # 管理面板 (端口 8080, 开发用)
  telemetry:    # 实时遥测采集 (profile 隔离, 按需启动)
```

**环境变量** (`.env`):
```
API_KEY, BESTTIME_API_KEY, GOOGLE_MAPS_API_KEY, GEMINI_API_KEY
JWT_SECRET, DB_HOST/PORT/USER/PASSWORD/NAME
DB_ENCRYPTION_CHECK, REDIS_URL, MEDICAL_PROFILE_ENCRYPTION_KEY
```

---

## 七、核心功能模块

| 模块 | 功能 | 实现 |
|------|------|------|
| **Find** | 搜索诊所/厕所/AED，语言+无障碍筛选 | `venues.py` + `routes.py` + 前端地图 |
| **Report** | 一键上报设施问题，2h 自动过期 | `reports.py` + `tasks.py` (Celery) |
| **Predict** | ML 拥挤度预测 + 12h 预报 | `insights.py` + ML Pipeline |
| **Assist** | Gemini+RAG 多语言 Chatbot | `chatbot.py` + `gemini_client.py` |
| **SOS** | 紧急求助 + 实时 SSE 推送 | `integrations.py` + `sos_buffer.py` + `realtime.py` |
| **Medical** | 加密医疗档案 + Show Staff 卡片 | `medical.py` + `medical_crypto.py` |
| **Route** | Google Maps 多模式导航 | `routes.py` + `google_maps_client.py` |
