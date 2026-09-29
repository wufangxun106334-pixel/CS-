

### 项目现状

- `busyness_scores` 表已定义（score 0-100, level, forecast_1h/4h/8h），但 **0 行数据**

- 无 ML 模型代码，无历史人流量数据

- BestTime API 有工作 notebook（test_besttime_heatmap.ipynb），但配额已耗尽且未集成到 DB

  

### 外部数据源可用性

  

**BestTime API**（首选）

- 返回 0-100 busyness score，按小时粒度

- 5 个端点已验证：forecasts, venues/filter, forecasts/week, week/raw, day

- 问题：配额耗尽，需重新申请

  

**Google Popular Times / SerpAPI**

- Google 没有面向本项目的官方 Popular Times API。SerpAPI 的 Google Maps Place 接口可在部分地点返回 `popular_times`。

- **覆盖率优先于数值解释**：必须先按 `place_id`/`data_cid` 匹配本项目 venue；只有 Google 本身展示 Popular Times 的地点才有结果，不能假定覆盖全部 Manhattan venue。

- 返回的历史小时曲线含 **数值** `busyness_score`（0–100）；部分地点还带 **等级文字** 的当前状态，例如 `Less busy than usual`。两者均为相对繁忙度，不是人数。

- 它提供历史模式和可选当前状态，**不提供未来时段预测**。若用于 `forecast_1h/4h/8h`，须由本项目模型结合历史模式、天气、交通和时间特征自行生成预测。

- 官方接口说明：<https://serpapi.com/maps-place-results>

  

### Manhattan 免费公开数据源（可批量查询）

  

| 数据源 | 粒度 | Manhattan 覆盖 | 批量查询 |

|--------|------|---------------|---------|

| TLC Taxi Zones | 0.3–1.5 km² | ~60-70 zones | ✅ BigQuery |

| MTA Subway | 站点级（~400m 缓冲区） | ~130 站点 | ✅ SODA API |

| Citi Bike | 站点级 | ~1700 站点 | ✅ CSV/API |

| Census Tracts | 0.05–0.2 km² | ~280 个 | ✅ |

  

**推荐组合方案**：TLC Taxi + MTA Subway + Citi Bike → 按 zone/tract 聚合 → busyness 分数

  

### 最小网格粒度

- 免费数据：~0.1–0.5 km²（census tract 到 taxi zone）

- 商业数据（SafeGraph）：~0.02 km²（census block group）或 100m×100m 自定义网格

  

### 相关文件

- BestTime notebook: `Data+ML/test/6.1_heatmap/test_besttime_heatmap.ipynb`

- Schema: `docker/mysql/init/001_clearpath_schema.sql`（busyness_scores 表）

- 项目计划: `Data+ML/plan/6.2_CC/6.2_tasks.md`（Phase 5 ML Integration）

---

## 公开 API 补充调研（2026-06-15）

### 结论

存在公开 API 可查询人流或人流代理数据，但目前没有一个免费来源能够直接提供 Manhattan
所有 venue 的实时客流量。

最值得新增的是 NYC DOT 在 2025 年底发布的自动自行车与行人计数器 API。它提供真实传感器
计数、15 分钟粒度，并持续更新；但当前活跃 pedestrian 传感器只有少量点位，不能直接覆盖
Manhattan 全部场所。

推荐架构：

1. NYC pedestrian sensors 作为真实人流标签和模型校准数据。
2. MTA、Citi Bike、TLC Taxi、道路交通作为全域时空特征。
3. 天气、节假日、活动作为外生特征。
4. BestTime 仅作为付费增强或离线验证，不作为免费基础设施依赖。

### 数据源对比

| 数据源                               | 覆盖率 / 覆盖边界                                           | 原始数据与时间性                          | 输出数据类型                                                                   | 是否直接给出未来预测               | 适用性                         |
| --------------------------------- | ---------------------------------------------------- | --------------------------------- | ------------------------------------------------------------------------ | ------------------------ | --------------------------- |
| NYC Bicycle and Pedestrian Counts | **低**：少量传感器点位，不能覆盖 Manhattan 全部 venue                | 约 15 分钟的实测人流                      | **数值**：`counts`                                                          | 否；可作真实标签或特征              | 最直接的真实人流标签，但必须以空间插值/特征方式使用  |
| NYC Pedestrian Sensor Locations   | **仅元数据**：只覆盖已部署传感器                                   | 随设备更新                             | **非预测**：坐标、方向等元数据                                                        | 否                        | 与计数表 join，确定观测覆盖范围          |
| MTA 2025 OD Ridership             | **中**：地铁站及其周边，不覆盖远离站点的 venue                         | 历史估算客流，站点/小时/OD                   | **数值**：`estimated_average_ridership`                                     | 否；可建历史小时 baseline        | 区域级特征，不应视为 venue 当前客流       |
| MTA GTFS-Realtime                 | **中**：MTA 线路/站点；不提供站内人数                              | 实时车辆、到站和告警                        | **数值**：到站数、延误分钟等派生特征                                                     | 否                        | 只构造服务强度特征                   |
| Citi Bike GBFS                    | **中低**：有 Citi Bike 站的区域                              | 约 60 秒的车辆/空桩状态                    | **数值**：车辆数、空桩数、周转量                                                       | 否                        | 可推断局部流动强度，不等于行人量            |
| NYC TLC Taxi                      | **中**：Taxi Zone；空间较粗、发布有延迟                           | 历史上下车记录                           | **数值**：pickup/dropoff 次数                                                 | 否                        | 区域活动强度训练特征                  |
| NYC SODA Traffic                  | **中**：有监测的道路 segment                                 | 数据集批次更新                           | **数值**：车辆流量/速度字段                                                         | 否                        | 车辆代理指标，不等于 venue 人流         |
| TomTom Traffic Flow               | **中高**：可访问的道路路段；不覆盖步行空间                              | 约 1 分钟实时道路流                       | **数值**：速度、流量、拥堵                                                          | 否                        | 道路拥堵特征，不是行人数据               |
| BestTime                          | **受商业目录限制**：仅其已收录且可查询的 venue                         | 小时级历史模式，部分地点有 live                | **数值**：0–100 busyness；可映射为等级                                             | 是，提供 venue 繁忙度预测/部分 live | 最接近目标输出，但受额度和覆盖限制           |
| Google Popular Times              | **不保证**：仅 Google 有 Popular Times 的单个 venue           | 历史按小时模式，部分地点有 live status         | **数值 + 等级文字**：相对 0–100 与繁忙描述                                             | 否；不是未来预测                 | 无官方公开 API；直接抓取有稳定性和合规风险     |
| SerpAPI Google Maps Place         | **不保证**：依赖 Google Maps 地点匹配，且仅部分地点返回 `popular_times` | 对单地点的 Google Maps 结果；可带当前 live 状态 | **数值 + 等级文字**：每小时 `busyness_score` 0–100，及如 “Less busy than usual” 的文字状态 | 否；提供历史模式/可选当前状态，不是预测模型   | 合规与成本需单独评估；适合作为授权后的外部特征或弱标签 |
| NYC DOT 公共交通摄像头                   | **低**：摄像头视野内，不可外推到全城 venue                           | 近实时图像                             | **数值（需 CV 派生）**：人/车计数                                                    | 否                        | 可研究 CV 计数，但有隐私、许可和维护成本      |

### 1. NYC 自动行人计数 API

这是目前找到的最直接免费人流 API。

- Counts 数据集：`ct66-47at`
- Sensor 元数据：`6up2-gnw8`
- 字段：`sensor_id`、`travelmode`、`direction`、`timestamp`、`granularity`、`counts`
- 时间粒度：`PT15M`
- 时区：数据集说明为 EST
- 2026-06-15 核查结果：
  - pedestrian 记录约 1,424,580 条
  - pedestrian 最新时间为 `2026-06-15T01:15:00`
  - 活跃 pedestrian 设备集中在 High Bridge、Emmons Ave、Concrete Plant Park 等少量点位

查询示例：

```text
https://data.cityofnewyork.us/resource/ct66-47at.json
  ?$select=sensor_id,timestamp,sum(counts)%20as%20pedestrian_count
  &$where=travelmode='pedestrian'
  &$group=sensor_id,timestamp
  &$order=timestamp%20DESC
  &$limit=5000
```

传感器位置：

```text
https://data.cityofnewyork.us/resource/6up2-gnw8.json?$limit=5000
```

限制：

- 点位非常有限，不能把最近传感器数值直接分配给 Manhattan 所有 venue。
- 可能受天气、通信、设备故障或破坏影响，必须检测缺失、延迟和异常值。
- 同一位置可能有 combined flow 和 pedestrian-only flow，聚合前必须避免重复计数。

官方来源：

- https://data.cityofnewyork.us/resource/ct66-47at.json
- https://data.cityofnewyork.us/resource/6up2-gnw8.json
- https://data.cityofnewyork.us/Transportation/Bicycle-and-Pedestrian-Counts/ct66-47at

### 2. MTA 小时级客流 API

MTA 2025 Origin-Destination Ridership Estimate 已公开：

- Dataset ID：`y2qv-fytt`
- 字段包含 origin/destination station、经纬度、weekday、hour 和
  `estimated_average_ridership`
- 适合生成站点周边小时级历史模式
- 不是实时监控数据，不能直接作为“当前客流”

```text
https://data.ny.gov/resource/y2qv-fytt.json?$limit=50000
```

MTA GTFS-Realtime 提供列车位置、到站预测和服务告警，但不提供可靠的站内实时乘客数量。
可以将单位时间到站列车数、延误和服务中断作为模型特征。

官方来源：

- https://data.ny.gov/Transportation/MTA-Subway-Origin-Destination-Ridership-Estimate-2025/y2qv-fytt
- https://www.mta.info/developers
- https://api.mta.info/

### 3. Citi Bike 实时 API

Citi Bike GBFS `station_status` 是无需 API key 的实时 JSON：

- 约 2,400 个站点
- 常见 TTL 为 60 秒
- 字段包括 `num_bikes_available`、`num_docks_available`、`last_reported`

```text
https://gbfs.lyft.com/gbfs/1.1/bkn/en/station_status.json
https://gbfs.lyft.com/gbfs/1.1/bkn/en/station_information.json
```

单次库存不是人流量。应保存时间序列，通过 bikes/docks 的变化量、短时间 turnover 和附近站点
净流入/净流出构造移动强度特征。

### 4. BestTime

BestTime 是最接近项目 `busyness_score 0-100` 目标的现成 API：

- 使用匿名手机信号生成每周小时级预测。
- 支持部分 venue 的 live foot traffic。
- forecast、live 和 venue search 使用 private API key 与 credits。
- 已有 forecast 可通过 public query key 查询，但不能视为无限免费数据源。
- live 数据并非所有 venue 都有。

官方文档：

- https://documentation.besttime.app/
- https://besttime.app/pricing

### 5. 摄像头与计算机视觉

NYC 有公开道路摄像头画面，可理论上使用 YOLO 等模型做行人检测和计数。但不建议作为当前
版本主数据源：

- 公开画面不等于稳定、受支持的数据 API。
- 摄像头角度、遮挡、夜间、天气会造成严重偏差。
- 需要确认图像保存、自动抓取和衍生分析的许可条款。
- 应只保存聚合计数，不保存人脸或可识别图像，并设置短期缓存。
- 摄像头通常覆盖路口，不代表室内 venue occupancy。

因此摄像头方案适合作为后续研究实验，不应替代官方传感器和交通代理数据。

### 推荐落地优先级

#### P0：接入真实 pedestrian 标签

新增 ingestion：

```text
fetch_pedestrian_counts()
fetch_pedestrian_sensors()
aggregate_pedestrian_hourly()
```

保存原始 15 分钟计数，并聚合为 sensor-hour。此数据用于校准和评估，不直接广播到所有 venue。

#### P1：构建多源区域特征

按 H3、census tract 或 300–500 米网格统一聚合：

```text
pedestrian_count
subway_ridership_baseline
citibike_turnover
taxi_pickups_dropoffs
vehicle_traffic
weather
event_features
```

输出应标记：

```text
score_source = observed | estimated | fallback
confidence = 0.0-1.0
observed_at
model_version
```

#### P2：venue 级预测

区域分数不能无差别分配给同 district 的所有 venue。建议加入：

- venue category
- opening hours
- historical BestTime pattern（如果有授权）
- 与地铁、计数器、Citi Bike、活动场馆的距离
- weekday、hour、weather、holiday

最终输出 venue-level busyness，并保留置信度和数据新鲜度。

### 最终判断

- **有公开真实人流 API**：NYC `ct66-47at`，但覆盖不足。
- **有公开实时移动代理 API**：Citi Bike GBFS、MTA GTFS-Realtime。
- **有公开历史客流 API**：MTA OD/hourly ridership、TLC Taxi。
- **没有免费、稳定、覆盖 Manhattan 全 venue 的实时 occupancy API**。
- 当前项目最佳路线不是寻找单一替代 BestTime 的 API，而是训练一个多源、带置信度的
  venue busyness estimation 模型。
