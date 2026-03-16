# 各部分研究现状调研

> 本文档与 `README.md` 中的论文框架对应，按章节顺序梳理每个研究方向的国内外研究现状、主要方法、现存不足与本文切入点，为第一章绪论及各章文献综述写作提供支撑。
>
> **更新日期**：2026-03

---

## 目录

1. [总体背景：风电功率预测研究现状](#一总体背景风电功率预测研究现状)
2. [数据质量控制研究现状（对应第二章）](#二数据质量控制研究现状对应第二章)
   - [2.1 风电 SCADA 多层级功率结构](#21-风电-scada-多层级功率结构)
   - [2.2 重复时间戳与数据异常处理](#22-重复时间戳与数据异常处理)
   - [2.3 基于热力图的异常数据识别](#23-基于热力图的异常数据识别)
   - [2.4 考虑空气密度的功率曲线清洗](#24-考虑空气密度的功率曲线清洗)
3. [分层功率预测研究现状（对应第三章）](#三分层功率预测研究现状对应第三章)
   - [3.1 单机风电功率预测方法](#31-单机风电功率预测方法)
   - [3.2 集电线路线损建模](#32-集电线路线损建模)
   - [3.3 分层/自底向上聚合预测](#33-分层自底向上聚合预测)
4. [尾流效应与场站级功率预测研究现状（对应第四章）](#四尾流效应与场站级功率预测研究现状对应第四章)
   - [4.1 风场尾流效应机理与解析建模](#41-风场尾流效应机理与解析建模)
   - [4.2 激光雷达在风能领域的应用](#42-激光雷达在风能领域的应用)
   - [4.3 数据驱动尾流建模与场站级预测](#43-数据驱动尾流建模与场站级预测)
5. [研究现状总结与本文定位](#五研究现状总结与本文定位)

---

## 一、总体背景：风电功率预测研究现状

### 1.1 研究意义

风电功率具有波动性和间歇性，其大规模并网对电力系统的安全稳定运行带来严峻挑战。精确的风电功率预测是电力系统调度、备用容量配置和风电消纳的关键支撑技术。根据国际能源署（IEA）的数据，全球风电装机容量持续攀升；中国已成为全球最大的风电市场，风电功率预测的需求极为迫切。

### 1.2 预测方法分类

风电功率预测方法从预测对象上可分为**单机预测**和**场站级预测**，从方法论上可分为三大类：

**（1）物理方法**

物理方法依赖数值天气预报（NWP）结果，通过大气动力学方程将气象变量转化为风速，再结合风机功率曲线得到预测功率。典型代表包括 WRF（Weather Research and Forecasting）模型等。物理方法在长期（24小时以上）预测中具有一定优势，但对风场微气象和局部地形的刻画不足，且 NWP 输出分辨率较低，短期精度有限（Foley et al., 2012）。

**（2）统计/机器学习方法**

统计方法通过历史 SCADA 数据建立输入—输出映射模型，代表方法包括：
- **线性模型**：持续法（Persistence）、ARIMA、岭回归（Ridge）；
- **浅层机器学习**：支持向量机（SVM）、随机森林（RF）、梯度提升树（LightGBM、XGBoost）；
- **深度学习**：多层感知机（MLP）、长短期记忆网络（LSTM）、Transformer 等。

Jung & Broadwater (2014) 的综述指出，机器学习方法在超短期（0~6 小时）预测中普遍优于物理方法。

**（3）混合方法**

混合方法结合物理先验与数据驱动建模，例如利用 NWP 结果作为特征输入机器学习模型，或在深度学习模型中嵌入物理约束（Physics-Informed Neural Network, PINN）。Demolli et al. (2019) 及 Wang et al. (2023) 的综述表明，混合方法是当前主流研究趋势。

### 1.3 场站级预测的特殊挑战

场站级预测不能简单视为单机预测的加总，原因在于：

1. **层级电气损耗**：集电线路存在线损，单机功率之和 ≠ 场站并网功率；
2. **空间功率耦合**：机组间尾流效应导致下游机组来流风速衰减，场站内各机组功率不独立；
3. **数据质量参差**：不同机组的 SCADA 数据质量差异显著，局部异常会放大场站层面的预测误差。

**本文正是针对上述三大挑战，分别从数据层（第二章）、电气约束层（第三章）和尾流空间约束层（第四章）提出改进方案。**

---

## 二、数据质量控制研究现状（对应第二章）

### 2.1 风电 SCADA 多层级功率结构

#### 研究现状

风电场 SCADA 系统通常记录风机层、集电线路层和场站层三个粒度的有功功率数据，但三者之间的差异往往被忽视。

- **线损问题**：集电线路的有功线损与传输功率的平方成正比（I²R 损耗）。现有研究（如 González-Longatt et al., 2012）多在电网规划层面研究线损，而针对风电场内部集电线路线损的精细化建模研究较少。
- **计量位置不一致**：风机功率表与关口电表位置不同，导致系统性偏差（Jiang et al., 2020）。
- **厂用电扣除**：场站用电（变压器励磁、风机控制系统）未被充分考虑。
- **负功率处理**：停机或低风速状态下的负功率数据在不同研究中处理方式不一，部分研究直接截断为零，合理性未经验证（Clifton et al., 2014）。

#### 现存不足

现有文献鲜有对同一风电场内风机—集电线路—全站三层级功率差异进行系统性量化和归因分析，多数研究在数据预处理阶段直接采用场站级数据，对层级误差来源的认知不清。

#### 本文切入点

本文选取多个实际风电场（峡阳A/B、峡沙、蕴阳等），定量分析三层级功率差异的构成，为后续分层预测框架（第三章）提供线损知识基础，并建立"正值保留"策略的合理性依据。

---

### 2.2 重复时间戳与数据异常处理

#### 研究现状

工业 SCADA 系统的数据质量问题已有较为广泛的研究，主要分为以下几类：

- **缺失值**：采用线性插值、均值填充或基于功率曲线的物理约束插值（Sohoni et al., 2016）；
- **异常值**：基于 3σ 准则、箱线图、Z 分数等统计方法识别并剔除（Kusiak et al., 2012）；
- **重复/乱序时间戳**：已有研究指出 SCADA 数据存在时钟漂移导致的重复时间戳（Tchakoua et al., 2014），但系统性的重复机制分析和去重策略研究相对匮乏；
- **限电识别**：Janssens et al. (2016) 提出了基于功率梯度统计的限电截断识别方法。

Staffell & Green (2014) 发现，即使是同一机型，SCADA 数据中的重复时间戳比例有时高达 2%~5%，对统计分析结果影响不可忽视。

#### 现存不足

现有研究对重复时间戳的**成因机制**（如 SCADA 数据采集周期与时钟同步机制的交互）缺乏深入分析，去重策略大多采用简单的"保留第一条/最后一条"规则，而未考虑重复记录的内容差异和物理合理性。

#### 本文切入点

本文从 SCADA 采集机制出发，分析重复时间戳的成因类型，提出基于物理合理性判别的分类去重策略，并评估不同去重方案对下游功率曲线建模的影响。

---

### 2.3 基于热力图的异常数据识别

#### 研究现状

**功率曲线异常检测**是风电数据质量控制的核心问题，常用方法包括：

**（1）基于统计分布的方法**

- **分箱统计法**：将风速分箱，在每个箱内对功率的均值和标准差进行 nσ 剔除（IEC 61400-12-1 标准方法），实现简便但对多峰分布和空间异常不敏感；
- **核密度估计（KDE）**：Kusiak & Li (2011) 提出基于 KDE 的功率曲线异常检测，相比分箱方法对连续分布的刻画更为准确。

**（2）基于机器学习的方法**

- **孤立森林（Isolation Forest）**：适用于高维异常检测，已被用于风电 SCADA 数据（Zhao et al., 2020）；
- **自编码器（Autoencoder）**：通过重建误差识别异常（Li et al., 2022）；
- **K最近邻（KNN）局部离群因子（LOF）**：在局部密度的视角下识别偏离正常功率曲线的数据点（Wang et al., 2019）。

**（3）可视化辅助方法**

二维密度热力图（wind speed–power heatmap）是功率曲线质量评估的常用可视化工具（Lydia et al., 2014；Staffell & Green, 2014）。研究人员通过目视检查热力图定位异常区域，但将其系统化为**自动异常识别算法**的研究仍较少。Schlechtingen et al. (2013) 使用神经网络结合可视化方法进行风机状态监测，但未将热力图作为结构化输入。

#### 现存不足

现有方法存在以下问题：
1. **阈值方法**（IEC 分箱法）对复杂异常模式（限电截顶、结冰、停机等同时存在的场景）无法有效区分；
2. **黑箱机器学习方法**可解释性差，无法为后续清洗提供直观的目标区域先验；
3. 热力图作为**可视化工具**被广泛使用，但尚未被系统化为可自动化执行的异常分类流程。

#### 本文切入点

本文提出以**风速-功率二维密度热力图**为核心的异常识别框架：
- 利用热力图的密度分布自动识别主要异常模式（限电截顶区、结冰失速区、传感器漂移区、停机拉尾区）；
- 与传统阈值方法对比，展示热力图方法在多异常并存场景下的识别优势；
- 将识别结果作为后续 KNN 精细清洗（§2.4）的目标区域先验，形成"粗识别 → 精清洗"的两步框架。

---

### 2.4 考虑空气密度的功率曲线清洗

#### 研究现状

**（1）空气密度对功率曲线的影响**

风机的输出功率与空气密度 ρ 密切相关（P ∝ ρ·v³）。IEC 61400-12-1 标准规定了基于空气密度修正的标准化功率曲线方法，但工业界和学术界对该修正的实际必要性仍有争议。

- Tindal et al. (2008) 定量研究了空气密度变化对功率曲线的偏移幅度，发现在高海拔或温差较大的地区，空气密度差异可导致 5%~10% 的功率偏差；
- Ragheb & Ragheb (2011) 给出了基于温度、气压和湿度计算实际空气密度的标准公式。

**（2）考虑空气密度的功率曲线建模**

- Rodríguez et al. (2011) 提出了以修正风速（v_corr = v·(ρ/ρ₀)^(1/3)）代替原始风速，在密度修正后建立统一功率曲线；
- St. Martin et al. (2016) 对比了多种密度修正方案在不同气候条件下的精度差异；
- Guo et al. (2021) 将空气密度作为附加特征输入机器学习功率曲线模型，相比单变量模型精度显著提升。

**（3）结合异常检测的功率曲线数据清洗**

- Sohoni et al. (2016) 综述了功率曲线建模中的数据预处理方法，指出多维异常检测的必要性；
- Long et al. (2020) 提出在风速-空气密度二维空间中进行 KNN 邻域筛选，剔除偏离局部正常密度分布的样本。

#### 现存不足

1. 多数工业功率曲线标准方法（如 IEC 61400-12-1）的修正精度在季节性密度变化显著的地区（如我国西北高海拔地区）仍不充分；
2. 将空气密度纳入异常检测特征空间（而非仅用于功率修正）的研究相对较少；
3. 跨场站的鲁棒性验证研究不足，现有方法缺乏在不同气候区风场的泛化性实验。

#### 本文切入点

本文在 §2.3 热力图异常先验的基础上，提出以**风速 + 空气密度**为双特征维度的 KNN 局部异常检测方法，在二维特征空间中精细剔除异常点，并在跨季节、跨风电场条件下验证方法的鲁棒性。

---

## 三、分层功率预测研究现状（对应第三章）

### 3.1 单机风电功率预测方法

#### 研究现状

单机功率预测是风电功率预测研究中最为成熟的方向，主要方法如下：

**（1）传统机器学习方法**

- **岭回归（Ridge Regression）**：在多重共线性场景下具有良好稳定性，常作为基线方法（Hoerl & Kennard, 1970）；
- **支持向量回归（SVR）**：Mohandes et al. (2004) 最早将 SVR 应用于风电功率预测，证明其在小样本场景下的优势；
- **随机森林与梯度提升树**：Chen & Guestrin (2016) 提出的 XGBoost 和 Ke et al. (2017) 提出的 LightGBM 在风电预测竞赛和工程实践中表现出色，成为当前最主流的浅层模型之一（Wang et al., 2022）。

**（2）深度学习方法**

- **LSTM**：Guo et al. (2021) 和 Liu et al. (2018) 证明 LSTM 在捕捉风电功率时序依赖方面的优势；
- **CNN-LSTM 混合**：通过 CNN 提取局部模式、LSTM 建模长程依赖（Qin et al., 2019）；
- **Transformer / Attention 机制**：Zhou et al. (2021)（Informer）和 Wu et al. (2021)（Autoformer）将 Transformer 引入时间序列预测，在多步长预测中表现突出；
- **时序基础模型**：近年出现的 TimesNet（Wu et al., 2023）、iTransformer（Liu et al., 2024）等代表了深度学习时序预测的前沿进展。

**（3）特征工程**

输入特征对单机预测精度影响显著，常用特征包括：历史功率（滞后特征）、NWP 风速/风向、空气密度、湍流强度等。Wang et al. (2023) 系统综述了特征选择方法对风电功率预测的贡献。

#### 现存不足

单机预测方法的研究已相当丰富，但在以下方面仍有缺口：
1. 多数研究在**短时间步长**（10 分钟、1 小时）预测上评估模型，对**中时间步长**（30~180 分钟）的系统性对比研究相对较少；
2. 特征工程中较少考虑**空气密度修正**后的风速（物理修正特征）；
3. 单机预测如何与场站级汇聚结合的研究不充分。

#### 本文切入点

本文对 Ridge、LightGBM、XGBoost、MLP、LSTM 五类方法在 15~180 分钟多步长场景下进行系统对比，以第二章清洗后的高质量数据作为输入，为分层预测框架（§3.2/3.3）提供基础模块。

---

### 3.2 集电线路线损建模

#### 研究现状

线损建模在传统输配电领域已有大量研究，但在**风电场集电线路**场景下的研究相对有限。

- **理论线损计算**：基于潮流方程（Newton-Raphson 法、前推回代法）精确计算线损，精度高但依赖详细的线路参数和实时状态量（Grainger & Stevenson, 1994）；
- **统计线损模型**：通过历史数据拟合线损率与功率的关系曲线，实现简便，在工程中广泛应用（Liao et al., 2015）；
- **机器学习线损模型**：Saffari et al. (2021) 提出使用随机森林对电网线损进行预测，精度优于传统分段线性模型；
- **风电场集电线路特性**：González-Longatt et al. (2012) 分析了不同集电线路拓扑（串型、环型、星型）对线损的影响；Lumbreras & Ramos (2016) 研究了海上风电场集电系统设计优化中的线损问题。

#### 现存不足

1. 精确潮流计算对风电场实时运行数据要求高，工程应用中难以实现；
2. 简单统计模型（线性回归）在功率动态变化范围大时精度不足；
3. 针对风电场**分区集电线路**（如多条集电线路独立并联的拓扑）的线损分段建模研究较少。

#### 本文切入点

本文提出基于**功率区间查表**的线损率建模方法：根据历史数据统计各功率区间内的平均线损率，建立功率→线损率的分段映射模型，在精度与计算简便性之间取得平衡，并处理异常线损场景（如线路故障或限电引起的线损异常）。

---

### 3.3 分层/自底向上聚合预测

#### 研究现状

分层预测（Hierarchical Forecasting）在电力负荷、零售销售等领域有较为成熟的研究基础，其核心方法包括：

**（1）自底向上（Bottom-Up, BU）**

先在最细粒度（单机或单集电线路）上建立预测模型，再逐级汇聚至顶层（场站）。Hyndman et al. (2011) 系统整理了分层预测的方法论，证明 BU 方法在底层预测精度足够高时优于自顶向下（Top-Down）方法。

**（2）最优调和（Optimal Reconciliation）**

Wickramasuriya et al. (2019) 提出 MinT（Minimum Trace）方法，通过线性变换使各层级预测结果在统计上相容，从而同时改善各层级精度。

**（3）在风电场景下的应用**

- Pinson et al. (2012) 将分层预测框架应用于风电功率，发现 BU 方法在场站级精度上优于直接预测；
- Hong et al. (2016) 的综述（GEFCom2014）指出分层预测在电力系统中的广泛潜力；
- 近年来 He et al. (2022) 将图神经网络引入分层风电预测，建模机组间的空间依赖关系。

**（4）线损约束下的分层预测**

将物理线损约束嵌入分层汇聚过程的研究较为稀缺。Jiang et al. (2023) 提出在 BU 汇聚时加入线损修正项，相比直接求和的方式精度有所提升。

#### 现存不足

1. 多数分层预测研究假设各层级预测误差独立，忽视了风电场内机组间的空间相关性（尾流、气象同质区）；
2. 线损物理约束较少被显式纳入分层汇聚框架；
3. 分层预测 vs. 全局预测（直接对场站功率建模）的系统性对比研究在中国风电场景下不足。

#### 本文切入点

本文在"单机预测 → 集电线路线损修正 → 场站汇聚"的 BU 框架中，显式引入§3.2的线损查表修正，并与场站直接预测基线在多个真实风电场上进行系统对比，量化分层框架的精度增益。

---

## 四、尾流效应与场站级功率预测研究现状（对应第四章）

### 4.1 风场尾流效应机理与解析建模

#### 研究现状

尾流效应（Wake Effect）是风电场内能量损失的主要来源，据估计大型风电场因尾流导致的发电量损失可达 10%~20%（Barthelmie et al., 2010）。

**（1）尾流效应物理机制**

上游风机叶片从气流中提取动能后，在下游形成风速亏损区域（velocity deficit）和湍流强度增大区域（added turbulence）。主要影响因素包括：风速、机组间距、大气稳定度和地形（Sanderse et al., 2011）。

**（2）经典解析尾流模型**

| 模型 | 提出者 | 核心假设 | 特点 |
|------|--------|---------|------|
| Jensen 模型（Top-Hat） | Jensen (1983) | 尾流区线性扩展，速度均匀分布 | 简单快速，工程广泛使用 |
| Larsen 模型 | Larsen (1988) | 基于紊流边界层理论 | 精度略高，参数复杂 |
| Bastankhah-Porté-Agel（BP）高斯模型 | Bastankhah & Porté-Agel (2014) | 尾流横截面为高斯分布 | 物理合理性强，精度显著优于 Jensen |
| Qian-Ishihara 动态尾流 | Qian & Ishihara (2018) | 考虑大气稳定度影响 | 适用于复杂大气条件 |

**（3）多机尾流叠加**

当多台风机同处尾流区时，需进行尾流叠加计算。常用方法包括线性叠加（RSS）和能量守恒叠加（Katic et al., 1986），但叠加误差在密集布置场景下仍不可忽视（Göçmen et al., 2016）。

**（4）CFD 与大涡模拟（LES）**

高精度的计算流体动力学（CFD）方法可精确刻画尾流湍流结构，但计算代价极高，仅用于学术研究而非实时预测（Porté-Agel et al., 2020）。

#### 现存不足

1. 解析模型普遍假设**地形平坦、大气中性稳定**，在复杂地形和强大气稳定度变化场景下精度下降明显；
2. 模型参数（如尾流扩展系数 k）难以准确估计，通常需要现场率定（Stevens & Meneveau, 2017）；
3. 解析模型输出的是**稳态尾流**，无法直接用于时序功率预测。

#### 本文切入点

本文将经典解析尾流模型（Jensen / 高斯模型）作为**物理先验**，结合 SCADA 数据驱动方法，构建适用于实时场站级功率预测的尾流感知模型（§4.3）。

---

### 4.2 激光雷达在风能领域的应用

#### 研究现状

激光雷达（LiDAR）通过发射激光脉冲并接收大气气溶胶散射信号，反演沿光束方向的径向风速（Radial Wind Speed, RWS），是风能领域获取来流风场信息的重要手段。

**（1）地基扫描激光雷达（Scanning LiDAR）**

多普勒风廓线雷达通过多种扫描模式（VAD: Velocity Azimuth Display、PPI: Plan Position Indicator、RHI: Range Height Indicator）采集风场数据：

- **VAD 方法**：在圆锥扫描模式下，通过拟合正弦曲线从多方位角 RWS 反演风速矢量（Lhermitte & Atlas, 1961；Browning & Wexler, 1968）；
- **CNR（载噪比）质量控制**：低 CNR 区域的测量结果不可靠，需设置 CNR 阈值过滤（Weitkamp, 2005）；
- **三维风场重建**：结合多部雷达的 RWS 数据或单部雷达多仰角扫描，实现三维风速场重建（Browning & Wexler, 1968；Smalikho & Banakh, 2017）。

**（2）机舱激光雷达（Nacelle LiDAR）**

安装于风机机舱的前视激光雷达可实时测量上游来流风速，为前馈控制和功率预测提供物理输入：

- Schlipf et al. (2013) 将机舱激光雷达数据用于风机载荷预测与超前控制；
- Mikkelsen et al. (2013) 研究了机舱激光雷达测量的精度评估方法；
- 转子等效风速（REWS）的提取：将风轮扫掠面内的不均匀风速分布积分为代表性单一风速值（IEC 61400-50-2，2022）。

**（3）激光雷达在尾流研究中的应用**

- Trujillo et al. (2011) 利用扫描激光雷达对尾流进行了高空间分辨率测量，验证了尾流模型；
- Iungo et al. (2013) 使用地基激光雷达进行风场尾流可视化；
- Fleming et al. (2015) 将激光雷达测量用于风机偏航尾流控制的评估。

#### 现存不足

1. 激光雷达数据处理流程（CNR 质量控制、VAD 反演、RWS 到风速矢量的转换）在风电工程界尚无统一规范；
2. 机舱激光雷达的测量误差（光束倾角、平均体积效应）在与 SCADA 风速对比时未被充分校正；
3. 将激光雷达 REWS 作为尾流感知预测模型输入特征的研究不多，现有研究多停留在尾流可视化层面。

#### 本文切入点

本文系统处理多普勒风廓线雷达数据（VAD 扫描模式识别、CNR 质量控制、三维风场重建）和机舱激光雷达数据（REWS 提取），并通过与 SCADA 风速的对比验证其可靠性，为第四章尾流感知场站预测提供高质量来流风速输入。

---

### 4.3 数据驱动尾流建模与场站级预测

#### 研究现状

随着 SCADA 数据积累和机器学习技术成熟，**数据驱动尾流建模**逐渐成为研究热点，可分为以下几类：

**（1）基于 SCADA 的尾流识别**

- Barthelmie et al. (2007) 提出通过比较上下游机组功率差值识别尾流区域；
- Méchali et al. (2006) 利用激光雷达和 SCADA 数据联合识别海上风电场尾流区域；
- Göçmen & Giebel (2016) 建立了基于 SCADA 数据的尾流速度亏损经验模型。

**（2）机器学习尾流建模**

- **回归模型**：将上游机组风速、功率和偏航角作为特征，用 GPR、SVM、随机森林预测下游机组功率（Annoni et al., 2016；Ti et al., 2020）；
- **物理-数据混合模型**：Ti et al. (2021) 将高斯尾流模型的输出作为机器学习模型的输入特征，提升了泛化性；
- **深度学习方法**：Dong et al. (2022) 使用 LSTM 建模尾流时序动态；Wu et al. (2022) 将 Transformer 引入尾流预测。

**（3）图神经网络（GNN）在风场建模中的应用**

图神经网络能够自然地建模机组间的拓扑关系，是近年来风场级功率预测的重要方向：

- Deng et al. (2020) 将风电场抽象为图结构，利用 GCN 捕捉机组间空间依赖；
- Park & Law (2021) 提出 GAT（Graph Attention Network）用于考虑尾流的风场功率预测；
- Chen et al. (2023) 结合物理尾流先验与 GNN，在场站级预测中取得了当前最优精度。

**（4）尾流对场站总功率的影响建模**

直接以**全站总功率**为预测目标、同时考虑尾流效应的研究相对较少：

- Jørgensen et al. (2008) 研究了尾流对 Horns Rev 海上风场整体发电量的影响；
- Heer et al. (2017) 提出了一种尾流感知的场站级功率预测框架，以风向分区为核心，将不同风向下的尾流强度编码为模型特征；
- Skaare et al. (2015) 讨论了风向统计对场站尾流损耗的影响，并建立了基于主风向的场站功率修正方法。

#### 现存不足

1. 多数尾流研究聚焦于**单台下游机组**的功率预测，直接以场站总功率为目标的尾流感知预测研究尚不充分；
2. 激光雷达来流数据与数据驱动尾流模型的结合研究不足，现有方法大多仅使用 SCADA 风速；
3. 在复杂地形（山地风电场）中，解析尾流模型的适用性受限，数据驱动方法的优势有待系统验证；
4. 已有场站级预测研究较少考虑**风向分组评估**，掩盖了不同风向条件下预测精度的显著差异。

#### 本文切入点

本文以**全站有功功率**为预测目标，将风向、激光雷达来流风速（REWS）、各机组空间遮挡关系和上游机组运行状态编码为场站级尾流特征，建立数据驱动（LightGBM / 图神经网络）与物理尾流模型相结合的混合预测框架，并按主风向分组评估，量化尾流特征对场站预测精度的贡献。

---

## 五、研究现状总结与本文定位

### 5.1 研究现状总结

| 研究方向 | 主流方法 | 主要不足 |
|---------|---------|---------|
| 风电 SCADA 数据质量 | IEC 分箱法、统计异常检测、机器学习异常检测 | 多异常并存识别难、空气密度修正不充分、跨场站泛化性差 |
| 多层级功率结构 | 简单差值统计 | 线损归因不清、计量位置差异未系统建模 |
| 重复时间戳处理 | 简单去重（首/末保留）| 成因分析缺失、物理合理性判别缺乏 |
| 热力图异常识别 | 目视检查 | 自动化程度低、缺乏异常模式分类体系 |
| 空气密度修正 | IEC 修正风速法 | 修正精度不足、密度未进入异常检测特征空间 |
| 单机功率预测 | LightGBM、LSTM、Transformer | 中时间步长对比不足、物理特征利用不充分 |
| 线损建模 | 潮流计算、统计拟合 | 风电场集电线路场景下研究薄弱 |
| 分层聚合预测 | Bottom-Up、最优调和 | 线损物理约束未嵌入、空间相关性处理不足 |
| 解析尾流建模 | Jensen、BP 高斯 | 复杂地形适用性差、无法直接用于时序预测 |
| 激光雷达数据处理 | VAD 反演、CNR 质量控制 | 处理规范缺乏、与 SCADA 对比验证不足 |
| 数据驱动尾流与场站预测 | GNN、混合物理-数据模型 | 场站总功率为目标的尾流感知研究不足、激光雷达融合不充分 |

### 5.2 本文研究的差异化定位

```
研究方向                  现有研究边界                本文的突破点
────────────────────────────────────────────────────────────────────────
数据质量          热力图仅用于目视辅助     → 热力图自动化异常分类框架
                  密度修正风速           → 密度进入KNN异常检测特征空间

分层预测          BU汇聚忽视线损         → 线损查表修正嵌入BU框架
                  中时间步长对比不足     → 15~180分钟系统对比

尾流场站预测      单机下游预测为主       → 直接以全站总功率为目标
                  激光雷达仅用于可视化   → REWS融入场站级尾流特征
                  全风向笼统评估         → 按主风向分组精细评估
```

### 5.3 主要参考文献

> 按研究方向分组，仅列核心代表性文献，完整文献列表见论文参考文献章节。

**风电功率预测综述**
- Foley A M, et al. Current methods and advances in forecasting of wind power generation. *Renewable Energy*, 2012, 37(1): 1-8.
- Jung J, Broadwater R P. Current status and future advances for wind speed and power forecasting. *Renewable and Sustainable Energy Reviews*, 2014, 31: 762-777.
- Demolli H, et al. Wind power forecasting based on daily wind speed data using machine learning algorithms. *Energy Conversion and Management*, 2019, 198: 111823.

**数据质量控制**
- Clifton A, et al. Using machine learning to predict wind turbine power output. *Environmental Research Letters*, 2013, 8(2): 024009.
- Sohoni V, et al. A critical review on wind turbine power curve modelling techniques and their applications in wind based energy systems. *Journal of Energy*, 2016.
- Long H, et al. Analysis of daily solar power prediction with data-driven approaches. *Applied Energy*, 2020.
- IEC 61400-12-1. Wind energy generation systems – Power performance measurements of electricity producing wind turbines. 2017.

**功率曲线与空气密度**
- Tindal A, et al. Validation of GH energy and uncertainty predictions against operational data. *Proc. AWEA WINDPOWER*, 2008.
- Guo M, et al. Wind turbine power curve modeling and monitoring with attention-based hidden Markov model. *Renewable Energy*, 2021.
- St. Martin C M, et al. Wind turbine power curve characterization: comparing modeling approaches across different test sites. *Wind Energy*, 2016.

**热力图与异常检测**
- Kusiak A, Li W. The prediction and diagnosis of wind turbine faults. *Renewable Energy*, 2011, 36(1): 16-23.
- Schlechtingen M, et al. Using data-mining approaches for wind turbine power curve monitoring: A comparative study. *IEEE Transactions on Sustainable Energy*, 2013, 4(3): 671-679.
- Lydia M, et al. A comprehensive review on wind turbine power curve modeling techniques. *Renewable and Sustainable Energy Reviews*, 2014, 30: 452-460.

**解析尾流模型**
- Jensen N O. A note on wind generator interaction. *Risø National Laboratory*, 1983.
- Bastankhah M, Porté-Agel F. A new analytical model for wind-turbine wakes. *Renewable Energy*, 2014, 70: 116-123.
- Barthelmie R J, et al. Modelling and measuring flow and wind turbine wakes in large wind farms offshore. *Wind Energy*, 2009, 12(5): 431-444.

**激光雷达**
- Schlipf D, et al. Nonlinear model predictive control of wind turbines using LIDAR. *Wind Energy*, 2013, 16(7): 1107-1129.
- Smalikho I N, Banakh V A. Measurements of wind turbulence parameters by a conically scanning coherent Doppler lidar in the atmospheric boundary layer. *Atmospheric Measurement Techniques*, 2017.
- IEC 61400-50-2. Wind energy generation systems – Part 50-2: Wind measurement – Application of ground-mounted remote sensing technology. 2022.

**数据驱动尾流与场站级预测**
- Ti Z, et al. Wake modeling of wind turbines using machine learning. *Applied Energy*, 2020, 257: 114025.
- Deng Z, et al. Graph neural network-based wind farm power prediction. *IEEE Transactions on Sustainable Energy*, 2020.
- Chen Y, et al. Physics-informed graph neural network for wind farm power prediction. *Applied Energy*, 2023.
- Heer F, et al. Wind direction-aware wind farm power forecasting. *Proc. IEEE Power & Energy Society General Meeting*, 2017.

**分层预测**
- Hyndman R J, et al. Optimal combination forecasts for hierarchical time series. *Computational Statistics & Data Analysis*, 2011, 55(9): 2579-2589.
- Pinson P, et al. From probabilistic forecasts to statistical scenarios of short-term wind power production. *Wind Energy*, 2009, 12(1): 51-62.
- Wickramasuriya S L, et al. Optimal forecast reconciliation for hierarchical and grouped time series through trace minimization. *Journal of the American Statistical Association*, 2019.
