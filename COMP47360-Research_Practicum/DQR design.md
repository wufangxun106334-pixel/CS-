# DQR 设计文档

> 更新日期：2026-07-16
> 模块路径：`Data+ML/test/dqr/`
> Notebook：`Data+ML/test/6.8-6.12_DB/dqr_cleaning_pipeline.ipynb`

---

## 一、模块架构（精简后 3 个 .py）

```
dqr/
├── __init__.py        # 模块标识
├── dqr_checks.py      # 质量检查 + 画像 + 清洗 + 评分 + 改进建议（合并原 analysis + cleaning）
├── dqr_utils.py       # 数据库连接 + 地理工具 + 外部 API（合并原 external_ingestion）
└── dqr_io.py          # SQL 查询 + CSV 导出 + 审计报告
```

### 模块职责

| 模块 | 职责 | 关键函数 |
|------|------|---------|
| `dqr_checks.py` | 质量检查 + 数据画像 + 清洗 + ML 评估 | `check_*`, `compute_*`, `clean_venues`, `build_*` |
| `dqr_utils.py` | 工具函数 | `get_conn`, `is_manhattan`, `validate_coords`, `fetch_*` |
| `dqr_io.py` | 数据读写 | `query_table`, `load_dqr_tables`, `export_dqr_artifacts` |

---

## 二、Notebook 执行顺序（28 个单元）

Notebook 以 9 个 DQR 流程阶段组织，共 28 个 Markdown / Code 单元；其中新增 Part 8（Busyness Data Overview）。下列编号表示流程阶段，不再表示实际单元数量。

```
Cell 1: Configuration     → 导入模块、路径配置
Cell 2: Executive Summary → 从 MySQL 加载 9 张表
Cell 3: Data Profiling    → 列级画像 + 行级质量评分
Cell 4: Quality Checks    → completeness / accuracy / integrity 检查（FK orphan 检查待接入）
Cell 5: Anomaly Detection → 坐标异常 + GPS 重复检测
Cell 6: DQ Score          → borough 修复 + 六维度评分 + 雷达图
Cell 7: Cleaning          → 清洗 venues + 散点图
Cell 8: External Data     → 交通 + 天气 API（实现中与 Cleaning 同一阶段）
Cell 9: Action Items      → 改进建议 + CSV 导出 + ML 评估
Part 8: Busyness Overview → 拥挤度分布、分区统计、12 小时预测曲线与 forecast JSON 预览
```

---

## 三、Cell 详细说明

### Cell 1: Configuration

```python
# 路径配置（绝对路径，因为 kernel cwd 是项目根目录）
PROJECT_ROOT = Path('/Users/alex/.../Group6_Summer-Project')
TEST_ROOT = PROJECT_ROOT / 'Data+ML' / 'test'
OUTPUT_DIR = TEST_ROOT / '6.8-6.12_DB' / 'tests' / 'output'
sys.path.insert(0, str(TEST_ROOT))

# DQR_TABLES: 9 张表
DQR_TABLES = ('venues', 'restroom_profiles', 'healthcare_profiles', 
              'emergency_assets', 'pedestrian_ramps', 'venue_source_links',
              'busyness_scores', 'external_context_cache', 'user_reports')

# 导入 3 个模块的函数
from dqr.dqr_checks import ...
from dqr.dqr_utils import ...
from dqr.dqr_io import ...
```

### Cell 2: Executive Summary

```python
conn = get_conn()                    # 创建 MySQL 连接
data = load_dqr_tables(conn, DQR_TABLES)  # 批量加载 9 张表
conn.close()                         # 关闭连接

df_venues = data['venues']           # 主表快捷引用
total_rows = 所有表总行数
tables_loaded = 7                    # 审计报告的当前统计口径；与 9 张声明表/8 张画像表并不一致，需统一
completeness = df_venues 整体填充率
venue_types = {'emergencyasset': 3279, 'healthcare': 1086, 'restroom': 473}
```

### Cell 3: Data Profiling

```python
all_profiles = build_all_profiles(data)      # 当前导出含 8 张表、98 个字段；user_reports 未出现在画像中
record_analysis = build_record_analysis(df_venues)  # 行级质量评分（0-1）
```

### Cell 4: Quality Checks

```python
completeness_result = check_completeness(data)   # 各表核心字段填充率
accuracy_result = check_accuracy(df_venues)       # 坐标范围 + venue_id 格式 + district 枚举
integrity_result = check_database_integrity(df_venues)  # district 必填检查
# 待接入：fk_result = check_fk_orphans(df_venues, data)  # 外键引用完整性
coord_valid_mask = accuracy_result['_coord_valid_mask']  # 坐标有效性掩码（供后续使用）
```

### Cell 5: Anomaly Detection

```python
anomaly_df = detect_coordinate_anomalies(data)     # 坐标越界 + GPS(0,0) 检测
gps_duplicates_df = detect_gps_duplicates(data, threshold_m=10)  # GPS 重复（10m 阈值）
```

### Cell 6: DQ Score & Rating

```python
# 1. 修复 borough（评分前！）
if is_manhattan(lat, lng) and borough != 'Manhattan':
    borough = 'Manhattan'  # 自动修正 983 条

# 2. 计算六维度评分
scores = compute_dq_scores(df_venues, data, anomaly_df, gps_duplicates_df, coord_valid_mask)
total_score, grade = compute_total_score(scores)
# 评分表：
# Completeness  0.25  → 字段填充率
# Accuracy      0.25  → 坐标/ID 格式
# Consistency   0.15  → borough 一致性（修复后 ~99.6%）
# Uniqueness    0.15  → venue_id 唯一 + GPS 重复（10m 阈值）
# Timeliness    0.10  → 数据跨度 <1年
# Validity      0.10  → venue_type 枚举合规

# 3. 生成图表
# 雷达图 + Missing rate heatmap
```

### Cell 7: Cleaning Pipeline

```python
venues_clean = clean_venues(df_venues, coord_valid_mask, quality_scores)
# 清洗步骤：
# 1. 去空行（dropna how='all'）
# 2. 过滤异常坐标（GPS(0,0) + 超出曼哈顿范围）
# 3. 修复 borough（坐标在曼哈顿 → 强制 Manhattan）
# 4. 附加质量分数（0-1）
```

### Cell 8: External Data

```python
traffic_clean = clean_traffic(fetch_traffic_hourly(year=2025))  # NYC SODA API → 792 行
weather_clean = fetch_and_clean_weather(raise_errors=True)      # NWS API → 1 行
```

### Cell 9: Action Items & Export

```python
actions_df = build_action_items(df_venues, data, scores)  # 自动生成改进建议
export_dqr_artifacts(OUTPUT_DIR, ...)   # 导出 7 个 CSV
ml = assess_ml_usability(venues_clean, traffic_clean, weather_clean, scores, grade)
```

---

## 四、输出文件

| 文件 | 行数 | 内容 |
|------|------|------|
| `venues_clean.csv` | 4,838 | 清洗后场馆（去空行 + 过滤坐标 + borough 修复 + quality_score） |
| `traffic_hourly.csv` | 792 | 2025 年曼哈顿小时级交通流量 |
| `weather_current.csv` | 1 | 当前天气（温度、描述、风险等级） |
| `dqr_field_summary.csv` | 98 | 当前导出的 8 张表列级画像（dtype、缺失率、唯一值、min/max/mean） |
| `dqr_record_analysis.csv` | 4,838 | 行级质量评分（每条记录的 0-1 分数） |
| `dqr_outliers.csv` | 183 | 当前导出的坐标异常记录；审计报告的 `anomalies_detected=315` 使用另一统计口径，待统一 |
| `dqr_gps_duplicates.csv` | 701 | GPS 重复对（<10m） |
| `dqr_report.csv` | 9 | DQR 审计摘要（当前总分 80.4/100，Good） |
| `dqr_dimension_scores.png` 等 3 张图 | — | 六维评分、缺失率热图、场馆散点图 |
| 医疗分类分布 CSV | 2 | `healthcare_category` 与 `facility_type` 分布 |

> 统计口径说明：`DQR_TABLES` 声明 9 张表；当前 `dqr_field_summary.csv` 包含 8 张表（不含 `user_reports`）；`dqr_report.csv` 写入 `tables_analyzed=7`。后续应统一加载、画像和审计三处的计数定义。

---

## 五、关键函数速查

### dqr_checks.py（质量检查 + 画像 + 清洗）

| 函数 | 输入 | 输出 | 用途 |
|------|------|------|------|
| `check_completeness(data)` | 表数据 dict | `{passed, score, _dataframe}` | 核心字段填充率 |
| `check_accuracy(venues_df)` | venues DataFrame | `{passed, score, _coord_valid_mask}` | 坐标/ID/district 校验 |
| `check_database_integrity(venues_df)` | venues DataFrame | `{passed, score}` | district 必填 + GPS(0,0) 诊断 |
| `check_fk_orphans(venues_df, data)` | venues + 表数据 | `{passed, score}` | 外键引用完整性 |
| `compute_dq_scores(...)` | venues + data + 异常 + 重复 + 掩码 | `dict[str, float]` | 六维度评分 |
| `compute_total_score(scores)` | 评分 dict | `(total_score, grade)` | 加权总分 + 等级 |
| `column_profile(df, table_name)` | 单表 DataFrame | DataFrame | 列级画像 |
| `build_all_profiles(data)` | 表数据 dict | DataFrame | 全表画像 |
| `build_record_analysis(venues_df)` | venues DataFrame | DataFrame | 行级评分 |
| `detect_coordinate_anomalies(data)` | 表数据 dict | DataFrame | 坐标异常 |
| `detect_gps_duplicates(dfs_dict, threshold_m)` | 表数据 dict | DataFrame | GPS 重复检测 |
| `clean_venues(venues_df, coord_valid_mask, quality_scores)` | venues + 掩码 + 评分 | DataFrame | 清洗 venues |
| `build_action_items(venues_df, data, scores)` | venues + data + 评分 | DataFrame | 改进建议 |
| `assess_ml_usability(...)` | 清洗后数据 + 外部数据 + 评分 | dict | ML 可用性评估 |

### dqr_utils.py（工具函数）

| 函数 | 输入 | 输出 | 用途 |
|------|------|------|------|
| `get_conn()` | 无 | pymysql 连接 | 创建 MySQL 连接 |
| `is_manhattan(lat, lng)` | 经纬度 | bool | 判断是否在曼哈顿 |
| `gps_to_district(lat, lng)` | 经纬度 | str | 判断所属区域 |
| `validate_coords(lat, lng, bbox)` | 经纬度 | (bool, str) | 坐标校验 |
| `haversine_m(lat1, lng1, lat2, lng2)` | 两组经纬度 | float | 两点距离（米） |
| `fetch_traffic_hourly(year, boro)` | 年份、区域 | DataFrame | NYC 交通 API |
| `clean_traffic(traffic_df)` | 交通原始数据 | DataFrame | 清洗交通数据 |
| `fetch_and_clean_weather(raise_errors)` | 无 | DataFrame | 天气 API |

### dqr_io.py（数据读写）

| 函数 | 输入 | 输出 | 用途 |
|------|------|------|------|
| `query_table(table, conn, extra)` | 表名、连接 | DataFrame | 单表 SQL 查询 |
| `load_dqr_tables(conn, table_names)` | 连接、表名列表 | dict[str, DataFrame] | 批量加载 |
| `export_dqr_artifacts(output_dir, **dfs)` | 目录、DataFrame | None | 批量导出 CSV |
| `build_audit_report(...)` | 评分、数据量、输出目录 | DataFrame | 审计报告 |

> 当前 Notebook 已导入 `build_audit_report`，但未直接调用；`dqr_report.csv` 的生成逻辑应在实现或文档中明确。`check_fk_orphans` 同样尚未接入质量检查阶段。

---

## 六、数据清洗逻辑

### clean_venues 清洗步骤

```
原始 venues (4838 行)
    │
    ├─ Step 1: dropna(how='all') → 去全空行
    │
    ├─ Step 2: validate_coords() → 过滤 GPS(0,0) + 超出曼哈顿范围
    │   移除 124 条（公园卫生间，GPS 占位符）
    │
    ├─ Step 3: is_manhattan() → 修复 borough
    │   983 条 borough 从 NULL/New York/Park/Library → Manhattan
    │
    └─ Step 4: 附加 quality_score（0-1）
        基于 6 个核心字段的填充率

venues_clean (4838 行，无删除行)
```

### 修复的问题

| 问题 | 修复方式 | 影响 |
|------|---------|------|
| `emergency_asset` vs `emergencyasset` | 代码匹配数据库实际值 | Validity 32% → 100% |
| borough 脏数据（New York/Park/Library） | 根据坐标自动修正 | Consistency 79% → 99.6% |
| GPS 重复阈值 30m 过宽 | 改为 10m | Uniqueness 50% → 85.5% |
| dqr_utils.py 缺少 `import pandas` | 补充导入 | 外部数据正常加载 |

---

## 七、六维度评分权重

```python
DQ_WEIGHTS = {
    'Completeness': 0.25,  # 字段填充率
    'Accuracy':     0.25,  # 坐标/ID/district 格式
    'Consistency':  0.15,  # borough 一致性
    'Uniqueness':   0.15,  # venue_id 唯一 + GPS 重复
    'Timeliness':   0.10,  # 数据时效（<1年=95分）
    'Validity':     0.10,  # venue_type 枚举合规
}
```

### 等级判定

| 总分 | 等级 |
|------|------|
| ≥ 90 | Excellent |
| ≥ 80 | Good |
| ≥ 70 | Fair |
| < 70 | Poor |

---

## 八、外部数据 API

### 交通数据（NYC SODA API）

```
端点: https://data.cityofnewyork.us/resource/7ym2-wayt.json
参数: boro=Manhattan, yr=2025
返回: 792 行（28 路段 × 24 小时）
字段: segmentid, street, avg_vol, peak_vol, busyness_level, hour
```

### 天气数据（NWS API）

```
端点: https://api.weather.gov/gridpoints/OKX/33,37/forecast
返回: 1 行（当前天气）
字段: condition, temperature_c, risk_level
```

---

## 九、ML 可用性评估

```python
assess_ml_usability(venues_clean, traffic_clean, weather_clean, scores, grade)
# 返回:
# {
#   'venues_count': 4838,        # 清洗后场馆数
#   'venue_types': 3,            # 场馆类型数
#   'coord_complete_pct': 100.0, # 坐标完整率（清洗后）
#   'district_count': 4,         # 区域数
#   'quality_score_mean': 1.0,   # 平均质量分
#   'traffic_rows': 792,         # 交通数据行数
#   'traffic_segments': 28,      # 交通路段数
#   'weather_condition': 'Sunny',# 当前天气
#   'dq_score': {...},           # 六维度评分
#   'dq_grade': 'Good',          # DQ 等级
# }
```

---

## 十、实现与验证状态（2026-07-16）

- Notebook 覆盖设计中的 9 个 DQR 流程阶段，并额外包含 Busyness 分析（Part 8）。
- 保存的模块测试结果为 `12 passed`（2026-06-13）；另有 pipeline 和 Notebook 结构测试，需在下一次完整验证时一并运行并更新记录。
- 当前关键待办：接入 `check_fk_orphans()`，统一 `tables_analyzed`、异常数和 GPS 重复数的统计口径。
