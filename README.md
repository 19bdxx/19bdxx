# 博士论文研究内容总览

> 本页面梳理了各仓库中已完成的研究工作，供博士论文写作参考。

---

## 研究方向：风电功率预测与数据质量提升

### 一、数据质量与预处理（已完成核心工作）

#### 1.1 考虑空气密度的风机数据清洗
**仓库**：[Wind-turbine-data-cleaning-considering-air-density](https://github.com/19bdxx/Wind-turbine-data-cleaning-considering-air-density)

**已完成工作：**
- 基于温度（T）、气压（P）、相对湿度（RH）计算空气密度 ρ，并将其纳入数据清洗流程
- 构建了模块化清洗框架（`stage2_modular`），包含 core / models / pipeline / thresholds 四个子模块
- 实现了基于 KNN 局部算法（`knn_local.py`）的异常值检测，以风速和空气密度为特征维度进行邻域筛选
- 探索了不同切向比例对清洗效果的影响（实验结果存储于 JSON 配置文件）
- 已讨论的论文补充实验目标：空气密度对功率曲线建模的提升效果、跨季节/跨风电场的鲁棒性验证

**论文意义：** 该研究解决了传统清洗方法忽略空气密度差异导致的清洗偏差问题，可支撑"考虑空气密度修正的风机功率曲线建模"研究方向。

---

#### 1.2 重复数据成因分析
**仓库**：[Analysis-of-the-causes-of-duplicate-data-from-wind-turbines](https://github.com/19bdxx/Analysis-of-the-causes-of-duplicate-data-from-wind-turbines)

**已完成工作：**
- 针对风机 SCADA 数据中的重复时间戳问题开展专项分析
- 识别重复数据的产生机制与特征规律

**论文意义：** 为数据清洗前处理阶段提供依据，可作为数据质量章节的重要支撑。

---

#### 1.3 多层级功率数据关系分析
**仓库**：[Fan-Line-Site-Active-Power-Data-Comparison](https://github.com/19bdxx/Fan-Line-Site-Active-Power-Data-Comparison)

**已完成工作：**
- 提取并对比风机有功（Fan）→ 集电线路有功（Line）→ 全站有功（Station）三个层级功率
- 分析了广东多个风电场（峡阳A、峡阳B、峡沙、蕴阳）的功率计算公式与集电线路映射关系
- 研究了层级功率差值（线损、厂用电、计量位置差异）的特征与规律
- 构建了正值保留 / 负值归零两种功率汇总策略

**论文意义：** 揭示了功率层级误差来源，为分层预测框架提供了数据层面的依据。

---

### 二、分层预测框架（核心研究方向）

#### 2.1 风机-集电线路-场站分层预测
**仓库**：[turbine-line-layered-forecast](https://github.com/19bdxx/turbine-line-layered-forecast)、[Line-station-layered-forecast](https://github.com/19bdxx/Line-station-layered-forecast)

**已完成工作：**
- 实现了"全局预测（直接对汇聚功率建模）"与"单机预测+汇聚（Bottom-Up）"两种预测策略
- 使用 Ridge、MLP、LightGBM、XGBoost、LSTM 等多模型对比实验
- 实现了基于线损查表的线损修正逻辑（查表扣除线损后与全局预测比较）
- 构建了滑动窗口样本生成机制（M 步输入、N 步预测），支持多步长实验（15/30/45/60/120/180 分钟）
- 已实现对比分析脚本：输电线功率 vs. 风机集合功率对比、预测结果汇总与分析

**论文意义：** 这是论文最核心的创新点之一——构建了"单机 → 集电线路 → 全站"的分层预测框架，并通过线损修正建立各层级之间的物理约束。

---

### 三、激光雷达与来流风速建模（创新补充方向）

#### 3.1 多普勒风廓线激光雷达数据分析
**仓库**：[Doppler-wind-profile-lidar-data](https://github.com/19bdxx/Doppler-wind-profile-lidar-data)

**已完成工作：**
- 对激光雷达 CSV 数据进行字段解析（Azimuth、Elevation、Distance、RWS、CNR、Spectrum Width 等）
- 分析了雷达扫描模式（VAD/PPI/RHI）并自动识别体扫周期（volume scan）
- 实现了基于 CNR 阈值的质量控制，对比筛选前后统计差异
- 完成了单角度与多角度 RWS 分布对比分析（含热力图、箱线图、方位玫瑰图）
- 梳理了三维风场重建原理（VAD 方法、Dual-Doppler 反演）、转子等效风速（REWS）计算方法

#### 3.2 风机机舱激光雷达数据
**仓库**：[LiDAR-for-wind-turbine-nacelles](https://github.com/19bdxx/LiDAR-for-wind-turbine-nacelles)

**已完成工作：**
- 机舱激光雷达 10 分钟风速数据采集与初步分析
- 数据分析建议报告（含基础统计与进阶分析方向）

**论文意义：** 激光雷达数据可为风电功率预测提供更准确的来流风速信息，是"基于激光雷达来流信息提升风电预测精度"方向的数据基础。

---

### 四、风机运行状态分析

#### 4.1 青州6机组全年运行状态分析
**仓库**：[wind-turbine-status-Qingzhou-6-](https://github.com/19bdxx/wind-turbine-status-Qingzhou-6-)

**已完成工作：**
- 分析了74台风机 2025 年全年运行状态数据（发电/停机/维护等）
- 统计了各机组各状态累计时长、利用率、状态转换频次
- 识别了超长停机事件（>100小时）、频繁启停机组、设备健康度评分

**论文意义：** 运行状态分析为功率预测的样本筛选（如限电剔除、维护期识别）提供依据。

---

### 五、数据集整理

| 仓库 | 数据内容 | 用途 |
|------|----------|------|
| [XYB-wind-turbine-data-for-2025](https://github.com/19bdxx/XYB-wind-turbine-data-for-2025) | XYB风电场2025年度风机数据 | 实验数据集 |
| [HaiLu-Station-Fan-Data-January-November-](https://github.com/19bdxx/HaiLu-Station-Fan-Data-January-November-) | 海鲁场站3台风机2023年1-11月数据 | 风机运行差异分析 |
| [LiDAR-for-wind-turbine-nacelles](https://github.com/19bdxx/LiDAR-for-wind-turbine-nacelles) | 机舱激光雷达10分钟风速数据 | 来流风速建模 |

---

### 六、背景知识积累

**仓库**：[Knowledge](https://github.com/19bdxx/Knowledge)

**已整理内容：**
- **WRF 中尺度气象模型**：模型原理、工作流程、与其他模型的区别、在风电场景中的应用（动力降尺度提供风速预报初始场）

---

## 论文框架建议（基于已有研究）

```
第一章 绪论
  - 研究背景：风电并网消纳、功率预测的重要性
  - 研究现状：数据清洗、功率预测、分层预测综述
  - 研究内容与章节安排

第二章 风电SCADA数据质量分析与清洗
  - 2.1 多层级功率数据结构与层间差异分析（Fan→Line→Station）
  - 2.2 重复数据成因分析与处理
  - 2.3 考虑空气密度的功率曲线数据清洗方法
      → 空气密度计算（T/P/RH）
      → KNN局部异常检测
      → 清洗效果评估

第三章 基于激光雷达的来流风速建模
  - 3.1 多普勒风廓线雷达数据分析
      → 扫描模式识别、CNR质量控制
      → 径向风速（RWS）统计分析
  - 3.2 转子等效风速（REWS）提取
  - 3.3 机舱激光雷达与SCADA风速对比

第四章 风机-集电线路-场站分层功率预测
  - 4.1 分层预测框架设计
  - 4.2 单机预测模型（Ridge / LightGBM / LSTM 等）
  - 4.3 线损修正与自底向上汇聚（Bottom-Up）
  - 4.4 全局预测 vs. 分层预测对比实验

第五章 总结与展望
```

---

## 各仓库研究进度

| 研究方向 | 仓库 | 状态 |
|----------|------|------|
| 数据清洗（空气密度） | Wind-turbine-data-cleaning-considering-air-density | ✅ 核心代码完成，待补充实验 |
| 分层预测框架 | turbine-line-layered-forecast / Line-station-layered-forecast | ✅ 框架完成，待优化与评估 |
| 多层级功率关系分析 | Fan-Line-Site-Active-Power-Data-Comparison | ✅ 分析完成 |
| 激光雷达数据分析 | Doppler-wind-profile-lidar-data | 🔄 进行中 |
| 机舱激光雷达 | LiDAR-for-wind-turbine-nacelles | 🔄 数据采集完成 |
| 风机状态分析 | wind-turbine-status-Qingzhou-6- | ✅ 完成 |
| 重复数据分析 | Analysis-of-the-causes-of-duplicate-data-from-wind-turbines | 🔄 进行中 |
| 数据集整理 | XYB-wind-turbine-data-for-2025 / HaiLu-Station-Fan-Data-January-November- | ✅ 数据就绪 |
| 背景知识整理 | Knowledge | 🔄 持续更新 |
