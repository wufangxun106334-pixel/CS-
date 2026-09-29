# ClearPath Sprint 3-4 当前问题敞口

> 更新日期: 2026-07-08  
> 目标: 本文件只保留仍需处理的问题、阻塞、验收门槛和下一步动作。已解决/已交付内容已迁移到 `docs/memory/project-issues.md`。

---

## P0 / P1 当前开放问题

| # | 问题 | 严重度 | 当前影响 | Owner | 下一步 |
|---|------|:------:|----------|-------|--------|
| O1 | D3.1 真实 telemetry feed / production schedule 未确认 | P0 | live capacity 仍停在脚本/测试能力；生产刷新链路、调度、失败重试、audit log 未冻结 | Data + Ops | 确认真实 feed source；部署 scheduler/worker；跑 `run_live_telemetry.py --execute` 到 DB；验证 realtime endpoint |
| O2 | D3.1 telemetry `--execute` 依赖表缺失 | P0 | Docker MySQL 冒烟时 `telemetry_audit_log` 与 `venue_source_links` 不存在，阻塞实库写入 | Data + Backend | 确认 `007_telemetry_audit_log.sql` 是否被 initdb 执行；补齐 `venue_source_links` DDL/seed；重跑 telemetry execute |
| O4 | D4.1 dashboard mock fallback freeze 口径未定 | P0 | `/api/v1/insights` 已 DB-backed，但 mock fallback 是否允许进入 Sprint 4 demo 尚未裁决 | Data + Backend + Frontend | 决定 `mock_data.INSIGHTS_DASHBOARD` 是否只限 dev/DB unavailable；response 必须显式标记 `data_mode` / fallback |
| O5 | D4.2 forecast-v2 production claim 仍需质量门禁 | P0 | V2 主链路可运行，但若覆盖不足、指标异常或外部特征未 execute，不能宣称 fully validated | Data | 按本文 “forecast-v2 部署前质量门禁” 执行；通过前只能标记 smoke-test / partial rollout |
| O6 | D4.3 Backend Gemini `/api/v1/chatbot` route 未实现/未验收 | P0 | Data RAG 已交付，但无稳定 Backend 调用层会阻塞 chatbot E2E 与隐私边界测试 | Backend | 实现 request/response schema、retrieval、Gemini call、timeout/retry、rate limit、error handling |
| O7 | D4.4 Backend chatbot route tests 缺失 | P0 | Data 侧 forbidden-source regression 已有，但 Backend prompt assembly/retrieval SQL 未被测试锁住 | Backend | 新增 `backend/tests/test_chatbot*.py`，覆盖 medical-source 禁访、medical-advice refusal、timeout mock、language preservation |
| O8 | `medical_profiles` vs `user_medical_profiles` 命名一致性 | P0 | DDL/OpenAPI/contract tests 与 cascade-delete 口径可能不一致 | Backend + Data | 统一表名/接口文档/测试 fixture；确认 forbidden-source allowlist 使用最终命名 |
| O9 | `settings.py` 残留 `medical_profile_encryption_key` | P2 | 旧 Fernet 方案遗留，可能误导开发者；不阻塞功能 | Backend | 若已接受 JSON 列 + tablespace encryption，删除 setting 和 `.env.example` 注释 |
| O10 | `test_medical_profile.py` 空 profile 返回值断言待确认 | P2 | 测试可能与实际 API `null` 响应不一致 | Backend | 确认空档案 contract；同步测试和 OpenAPI example |
| O11 | M2.6 Post-Registration Intercept PRD 对齐 | P2 | `profile-guest.tsx` 存在，但未确认是否符合 PRD | Frontend | 对照 PRD 验证注册后拦截、guest/profile flow、错误状态 |
| O12 | 后端尚未 Docker 化 | P1 | `docker-compose.yml` 只有 MySQL/Redis/phpMyAdmin；无 backend service/Dockerfile | Backend + Ops | 新增 `backend/Dockerfile`、compose `backend` service、容器内 DB host=`mysql`、Redis host=`redis` |
| O13 | 2026-07-08 Medical profile 双实现冲突 | P0 | `user.py` 与 `api/medical.py` 同时注册 `GET/PUT/DELETE /api/v1/user/medical-profile`；Flask 先命中 `user.py`，但该实现查询不存在的 `encrypted_payload` 列 | Backend | 保留一个实现；建议保留 `api/medical.py` 列式 schema 版本，删除/迁移 `user.py` medical-profile routes；补 route contract test |
| O14 | 2026-07-08 report_categories 部署补种风险 | P0 | `user_reports.issue_type` 有 FK；若旧 MySQL volume 未执行 `006_seed_report_categories.sql`，DB-backed report submit 会失败并静默 fallback mock | Backend + Ops | 将 006/007 纳入部署 SOP 和 repeatable migration；启动/部署前检查 `report_categories` 行数；禁止吞掉 DB insert 异常 |
| O15 | 2026-07-08 report issue types 6 vs 9 不一致 | P1 | DB seed 有 9 类，但 `reports.py` 只允许 6 类；`long_waiting_time`、`ramp_blocked`、`closed_early` 无法提交/无稳定 label | Backend | 同步 `ALLOWED_REPORT_TYPES`、`ISSUE_TYPE_LABELS`、OpenAPI、mock data 和 seed dictionary |
| O16 | 2026-07-08 venue_type contract 与 seed/mock 不一致 | P1 | `venues.py` 允许 `urgent_care/mental_health/shelter`，但 DB/mock 使用 `restroom/emergencyasset` 等；`?venue_type=restroom` 会 400 | Backend + Data + Frontend | 冻结 venue type enum；同步 schema seed、backend validation、OpenAPI 和 client filter |
| O17 | 2026-07-08 `GET /reports` DB/mock response shape 不一致 | P1 | DB path 返回 `_format_report()` lean shape；mock fallback 返回 raw `REPORTS` richer shape，同一 endpoint contract 随 DB 可用性变化 | Backend | mock fallback 也走 `_format_report()`，或移除生产 fallback；补 DB unavailable contract test |
| O18 | 2026-07-08 `GET /venues` busyness 与 richer fields contract 未冻结 | P1 | DB-backed list 不 inline busyness；busyness 需调用 `/venues/{id}/busyness` / forecast。language/accessibility/warnings 部分 inline，但 mock shape 更丰富 | Backend + Frontend + Data | 明确 list/detail/busyness 分层 contract；前端按 separate busyness call 实现；统一 DB/mock response shape |
| O19 | 2026-07-08 auth mock / refresh token placeholder | P2 | `mock_data.AUTH_USERS` 重复定义；login 返回 placeholder `refresh_token`，无 refresh endpoint，access token 1 小时后无法续期 | Backend | 删除重复 mock 定义；决定 initial deployment 是否支持 refresh；若不支持，从 response/OpenAPI 移除或标记 placeholder |
| O20 | 2026-07-08 favourites 仍是全局内存 placeholder | P0 | `GET/POST/DELETE /api/v1/user/favourites` 使用 API key，无 `g.user_id`；全局 `FAVOURITES` 重启丢失；新增 favourite 固定 `fav_003` 和 fake timestamp | Backend + Frontend | 改为 `require_bearer_auth`；使用 `user_favorite_venues` 表按 `g.user_id` 读写；DB 生成 `created_at`；前端将操作 gate behind login |

---

## Sprint 3 剩余执行方案

### O1 / O2: 实时遥测生产链路

- **当前已知**: telemetry runner dry-run 与测试已通过，但生产 feed、调度、实库写入和 audit table 仍是开放项。
- **必须补齐**:
  - `telemetry_audit_log` DDL 在 Docker init 中可重复执行。
  - `venue_source_links` 存在并能把 source venue id 映射到 `venues.venue_id`。
  - `run_live_telemetry.py --execute` 写入 `busyness_scores`。
  - `/api/v1/realtime/map-updates` 或 venue busyness endpoint 能读到 `model_version='live-telemetry-v1'` 的新数据。
- **验收**:
  - 真实或模拟生产 payload 至少 1 批 execute 成功。
  - 记录 received / normalized / rejected / inserted / failed mapping。
  - DB 和 endpoint smoke test 通过。

## Sprint 4 剩余执行方案

### O4: Dashboard mock fallback freeze

- **当前已知**: `/api/v1/insights` DB-backed aggregation 已落地，但 mock fallback 仍存在。
- **需要裁决**:
  - Demo 是否允许 mock fallback。
  - fallback 是否只允许 DB unavailable 或 explicit dev mode。
  - response 是否必须暴露 `data_mode`, `formula_version`, `fallback_reason`。
- **验收**:
  - 前端只依赖 frozen schema。
  - 空数据返回 `no_data` 或 partial payload，不返回静默 mock 常量。
  - OpenAPI example 与实现一致。

### O5: forecast-v2 部署前质量门禁

`forecast-v2` 能部署不等于 production ML fully validated。以下门禁未全部通过前，只能标记为 smoke-test / partial rollout。

- **Venue 覆盖门禁**:
  - `--live-db` 训练/预测必须显式记录覆盖数，不接受隐式默认。
  - 命令需提供 `--max-venues` 或 `--n-synth-venues` 等控制参数。
  - 输出必须记录 `training_venue_count`, `prediction_venue_count`, `forecast_row_count`, `rows_per_venue`。
- **Leakage / 过拟合门禁**:
  - 若 validation/test `R2 >= 0.99` 或 `MAE <= 0.1`，暂停 production claim。
  - 检查 `label_score` 不在 feature columns。
  - 检查 `predicted_score` / 未来 `busyness_forecasts` 未作为训练特征。
  - rolling window 只能读取 `forecast_for` 之前的数据。
  - time split 必须满足 train max timestamp < val min timestamp < test min timestamp。
  - 同一 `(venue_id, forecast_for)` 不得跨 split 重复。
- **标签分布门禁**:
  - 训练、验证、测试和 prediction curve 都必须输出 quiet/moderate/busy 分布。
  - 若没有 busy 样本或 busy 占比过低，不得宣称可可靠预测 high-crowd 场景。
- **真实外部特征门禁**:
  - `external_feature_ingest.py --dry-run` 只证明 API 可达，不证明模型使用真实外部特征。
  - 部署前必须对目标源执行 `--execute` 写入 `external_context_cache`。
  - 重新跑 feature pipeline、model training、writer 后，feature audit 应显示 `weather_source=open_meteo`, `gbfs_source=lyft_gbfs_2.3`, `mta_source=gtfs_rt` 或明确 `partial`。
- **推荐执行顺序**:
  1. `external_feature_ingest.py --source weather --execute`
  2. `external_feature_ingest.py --source holiday --execute`
  3. `external_feature_ingest.py --source gbfs --execute`
  4. `external_feature_ingest.py --source mta_gtfs_rt --execute`
  5. `forecast_v2_feature_pipeline.py --live-db --max-venues <target>`
  6. `forecast_v2_model.py --features output/forecast_v2_training_features.csv --pred-features output/forecast_v2_prediction_features.csv`
  7. `forecast_v2_writer.py --dry-run --csv output/prediction_curve_v2.csv --model-version forecast-v2`
  8. `forecast_v2_writer.py --execute --csv output/prediction_curve_v2.csv --model-version forecast-v2`
  9. `GET /api/v1/venues/{venue_id}/busyness/forecast` smoke test，确认 12 points、`model_version=forecast-v2`、`external_feature_status.status=ok`。
- **阻塞规则**:
  - 阻塞部署: leakage audit 未通过、真实 DB writer `--execute` 失败、endpoint 不能返回 12h curve。
  - 可降级发布: venue 覆盖不足、busy 标签缺失、部分外部源缺失，但必须标记 partial rollout / known limitation。

### O6 / O7: Backend Chatbot + RAG 隐私测试

- **Data 边界**:
  - 允许检索: `venues`, `venue_accessibility`, `venue_language`, `venue_warnings`, `busyness_scores`, `busyness_forecasts`, `user_reports`。
  - 禁止检索/嵌入/prompt: `medical_profiles`, `user_medical_profiles`, allergies, conditions, medications, emergency contacts, preferences, favorites, notification settings。
- **Backend route 要求**:
  - `POST /api/v1/chatbot`
  - API key auth / rate limit
  - request validation
  - retrieval allowlist
  - Gemini timeout / retry / error fallback
  - response language preservation
  - medical-advice refusal
- **测试要求**:
  - Gemini 调用必须 mock。
  - forbidden-source regression 失败即阻塞合并。
  - 英文/中文/法语 query 至少覆盖 venue lookup、accessibility、forecast、forbidden medical query。

### O12: 后端 Docker 化

- **当前状态**: `docker-compose.yml` 只有 `mysql`, `redis`, `phpmyadmin`；后端仍是本地 Flask/Python 运行。
- **需要新增**:
  - `backend/Dockerfile`
  - `.dockerignore`
  - compose `backend` service
  - 容器内环境变量: `DB_HOST=mysql`, `REDIS_URL=redis://redis:6379/0`, `PORT=5000`
  - `depends_on` mysql/redis healthcheck
  - `5000:5000` port mapping
- **验收**:
  - `docker compose up --build backend`
  - `/api/v1/health` 返回 200
  - forecast endpoint 能连接 compose MySQL
  - 不依赖宿主机 `127.0.0.1` DB

### O13-O20: 2026-07-08 Backend/API contract audit

> 日期戳: 2026-07-08  
> 目标: 关闭后端 contract、DB schema、mock fallback 和前端调用假设之间的不一致。

| Issue | Owner | 对应文件 | 验收口径 |
|---|---|---|---|
| O13 Medical profile 双实现冲突 | Backend | `backend/src/app.py`; `backend/src/api/user.py`; `backend/src/api/medical.py`; `docker/mysql/init/004_medical_profiles.sql`; `backend/tests/test_medical_profile.py` | 同一 method/path 只注册一个 endpoint；实现字段与 `medical_profiles` DDL 一致；GET/PUT/DELETE contract test 通过 |
| O14 report_categories 补种风险 | Backend + Ops | `docker/mysql/init/001_clearpath_schema.sql`; `docker/mysql/init/006_seed_report_categories.sql`; `docker/mysql/init/007_telemetry_audit_log.sql`; deployment SOP | 新老 DB volume 都能补齐 `report_categories`；部署 smoke check 包含 `SELECT COUNT(*) FROM report_categories`；DB insert 失败不得静默伪成功 |
| O15 report issue types 6 vs 9 | Backend | `backend/src/api/reports.py`; `backend/src/mock_data.py`; `docker/mysql/init/006_seed_report_categories.sql`; OpenAPI spec | 9 个 category 均可提交并返回 label；mock/report seed 不含 unknown label |
| O16 venue_type contract 不一致 | Backend + Data + Frontend | `backend/src/api/venues.py`; `backend/src/mock_data.py`; `docker/mysql/init/001_clearpath_schema.sql`; `docker/mysql/init/005_seed_venues.sql`; OpenAPI spec | `VALID_VENUE_TYPES` 与 DB enum/seed/client filter 一致；`restroom`/AED 口径明确且测试覆盖 |
| O17 reports response shape 不一致 | Backend | `backend/src/api/reports.py`; `backend/src/mock_data.py`; `backend/tests/test_reports.py` | DB available/unavailable 时 `GET /reports` 返回同一 item schema；fallback 行为显式标记 |
| O18 venues list vs busyness 分层 contract | Backend + Frontend + Data | `backend/src/api/venues.py`; `backend/src/mock_data.py`; `docker/mysql/init/001_clearpath_schema.sql`; `backend/tests/test_venues.py`; `backend/tests/test_live_capacity_api.py` | 文档确认 `GET /venues` 不 inline busyness；mobile 对 busyness 使用独立 endpoint；DB/mock venue item shape 对齐 |
| O19 auth mock / refresh token placeholder | Backend | `backend/src/api/auth.py`; `backend/src/auth.py`; `backend/src/mock_data.py`; `backend/tests/test_auth_login.py`; `backend/tests/test_jwt_signature.py` | 要么实现 refresh flow，要么从 contract 中移除/标记 refresh token；`AUTH_USERS` 只定义一次 |
| O20 favourites 用户级持久化 | Backend + Frontend | `backend/src/api/user.py`; `backend/src/mock_data.py`; `docker/mysql/init/001_clearpath_schema.sql`; `backend/tests/test_user_profile.py` 或新增 `backend/tests/test_user_favourites.py` | favourites routes 使用 Bearer auth；按 `g.user_id` 读写 `user_favorite_venues`；新增/删除不影响其他用户；重启不丢 |

### 2026-07-08 执行 SOP

#### 0. 执行前统一动作

1. 先只做读，不改代码。
2. 对照下面三类真源：
   - SQL DDL / seed: `docker/mysql/init/*.sql`
   - 后端实现: `backend/src/api/*.py`
   - 测试与 mock: `backend/tests/*.py`, `backend/src/mock_data.py`
3. 每个问题都要同时确认这四件事：
   - 路由有没有重复定义
   - 数据库表结构是不是同一个 contract
   - mock fallback 有没有和 DB 路径对齐
   - 测试有没有锁住当前行为
4. 修复顺序固定为：
   - 先 contract
   - 再 schema/seed
   - 再后端实现
   - 再测试
   - 最后更新文档
5. 每修一个问题，都要跑一次最小验证，不要攒到最后。

#### 1. Medical profile

1. 只保留一个实现。优先保留 `backend/src/api/medical.py` 的列式 schema 版本。
2. 删除或停用 `backend/src/api/user.py` 里的 `GET/PUT/DELETE /api/v1/user/medical-profile`。
3. 确认 `docker/mysql/init/004_medical_profiles.sql` 里的字段仍然是唯一真源。
4. 把测试改成只命中一个 route，并确认返回字段包含 `medications` 和 `emergency_contacts`。
5. 验收标准：
   - `GET /api/v1/user/medical-profile` 只走一个 endpoint
   - `PUT` 读写字段与 DDL 一致
   - `DELETE` 删除后重新 `GET` 返回默认档案

#### 2. Reports

1. 先把 `report_categories` seed 补种路径收口。
2. 检查部署 SOP 是否真的执行到 `006_seed_report_categories.sql`。
3. 把 `backend/src/api/reports.py` 的允许类型和 label 字典补到 9 个。
4. 让 mock data 和 DB 返回统一的 report shape。
5. 验收标准：
   - DB 模式下提交成功后，`user_reports` 有真实记录
   - 9 个 issue type 都能提交并显示正确 label
   - `GET /reports` 不再因为 DB 可用性变化而换 shape

#### 3. Venues

1. 冻结 `VALID_VENUE_TYPES`，以 DB seed 和前端 filter 为准。
2. 确认 `venues` 列表返回的是哪些 inline 字段，哪些必须走独立 busyness 接口。
3. 把 mock 和 DB 的 venue item shape 统一。
4. 验收标准：
   - `?venue_type=` 不会再出现 seed 里已有值却被 400 的情况
   - 前端知道 busyness 要单独取，不会假设 list 已经带齐
   - language/accessibility/warning 字段在 DB 和 mock 中语义一致

#### 4. Auth / Favourites

1. `refresh_token` 先定策略：真做，还是删掉占位。
2. `AUTH_USERS` 只保留一份定义。
3. favourites 改成 Bearer auth + `user_favorite_venues`。
4. 不再使用全局 `FAVOURITES` 内存列表。
5. 验收标准：
   - 收藏只属于当前登录用户
   - 重启服务不会丢数据
   - `created_at` 由数据库生成，不靠模板常量

#### 5. 变更提交顺序

1. 先提交 schema / seed 变更。
2. 再提交后端 route / service 变更。
3. 再补测试。
4. 最后改前端调用或文档。
5. 每个提交都要能单独解释“解决了哪个 contract 不一致”。

---

## 当前跨阶段依赖

- **O1/O2 → O5**: 没有生产 telemetry 累积，forecast-v2 只能继续 tabular baseline；不得重启 ARIMA/LSTM claim。
- **O4 → Dashboard freeze**: mock fallback 口径不定会影响前端是否能按 frozen schema 验收。
- **O6 → O7**: chatbot route 不实现，Backend forbidden-source regression 无法落地。
- **O8 → RAG 隐私边界**: 医疗表命名不一致会影响 allowlist/denylist 测试可靠性。
- **O12 → 部署验收**: 后端未容器化时，Docker 环境只能验证 DB/Redis，不能验证完整 backend stack。
- **O13 → Backend user data contract**: medical-profile route 冲突未解决前，前端不能可靠调用医疗档案 API。
- **O14/O15/O17 → Reports contract freeze**: report category seed、allowed type、fallback shape 必须一起修，否则前端会遇到提交成功但不入库或渲染字段漂移。
- **O16/O18 → Venues/mobile contract freeze**: venue type enum 与 busyness 分层 contract 未冻结前，mobile filter 和 venue list rendering 不能视为稳定。
- **O20 → Login-gated favourites**: favourites 必须后端改为 Bearer + DB-backed 后，前端才能提供真实用户收藏能力。
