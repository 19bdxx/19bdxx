# 博士论文整体框架

> **论文核心主题**：考虑物理约束的风电场精细化功率预测方法研究
>
> **统一逻辑主线**：风电功率预测的三大精度瓶颈分别来自**数据质量**、**风机间空间耦合（尾流效应）**和**场站层级电气约束（线损）**。本论文围绕这三个维度，从数据 → 单机建模 → 场站聚合，构建一套完整的、物理约束驱动的风电场功率精细化预测体系。

---

## 论文结构总览

```
┌─────────────────────────────────────────────────────────┐
│  第一章  绪论                                            │
│  研究背景 · 文献综述 · 研究内容与框架                    │
└─────────────────────┬───────────────────────────────────┘
                      │ 问题提出
                      ▼
┌─────────────────────────────────────────────────────────┐
│  第二章  数据质量提升                                    │
│  多层级功率结构分析 · 重复数据处理                       │
│  考虑空气密度的功率曲线数据清洗                          │
└─────────┬───────────────────────────┬───────────────────┘
          │ 干净的单机数据              │ 层级结构与线损知识
          ▼                           ▼
┌─────────────────────┐  ┌──────────────────────────────┐
│  第三章  尾流建模    │  │  第四章  分层功率预测框架     │
│  尾流效应识别        │  │  单机预测模型                 │
│  尾流感知功率曲线    │  │  尾流修正集成                 │
│  激光雷达来流支撑    │  │  线损修正与Bottom-Up汇聚      │
└─────────┬───────────┘  └───────────┬──────────────────┘
          │ 尾流修正的单机预测         │
          └─────────────┬────────────┘
                        ▼
┌─────────────────────────────────────────────────────────┐
│  第五章  总结与展望                                      │
└─────────────────────────────────────────────────────────┘
```

---

## 各章详细内容

### 第一章 绪论

- **研究背景**：风电大规模并网，功率预测精度对电力系统调度的重要性
- **现有方法的三大局限**：
  - 局限①：SCADA 数据质量差（含噪、限电、重复时间戳），导致功率曲线建模偏差
  - 局限②：传统预测忽视尾流效应，机组间空间耦合未被建模
  - 局限③：点预测方法未考虑"单机→集电线路→全站"的层级电气约束
- **研究内容**：针对上述三大局限，分别在数据层、建模层、聚合层提出改进

---

### 第二章 数据质量提升与多层级功率结构分析

> **作用**：为后续所有建模提供干净可信的数据基础，并揭示层级功率关系

**2.1 风电 SCADA 数据多层级功率结构**
- 风机有功 → 集电线路有功 → 全站有功（三层级差异来源分析）
- 线损、厂用电、计量位置差异的量化
- 正值保留策略的合理性论证
- 仓库：[Fan-Line-Site-Active-Power-Data-Comparison](https://github.com/19bdxx/Fan-Line-Site-Active-Power-Data-Comparison)

**2.2 重复时间戳成因分析与处理**
- 重复数据产生机制（SCADA 采集特性）
- 去重策略选择与效果评估
- 仓库：[Analysis-of-the-causes-of-duplicate-data-from-wind-turbines](https://github.com/19bdxx/Analysis-of-the-causes-of-duplicate-data-from-wind-turbines)

**2.3 考虑空气密度的功率曲线数据清洗**
- 空气密度计算：ρ = f(T, P, RH)
- KNN 局部异常检测：以风速和空气密度为特征维度进行邻域筛选
- 清洗前后功率曲线质量对比；跨季节/跨风电场鲁棒性验证
- 仓库：[Wind-turbine-data-cleaning-considering-air-density](https://github.com/19bdxx/Wind-turbine-data-cleaning-considering-air-density)

---

### 第三章 风场尾流建模与尾流感知功率预测

> **作用**：揭示风机间的空间耦合机制，构建考虑尾流的单机功率预测模型

**3.1 风场尾流效应分析**
- 尾流对下游风机来流风速与湍流强度的影响机制
- 基于 SCADA 数据的尾流区域识别（主风向统计、功率下降特征）
- 经典解析尾流模型（Jensen / Gaussian 等）介绍与局限

**3.2 激光雷达来流风速支撑**
- 多普勒风廓线雷达数据分析：扫描模式识别（VAD/PPI/RHI）、CNR 质量控制
- 径向风速（RWS）反演与三维风场重建（VAD 方法）
- 转子等效风速（REWS）提取：$U_{\text{REWS}} = \frac{\int_A U(x,y)\,dA}{A}$
- 机舱激光雷达数据与 SCADA 风速对比
- 仓库：[Doppler-wind-profile-lidar-data](https://github.com/19bdxx/Doppler-wind-profile-lidar-data)、[LiDAR-for-wind-turbine-nacelles](https://github.com/19bdxx/LiDAR-for-wind-turbine-nacelles)

**3.3 尾流感知的单机功率预测**
- 将上游机组运行状态（风速、功率、偏航角）作为下游预测特征
- 数据驱动尾流模型（机器学习 / 深度学习）
- 相比不考虑尾流的基线模型的精度提升实验

---

### 第四章 风机-集电线路-场站分层功率预测框架

> **作用**：整合单机预测（含尾流修正）与层级电气约束，实现场站级精确预测

**4.1 分层预测框架设计**
- 层级结构：单机预测 → 集电线路聚合（含线损修正）→ 全站功率
- 与全局预测（直接对场站功率建模）的对比分析
- 自底向上（Bottom-Up）策略的适用条件

**4.2 单机预测模型**
- 多模型对比：Ridge、LightGBM、XGBoost、MLP、LSTM
- 特征工程：历史功率、风速、空气密度（来自第二章清洗结果）、尾流特征（来自第三章）
- 多步长预测实验（15/30/45/60/120/180 分钟）
- 仓库：[turbine-line-layered-forecast](https://github.com/19bdxx/turbine-line-layered-forecast)

**4.3 线损修正与集电线路聚合**
- 线损查表模型：功率区间 → 线损率映射
- 集电线路聚合策略（峡阳A/B、峡沙、蕴阳等多场站验证）
- 异常线损场景的鲁棒处理
- 仓库：[Line-station-layered-forecast](https://github.com/19bdxx/Line-station-layered-forecast)

**4.4 综合对比实验**
- 消融实验：① 不考虑空气密度 → ② 加入空气密度清洗 → ③ 加入尾流特征 → ④ 完整分层框架
- 全局预测 vs. 分层预测精度对比（RMSE / MAE / NRMSE）
- 不同风电场的泛化能力验证

---

### 第五章 总结与展望

- 各章研究成果总结
- 三大创新点凝练：
  - **创新点①**：提出了考虑空气密度的 KNN 局部异常检测数据清洗方法
  - **创新点②**：构建了基于 SCADA 数据与激光雷达的数据驱动尾流建模方法
  - **创新点③**：设计了融合尾流修正与线损约束的"单机-集电线路-全站"分层预测框架
- 研究局限与未来工作

---

## 三大创新点与研究的内在逻辑

```
传统方法的局限              本文的解决方案              对应章节
─────────────────────────────────────────────────────────
数据含噪/功率曲线偏差  →  空气密度感知的清洗方法    →  第二章
忽视尾流空间耦合      →  数据驱动尾流建模+激光雷达  →  第三章
点预测忽视层级约束    →  分层框架+线损物理修正      →  第四章
                      ↘ ↙
                  三章联合：消融实验验证每层改进的贡献
```

---

## 各仓库与论文章节对应关系

| 仓库 | 对应章节 | 状态 |
|------|----------|------|
| [Wind-turbine-data-cleaning-considering-air-density](https://github.com/19bdxx/Wind-turbine-data-cleaning-considering-air-density) | 第二章 §2.3 | ✅ 核心完成，待补充实验 |
| [Analysis-of-the-causes-of-duplicate-data-from-wind-turbines](https://github.com/19bdxx/Analysis-of-the-causes-of-duplicate-data-from-wind-turbines) | 第二章 §2.2 | 🔄 进行中 |
| [Fan-Line-Site-Active-Power-Data-Comparison](https://github.com/19bdxx/Fan-Line-Site-Active-Power-Data-Comparison) | 第二章 §2.1 | ✅ 完成 |
| 尾流建模（待建仓库） | 第三章 §3.1 §3.3 | 📋 待启动 |
| [Doppler-wind-profile-lidar-data](https://github.com/19bdxx/Doppler-wind-profile-lidar-data) | 第三章 §3.2 | 🔄 进行中 |
| [LiDAR-for-wind-turbine-nacelles](https://github.com/19bdxx/LiDAR-for-wind-turbine-nacelles) | 第三章 §3.2 | 🔄 数据就绪 |
| [turbine-line-layered-forecast](https://github.com/19bdxx/turbine-line-layered-forecast) | 第四章 §4.2 | ✅ 框架完成 |
| [Line-station-layered-forecast](https://github.com/19bdxx/Line-station-layered-forecast) | 第四章 §4.3 | ✅ 框架完成 |
| [XYB-wind-turbine-data-for-2025](https://github.com/19bdxx/XYB-wind-turbine-data-for-2025) | 第四章 实验数据 | ✅ 就绪 |
| [HaiLu-Station-Fan-Data-January-November-](https://github.com/19bdxx/HaiLu-Station-Fan-Data-January-November-) | 第二/四章 实验数据 | ✅ 就绪 |
| [wind-turbine-status-Qingzhou-6-](https://github.com/19bdxx/wind-turbine-status-Qingzhou-6-) | 第二章 附录/支撑 | ✅ 完成 |
| [Knowledge](https://github.com/19bdxx/Knowledge) | 贯穿全文 | 🔄 持续更新 |

---

## 下一步工作重点

1. **第三章（尾流）** 是目前最需要补充的部分——建议新建仓库，整理尾流建模代码和实验结果
2. **消融实验设计**（第四章§4.4）——这是串联各章节成果的关键实验，建议尽早规划
3. **统一实验数据集**——建议选定 1~2 个主要风电场作为贯穿全文的基准数据集，其余作为泛化验证
4. **补充第二章实验**——空气密度清洗方法的跨场站对比实验（已在 Issue#11 中规划）
