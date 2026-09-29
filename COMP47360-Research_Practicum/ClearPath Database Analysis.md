

## Purpose

This note summarises the recommended datasets for ClearPath based on the current project discussion. The focus is Manhattan accessibility and healthcare support, especially toilets, ramps, elevators, translation/language support, waiting time, opening hours, and transport time.

The key conclusion is that ClearPath should not rely on one single dataset. NYC Hospitals alone is too small and lightweight, so the project needs a combined dataset strategy.

---

## Recommended Dataset Sources

| Priority | NO. | Dataset / Source                                     | NYC (Total) | Manhattan | Access Type                         | Main Use in ClearPath                                        | Key Information                                                                                                                   |
| -------- | --: | ---------------------------------------------------- | ----------: | --------: | ----------------------------------- | ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| P0       |   1 | User Reports Database                                |           — |         — | Internal app/API                    | Real-time accessibility and crowd status                     | Elevator broken, toilet out of order, crowd, protest, queue, status confirmation, auto-expiry                                     |
| P0       |   2 | NYC Public Restrooms `i7jb-7jku`                     |       1,066 |       358 | NYC Open Data API + CSV             | Main public toilet dataset                                   | Toilet name, location, operator, status, opening hours, seasonal opening, ADA accessibility, changing station, latitude/longitude |
| P1       |   3 | Directory of Toilets in Public Parks `hjae-yuav`     |         616 |       129 | NYC Open Data API + CSV             | Public park toilet supplement                                | Toilet name, borough, location, open year-round, handicap accessible                                                              |
| P0       |   4 | MTA Elevator / Escalator Outages                     |           — |         — | MTA API                             | Real-time transit elevator and escalator status              | Station, equipment ID, outage status, reason, estimated return to service                                                         |
| P0       |   5 | MTA Subway Stations `39hk-dx4f`                      |         496 |       153 | NY Open Data CSV/API                | Subway accessibility and route support                       | Station location, GTFS stop ID, borough, ADA status, ADA notes                                                                    |
| P0       |   6 | OpenStreetMap / Overpass POI (Healthcare)            |         966 |       900 | Overpass API / GeoJSON / CSV export | Main healthcare and POI expansion source                     | Clinics, hospitals, pharmacies, toilets, wheelchair tags, opening hours, phone, website, address                                  |
| P1       |   7 | HRSA Health Care Service Delivery Sites              |         875 |        99 | Dashboard export / GIS REST / CSV   | Community clinics and vulnerable-population healthcare sites | Health centre locations, addresses, coordinates, service delivery sites                                                           |
| P1       |   8 | NYS Health Facility General Information `vn5v-hh5r`  |       5,963 |       454 | NY Health Data API + CSV            | Healthcare facility supplement                               | Facility name, type, address, operator, location                                                                                  |
| P2       |   9 | NYC Hospitals `q6fj-vxf8`                            |          78 |         — | NYC Open Data API + CSV             | Public hospital seed data only                               | Facility name, type, phone, address, location                                                                                     |
| P0       |  10 | Pedestrian Ramp Locations `ufzp-rrqu`                |     217,679 |    23,625 | NYC Open Data API + CSV             | Wheelchair routing and ramp support                          | Ramp location, coordinates, ADA/ramp review information                                                                           |
| P1       |  11 | OpenStreetMap / Overpass POI (Accessibility)         |      15,282 |         — | Overpass API / GeoJSON / CSV export | Accessibility: wheelchair, ramps, entrances                  | barrier, kerb, tactile_paving, ramp, elevator, level, access, entrance                                                            |
| P1       |  12 | AED Inventory `2er2-jqsx`                            |       7,373 |     3,393 | NYC Open Data API + CSV             | Emergency essential service layer                            | AED location, facility name, address, coordinates                                                                                 |
| P2       |  13 | Language Access Secret Shopper `3m3d-zzwn`           |       1,231 |       442 | NYC Open Data API + CSV             | Language service background                                  | Agency language service ratings; not clinic-level language support                                                                |
| P2       |  14 | Population and Languages of LEP Speakers `ajin-gkbp` |       8,024 |     1,632 | NYC Open Data API + CSV             | Language demand analysis                                     | Limited English Proficiency population and language distribution                                                                  |
| P2       |  15 | Yellow Taxi Trip Data                                |  39.2M/year |         — | CSV / API / bulk download           | Busyness and mobility prediction                             | Pickup/dropoff zones, trip time, location demand patterns                                                                         |
| P2       |  16 | Automated Traffic Volume Counts `7ym2-wayt`          |    1M+/year |         — | API / CSV / feeds                   | Transport and congestion context                             | Traffic volume by street segment and time                                                                                         |
| P2       |  17 | Weather / NYC Urban Heat Portal                      |           — |         — | CSV / data download                 | Busyness and health-risk prediction                          | Weather, heat exposure, environmental conditions                                                                                  |
| P3       |  18 | CityMD Locations                                     |          28 |         — | Website / manual CSV                | Urgent care supplement                                       | Location name, address, opening status, possible wait-related indicators                                                          |
| P3       |  19 | Google Places API                                    |           — |         — | API                                 | Clinic and opening-hour enrichment                           | Place name, address, rating, phone, opening hours, coordinates                                                                    |
| P3       |  20 | BestTime.app                                         |           — |         — | Third-party API                     | Crowd / foot traffic prediction                              | Venue busyness and foot traffic estimates                                                                                         |

---

## 需求覆盖分析

| 需求                         | 推荐数据源                                                                     | 覆盖评估                              |
| -------------------------- | ------------------------------------------------------------------------- | --------------------------------- |
| **Toilets**                | NYC Public Restrooms, Park Toilets, User Reports                          | 位置和基础无障碍覆盖强；实时可用性需依赖 User Reports |
| **Ramps**                  | Pedestrian Ramp Locations, OSM wheelchair tags                            | 街道级无障碍路径覆盖良好；室内场地访问有限             |
| **Elevators**              | MTA Elevator/Escalator Outages, MTA Stations, User Reports                | 地铁电梯覆盖强；诊所/建筑电梯需依赖 User Reports   |
| **Healthcare**             | OSM Healthcare POI, HRSA, NYS Health Facility, CityMD, NYC Hospitals      | 中等覆盖；需多源合并，NYC Hospitals 数据量过小    |
| **Translation / Language** | LASS, LEP population data, manual clinic language CSV, Gemini API         | 诊所级语言数据不足；需手动补充                   |
| **Waiting time**           | Taxi data, traffic data, weather, BestTime, user reports                  | 公开数据无实时等待时间；需预测和用户上报              |
| **Opening hours**          | NYC Public Restrooms, OSM `opening_hours`, Google Places, clinic websites | 中等覆盖；医疗设施营业时间需补充                  |
| **Transport time**         | OSRM/OSM, MTA GTFS, MTA elevator data, 511NY, Google Maps API             | 集成路由 API 后覆盖强                     |

---

## NYC Hospitals 数据局限性

`NYC Hospitals q6fj-vxf8` 不应作为主要医疗数据集。

**主要局限：**
- 仅约 78 个位置
- 主要为公立医院或公共卫生设施
- 未覆盖私立诊所、urgent care、walk-in clinic、药房、学生健康中心
- 不含语言支持、费用估算、等待时间、无障碍状态、营业时间详情

**建议用途：**
> 仅作为小型公立医院种子数据集，不作为主要医疗场所数据库

**更好的医疗覆盖应来自：**
- OSM Healthcare POI + HRSA Health Centers + NYS Health Facility
- CityMD 手动补充
- Google Places API（如允许）

---

## MVP 数据栈推荐

### Must-have

1. NYC Public Restrooms `i7jb-7jku`
2. OSM / Overpass healthcare, pharmacy, and toilet POI for Manhattan
3. MTA Elevator / Escalator Outages
4. MTA Subway Stations
5. Pedestrian Ramp Locations
6. User Reports database
7. Manual clinic enrichment CSV

### Nice-to-have

1. HRSA Health Care Service Delivery Sites
2. NYS Health Facility General Information
3. AED Inventory
4. Taxi trip aggregates
5. Traffic Volume Counts
6. Weather / Urban Heat data
7. BestTime.app crowd estimates
8. Google Places API

---

## 最终建议：分层数据策略

```text
Baseline venue data:
NYC Open Data + OSM + HRSA + NYS Health Data

Accessibility data:
Public Restrooms + Pedestrian Ramps + MTA Elevator Status

Real-time layer:
User reports and confirmation prompts

Prediction layer:
Taxi data + traffic + weather + historical patterns + user reports

Enrichment layer:
Manual CSV for clinic language, cost, opening hours, and accessibility notes
```

此方法为项目提供足够的 MVP 覆盖，同时清晰区分公开可用数据与需预测、上报或手动补充的数据。

---

## 数据库信息收集与预测能力

> 数据来源: `Data Source Review_5.26.xlsx` + 本地 CSV/GeoJSON 文件实际内容核查

### 一、可收集的信息

#### 1. 无障碍设施空间分布

| 数据源                           | NYC (Total) | Manhattan | 关键字段                                                                                                     |
| ----------------------------- | ----------- | --------- | -------------------------------------------------------------------------------------------------------- |
| **Pedestrian Ramp Locations** | 217,679     | 23,625    | RampID, CornerID, 坡道街道, Ramp Slope, DWS Conditions, Curb Reveal, 坡道宽度/长度/横坡, Ponding, Obstacles          |
| **MTA Subway Stations**       | 496         | 153       | ADA 总状态, 北/南向 ADA 状态, ADA 备注, GTFS Stop ID, 分区, 线路                                                       |
| **MTA 电梯/扶梯故障**               | —           | —         | 车站, 设备ID, 故障状态, 原因, 预计恢复服务时间                                                                             |
| **OSM 无障碍 POI**               | 15,282      | —         | wheelchair=yes/no (58.2%/13.3%), tactile_paving, ramp, elevator, handrail, level, entrance, surface, lit |

#### 2. 厕所设施

| 数据源                  | NYC (Total) | Manhattan | 关键字段                                                      |
| -------------------- | ----------- | --------- | --------------------------------------------------------- |
| **Public Restrooms** | 1,066       | 358       | 设施名, 位置类型, 运营商, 状态, 开放时间, 季节性开放, ADA 无障碍, 换尿布台, 卫生间类型, 网址 |
| **Park Toilets**     | 616         | 129       | 名称, 位置, 全年开放, 无障碍                                         |
|                      |             |           |                                                           |

#### 3. 医疗资源网络

| 数据源 | NYC (Total) | Manhattan | 关键字段 |
|--------|------------|-----------|----------|
| **NYS Health Facility** | 5,963 | 454 | 设施ID, 名称, 描述, 地址, 城市, 邮编, 电话, 所有权类型, 经纬度 (36列) |
| **HRSA Health Centers** | 875 | 99 | 诊所类型, 编号, 站点名, 地址, 城市, 州, 每周营业时间, 地理坐标, 国会选区 (56列) |
| **OSM Healthcare POI** | 966 | 900 | clinic, doctors, hospital, pharmacy, urgent_care, healthcare:speciality, 地址, 电话, 网站, 营业时间, 轮椅标识 |
| **AED Inventory** | 7,373 | 3,393 | 设备名, 地址, 楼层, 区域, 邮编, NumPersonTrained, AED数量, 经纬度, 位置类型, 最后更新时间 |

#### 4. 语言服务与人口

| 数据源 | NYC (Total) | Manhattan | 关键字段 |
|--------|------------|-----------|----------|
| **LEP Speakers** | 8,024 | 1,632 | 社区区代码, 社区名, 语言, LEP 人口估计值, LEP 百分比, CVALEP 人口/百分比 |
| **LASS Ratings** | 1,231 | 442 | 访问日期, 机构, 站点ID, 服务中心, 地址, 区域, 暗访语言, 解释方式, 等待时间, 评分, 社区委员会, 人口普查区, NTA |

#### 5. 交通与人口流动

| 数据源 | 规模 | 关键字段 |
|--------|------|----------|
| **Yellow Taxi Trip Data** | 每年 3,920 万条 | 上下车时间, 距离, 乘客数, PULocationID/DOLocationID, 车费, 附加费, 拥堵附加费 |
| **Traffic Volume Counts** | 每年 100 万+条 | 行政区域, 年月日时, 15分钟流量, 街道段ID, 几何坐标 |

#### 6. 天气与环境

| 数据源 | 说明 |
|--------|------|
| **NYC Urban Heat Portal** | 静态热脆弱性图层（CSV下载） |
| **NWS API** | 当前/预报天气数据 |

---

### 二、可支持的预测能力

#### 预测 1：轮椅无障碍路径可行性评分

| 维度 | 说明 |
|------|------|
| **输入** | Pedestrian Ramp 条件（坡度、宽度、障碍物、积水）、MTA 电梯故障状态、OSM wheelchair/ramp/elevator 标签 |
| **方法** | 为每条路径段计算通行难度分数，综合坡度>5%、障碍物存在、积水、宽度<90cm 等因素标记为不可通行 |
| **输出** | 从起点到终点的无障碍路径及每段可行性评分 |

#### 预测 2：医疗机构紧急响应覆盖分析

| 维度     | 说明                           |
| ------ | ---------------------------- |
| **输入** | AED 位置（含楼层）、医疗设施位置、人口密度      |
| **方法** | 空间最近邻分析 + 等时圈 (Isochrone) 计算 |
| **输出** | 各坐标点到最近 AED / 最近医院的预计到达时间    |

#### 预测 3：社区语言需求热力图

| 维度 | 说明 |
|------|------|
| **输入** | LEP 人口（按社区区和语言分解）、LASS 暗访评分 |
| **方法** | 按语言分组 LEP 人口，叠加各语言服务评分，识别需求高但服务差的社区 |
| **输出** | 各社区主要语言需求排序 + 语言服务差距指数 |

#### 预测 4：出行时间预测

| 维度 | 说明 |
|------|------|
| **输入** | Yellow Taxi 行程（含时间+距离+持续时间）、交通流量统计、天气 |
| **方法** | 以 Taxi 数据训练旅行时间预测模型（特征：时间、日期、区域、天气），用交通流量作为辅助特征 |
| **输出** | 给定起点、终点、出发时间，预测旅行时间 |

#### 预测 5：设施维护优先级排序

| 维度 | 说明 |
|------|------|
| **输入** | Pedestrian Ramp 条件（DWS条件、坡度、障碍物、积水）、公共厕所状态/营业时间 |
| **方法** | 基于条件恶化程度标记维护优先级（紧急/高/中/低） |
| **输出** | 需要优先维修/清理的设施列表 |

#### 预测 6：厕所可用性动态预测

| 维度     | 说明                                                 |
| ------ | -------------------------------------------------- |
| **输入** | 公共厕所开放时间 + 历史用户上报（User Reports Database, P0）+ 天气数据 |
| **方法** | 用户上报厕所关闭/故障时标记为不可用，结合天气（雨天/高温增加使用需求）预测拥挤度          |
| **输出** | 各厕所当前可用状态 + 预计等待时间                                 |

#### 预测 7：热脆弱性健康风险地图

| 维度 | 说明 |
|------|------|
| **输入** | NYC Urban Heat（温度/热岛强度）+ LEP 人口 + 医疗设施分布 + AED 分布 |
| **方法** | 叠加高温暴露与脆弱人群，计算各区域热风险指数 |
| **输出** | 高风险社区热力图，标记应急资源覆盖缺口 |

---

### 三、能力-数据源映射总表

| 预测/能力 | 核心数据源 | 辅助数据源 |
|-----------|-----------|-----------|
| 轮椅无障碍路径 | Pedestrian Ramp (P0) + MTA ADA (P0) + MTA Outages (P0) | OSM Accessibility (P1) |
| 厕所可用状态 | Public Restrooms (P0) + User Reports (P0) | Park Toilets (P1), Weather (P2) |
| 医疗资源导航 | OSM Healthcare (P0) + HRSA (P1) + NYS Health (P1) | NYC Hospitals (P2), CityMD (P3) |
| AED 应急覆盖 | AED Inventory (P1) | Healthcare POI (P0), NYS Health (P1) |
| 语言需求分析 | LEP Population (P2) + LASS Ratings (P2) | — |
| 出行时间预测 | Yellow Taxi (P2) + Traffic Volume (P2) | Weather (P2) |
| 维护优先级 | Pedestrian Ramp 条件 (P0) | User Reports (P0) |
| 热风险预警 | NYC Urban Heat (P2) + LEP (P2) | AED (P1), Healthcare (P0/P1) |

---

### 四、关键发现

1. **行人坡道数据是最大优势**：217,679 条包含详细物理条件（坡度、宽度、障碍、积水、预警面），是全球公开数据中少见的精细无障碍基础设施数据集，支持真正的轮椅通行能力评估，而非仅是路径几何规划

2. **Real-time layer (P0) 是差异化关键**：User Reports + MTA Outages 提供实时状态，使 ClearPath 区别于静态地图——能响应电梯故障、厕所关闭、人群聚集等动态事件

3. **LEP 数据支撑语言服务独特价值**：1,632 条社区-语言级 LEP 数据 + 442 条 LASS 评分，可构建全市语言需求-服务质量差距分析，这是其他导航应用不具备的能力

4. **AED 数据支持独特急救功能**：含楼层位置、训练人数、设备数量，支持"最近且有人可操作"的 AED 推荐，优于仅显示位置的 AED 地图

5. **Taxi 数据支持旅行时间预测**：3,920 万条/年的行程数据足以训练可靠的旅行时间模型，弥补公开数据中缺乏实时交通预测的短板

---

### 五、待补充的数据缺口

| 缺口 | 影响 | 建议 |
|------|------|------|
| 医疗机构实时拥挤度/排队时间 | 无法预测就诊等待 | User Reports 采集 + BestTime.app (P3) |
| 建筑内部无障碍（电梯、门宽） | 轮椅路径仅覆盖室外 | 用户上报 + OSM 标签补充 |
| 诊所级别语言支持列表 | 无法按语言匹配诊所 | 手动 CSV 补充（已在 MVP 计划中） |
| 卫生间实时占用状态 | 无法判断是否有人排队 | User Reports 采集 |
| MTA 故障历史数据 | 无法训练故障预测模型 | 定期从 API 抓取并存储历史 |
| 出租车/交通数据本地文件 | P2 数据未下载 | 按需下载年度数据 |

---

## API 接口汇总

> 数据来源: 实际 API 调用测试 + 本地文件分析

### 1. Urban Heat NYC Supabase API

| 参数 | 值 |
|------|-----|
| **Base URL** | `https://vcadeeaimofyayyevakl.supabase.co/rest/v1/` |
| **API Key** | 见项目配置文件 |

**数据表：**

| 表名 | 记录数 | 说明 |
|------|--------|------|
| `stations_point` | 74 | 气象站点位置 (GeoJSON) |
| `weather_stations_year` | 124,542 | 每日天气数据 (2013-2023) |
| `stations_summerstat` | 814 | 夏季热统计 |

**Manhattan 气象站点 (7个)：**

| 站点ID | 纬度 | 经度 | 位置 |
|--------|------|------|------|
| BATN6 | 40.7006 | -74.0142 | Battery Park |
| KJRB | 40.7000 | -74.0100 | Lower Manhattan |
| F4321 | 40.7717 | -73.9572 | Central Park |
| D3216 | 40.7763 | -73.9110 | East Side |
| KNYC | 40.7700 | -73.9800 | Central Park (主站) |
| A3655 | 40.8535 | -73.9661 | Harlem |
| D2034 | 40.8775 | -74.0044 | Washington Heights |

### 2. MTA Elevator/Escalator API

| 端点 | URL |
|------|-----|
| **实时故障** | `https://api-endpoint.mta.info/Dataservice/mtagtfsfeeds/nyct%2Fnyct_ene.json` |
| **设备信息** | `https://api-endpoint.mta.info/Dataservice/mtagtfsfeeds/nyct%2Fnyct_ene_equipments.json` |

### 3. NYC Open Data API

| 数据集 | API 端点 |
|--------|----------|
| **Public Restrooms** | `https://data.cityofnewyork.us/resource/i7jb-7jku.json` |
| **Park Toilets** | `https://data.cityofnewyork.us/resource/hjae-yuav.json` |
| **Pedestrian Ramps** | `https://data.cityofnewyork.us/resource/ufzp-rrqu.json` |
| **AED Inventory** | `https://data.cityofnewyork.us/resource/2er2-jqsx.json` |
| **MTA Stations** | `https://data.ny.gov/resource/39hk-dx4f.json` |
| **Traffic Volume** | `https://data.cityofnewyork.us/resource/7ym2-wayt.json` |
| **Yellow Taxi** | `https://data.cityofnewyork.us/resource/kxp8-n2sj.json` |

---

## 数据详细统计

### 医疗设施

| 数据源 | NYC (Total) | Manhattan | 字段数 |
|--------|------------|-----------|--------|
| **OSM Healthcare POI** | 966 | 900 | 177 |
| **HRSA Health Centers** | 875 | 99 | 56 |
| **NYS Health Facility** | 5,963 | 454 | 36 |
| **NYC Hospitals** | 78 | — | ~10 |

**OSM Healthcare 类型分布 (Manhattan)：**

| 类型 | 数量 |
|------|------|
| Pharmacy | 303 |
| Clinic | 201 |
| Dentist | 109 |
| Doctors | 79 |
| Hospital | 31 |
| Alternative | 38 |
| Physiotherapist | 32 |
| Optometrist | 26 |
| 其他 | 81 |

### 卫生间设施

| 数据源                  | NYC (Total) | Manhattan |
| -------------------- | ----------- | --------- |
| **Public Restrooms** | 1,066       | 358       |
| **Park Toilets**     | 616         | 129       |
| **重叠**               | ~211        | ~37       |
| **唯一设施**             | ~1,471      | ~450      |

**Public Restrooms 位置类型分布：**

| 类型 | 数量 |
|------|------|
| Park | 824 |
| Library | 216 |
| POPS | 14 |
| Public Plaza | 7 |
| Transit | 5 |

### 无障碍设施

| 数据源 | NYC (Total) | Manhattan |
|--------|------------|-----------|
| **Pedestrian Ramps** | 217,679 | 23,625 |
| **MTA Subway Stations** | 496 | 153 |
| **OSM Accessibility** | 15,282 | — |
| **AED Inventory** | 7,373 | 3,393 |

### 语言服务

| 数据源 | NYC (Total) | Manhattan |
|--------|------------|-----------|
| **LEP Speakers** | 8,024 | 1,632 |
| **LASS Ratings** | 1,231 | 442 |

---

## 数据集重叠度分析

### 卫生间数据重叠

```
Public Restrooms (358 Manhattan) ∩ Park Toilets (129 Manhattan) = ~37 facilities
重叠率: 28.7% (Parks), 10.3% (Restrooms)
仅在 Parks: 92 个
仅在 Restrooms: 321 个
```

### 医疗设施数据重叠

```
OSM Healthcare (900 Manhattan) ∩ HRSA (99 Manhattan) = 部分重叠
OSM Healthcare (900 Manhattan) ∩ NYS Health (454 Manhattan) = 部分重叠
建议: 以 OSM 为主，HRSA/NYS 为补充
```



---


