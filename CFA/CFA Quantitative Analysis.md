## CFA Level I Quantitative Analysis

### 1. Time value of money

#### 1. Interest rate

1. 概率论统计
   - 描述性统计
   - 推断性统计
     - 由样本估计总体
     - 假设检验

2. 货币的时间价值
   - Interest rate
   - EAR 有效年利率
   - Annuities 年金
   - Required rate 要求回报率
   - Discount rate 折现率
   - Oppotunity cost 机会成本
   - Risk-free rate 名义的无风险利率
     - Risk - free rate  = real risk-free rate + inflation rate 
     - Risk premium 风险溢价
     - Ri = Rf + Rp
     - Rp = default Rp 违约风险 + liquidity Rp 流动性风险 + maturity Rp 期限风险

#### 2. EAR

1. EAR => Effective annual rate
2. 单利 仅本金计息 360天
3. 复利 利息也计息 365天
4. $EAR = (1 + \frac{r}{m})^m -1$ 
5. **r** → nominal rate per year **m** → 年计息次数
6. 连续复利 EAR = e^r^ -1 
7. 金融计算器
   - CHN 链式运算
   - AOS 代数运算

#### 3. Annuity
1. amortization 分期偿还 
2. ordinary annuities 后付年金
3. annuity due 后付年金
4. 我 outflow "-" 银行 inflow "+"
5. Perpetuity 永久
6. 永续年金的PV = A除以r

### 2. Descriptive statistics

#### 1. HPR

1. **Holding Period Return【HPR】 持有期收益率**
2. R = **收益** 除以 成本 (**不是一个年化的收益率**)

3. **收益**
4. Capital gain 资本利得
   - Income 
5. T-bill 
   - Treasury 国债
     - 小于等于 1 年 T-bill **零息债券**
     - 1 到 10年 T-note
     - 大于10年 T-bond
6. Descriptive statistics 描述统计
7. Inferential statistics 推断统计
8. Population 总体
9. sample statics 样本统计量

####  2. Measurement scale

1. Types of measurement scales: 
   - Nomial scales
   - Ordinal scales < >
   - Interval scales < > + - 
   - Ratio scales < > + - * /

#### 3. Mean dispersion -CV-and-SR

1. dispersion 散布
2. cumulative 累积
3. measures of central tendency 衡量中心趋势
   - mode 众数
   - Median 中位线
   - **The Arithmetic Mean ** 算术平均
   - **The Weighted Mean** 加权平均
   - **The Geometric Mean** 几何平均
   - **The Harmonic Mean** 调和平均
   - **A ≥ G ≥ H**
   - 算术/加权 ：预测未来
   - 几何
     - focus on ending value
     - 历史业绩
4. Quintile 五分位数
5. Measures of Dispersion 衡量离散程度
   - Range = max - min
   - MAD (**Mean Absolute deviation**)  平均绝对偏差
   - ![image-20210101180915672](https://raw.githubusercontent.com/silence/blog/assets/assets/20210224232350.png)
   - 总体 **希腊字母** 样本 **英文字母**
6. Chebyshev's Inequality  切比雪夫不等式
   - $P(\mu - k\sigma \le x \le \mu + k \sigma)  \ge 1-\frac{1}{k^2}$
7. CV and SR
   - **Coefficient of variation**  变异系数 又称 离散系数 为 标准差 与 平均值 之比 $CV = \frac{S}{\bar X}$
     - 衡量 每单位均值的离散程度 / 每单位均值的风险
     - 性质
       - Scale - free
       - Relative 相对离散程度
   - **Sharpe ratio**
     - 业绩衡量的指标
     - $SR = \frac{R_p - R_f}{\sigma_p}$ P: portfolio
     - 每单位风险的超额收益
     - excess return 超额收益

#### 4. Skewness

1. **Positve skewed** and **negative skewed**

   - 长尾在哪边就往哪边偏

   - gain 利润
   - $$S_k= [\frac{n}{(n-1)(n-2)}]\frac{\sum_{i=1}^n(X_i-\bar X)^3}{s^3}\approx(\frac{1}{n})\frac{\sum_{i=1}^n(X_i-\bar X)^3}{s^3}$$

   - 右偏 > 0
   - 左偏 < 0

#### 5. Kurtosis

1. L > 3 N = 3 P < 3
2. power = 4
3. 高峰肥尾 假设$\sigma^2$和正态分布一样 

### 3. Probability concepts

#### 1. Probability Concepts

1. Random variable 随机变量
2. Outcome 结果
3. Event 事件
   - Mutually exclusive events 互斥事件
   - Exhaustive events 遍历事件/全部事件
4. Objective Probability 客观概率
   - Empirical probability 经验概率
   - Priori probability 先验概率
5. Subjective probability 主观概率
6. **Odds for** an event 赔率 p/(1-p)
7. Odds against an event  (1-p)/p
8. Unconditional Probability (marginal probability) 非条件概率/边际概率
9. Conditional probability : P(A丨B) 条件概率

#### 2. Multiplicaion and Addition Rule

1. Joint probability P(AB) 联合概率

   - Multiplication rule $P(AB) = P(A\lvert B)\times P(B) = P(B\lvert A) \times P(A)$

   - If A and B are mutually exclusive events, then : P(AB) = 0

2. Addition rule

   - $P(A or B) = P(A) + P(B) - P(AB)$
   - If A and B are mutually exclusive events, then : P(AB) = P(A) + P(B)

3. 互斥事件一定不是独立事件

4. **Expected Value** 期望

5. Earnings per share 每股收益 

#### 3. Covariance and Correlation

1. Covariance 协方差
   - 定义：两个随机变量变化的方向性 同向 > 0 反向 < 0
   - $Cov_{xy} = E[(x_i-\bar x)(y_i - \bar y)]$  范围： $-\infin \sim + \infin$
   - $Cov_{xx} = \sigma_x^2$ **自己和自己的协方差就是方差**
   
2. Correlation 相关系数
   - $\rho_{xy} = \frac{Cov_{xy}}{\sigma_x\sigma_y}$
   - **Linear** relationship **线性关系**
   - 范围： $-1\sim +1$
   - slope 斜率 
   - 缺陷
     - 只能衡量线性关系
     - 异常值
     - 伪相关
#### 4. Portfolio Return and Risk

1. $\sigma_p^2  = w_1^2\sigma_1^2 + w_2^2\sigma_2^2 + 2w_1w_2\sigma_1\sigma_2\rho_{12}$
   - 若$\rho = 1$  $\sigma_p = w_1\sigma_1 + w_2\sigma_2$
   - 若$\rho = -1$  $\sigma_p = \lvert w_1\sigma_1 - w_2\sigma_2 \lvert$
   - 分散化效果好

#### 5. Bayes' Formula

#### 6. Principles of Counting

1. 乘法法则
2. 阶乘
3. 排列组合

#### 7. Different Types of Random Variables

1. Discrete 离散
2. Continuous 连续
3. Discrete uniform distribution 离散均匀分布
4. Binomial distribution 二项式分布
   - $C_n^xP^x(1-p)^{n-x}$
   - Expectation : np
   - Variance: np(1-p)
5. Continuous Uniform Distribution 连续均匀分布
6.  Decade 十年

### 4. Normal Distribution

1.  $X \sim N(\mu, \sigma^2)$

2. skewness = 0 ; kurtosis = 3

3. 正态分布的组合依然符合正态分布

4. 概率分布同k值

   - **68% k = 1**

   - **90% k = 1.65**
   - **95% k = 1.96**
   - **99% k = 2.58**

5. 标准化

   - $\mu = 0, \sigma^2 = 1$
   - If  $X \sim N(\mu, \sigma^2)$ ; $\frac{X-\mu}{\sigma} \sim N(0,1)$

#### 1. Safety First Ratio

1. 第一安全比率
2. allocation 分配

#### 2. Lognormal Distribution 

1. 若 $lnx$ is normal 则 x is lognormal
2. Right skewed

#### 3. Simulation

1. 蒙特卡洛模拟
2. 历史模拟

### 5. Sampling and estimation

#### 1. Sampling

1. Simple random sampling 简单随机抽样法

2. Stratified random sampling 分层随机抽样法

   - 将整体分成 M 份，再从 M 份中抽取 N 份

3. Sampling error

   - sampling error of the mean = sample mean - population mean 抽样误差等于抽样均值 - 整体均值

4. Data 分类

   - Time series data 时间序列数据
   - Cross sectional data 横截面数据

5. **Central limit theory 中心极限定理**

   > 中心极限定理指的是给定一个任意分布的总体。我每次从这些总体中随机抽取 n 个抽样，一共抽 m 次。 然后把这 m 组抽样分别求出平均值。 这些平均值的分布接近正态分布。

   - 条件：
     - 样本容量足够大 n >= 30
     - 总体的均值和方差已知
   - **结论：**
     - 样本均值服从正态分布
     - 样本均值的均值 = 总体均值
     - 样本均值的方差 = 总体方差 / n
     - 若想用 样本均值 $\bar{x}$ => 去估计整体的 $\mu$ ，则 $\bar{x}$ ~ $N(\mu_\bar{x}, \sigma^2_\bar{x} = \frac{\sigma^2_x}{n})$ 
   - standard error 标准误
     - 衡量的是**样本均值的离散程度**
     - 样本均值的标准差称为 standard error
     - 计算：$\sigma_\bar{x} = \frac{\sigma_x}{\sqrt{n}}$

#### 2.  Estimation

1. properties of estimator 
   - Unbiasedness 无偏性，即中心极限定理的结论
   - Efficiency 有效性，方差最小的
   - Consistency 一致性，抽样的数目越多，估计的越准确
2. 估计方法
   - Point estimate 点估计，认为抽样的样本均值就是总体均值
   - Interval estimate 区间估计，（统计学要严谨，即存在置信区间，认为抽样的样本均值在某个区间的概率是多少）
     - 我们要用 $\bar{x}$ 去估计 $\mu_x$ ，所以 $CI_\bar{x} = \bar{x} \pm k\frac{\sigma}{\sqrt{n}}$
     - 计算
     - 置信区间的宽度 $(\bar{x} + k\frac{\sigma}{\sqrt{n}}) - (\bar{x} - k\frac{\sigma}{\sqrt{n}}) = 2k\frac{\sigma}{\sqrt{n}}$

#### 3. T-distribution

1. 特征
   - Symmetrical (Skewness = 0)
   - Degrees of freedom （df = n -1）
   - Less peaked , fatter tails 低峰肥尾
     - kurtosis < 3
     - 方差大 $\sigma^2 > 1$
   - Student's t-distribution converages to the standard normal distribution as degrees of freedom goes to infinity 当自由度逐渐增大时，t - 分布会逐渐接近于标准正态分布 N(0, 1) 。
2. z/t 分布判断
   - 方差已知用z
   - 方差未知用t
   - 非正态总体小样本不可估计
   - n >= 30，任何情况下均可用 z

#### 4. Bias

1. Data-mining bias 把偶然当必然，（得到的只是数据和数据之间的关系，实际没有任何含义）
2. Sample selection bias 样本选择性偏差，（样本是有选择性的，有些样本没有被选到）
3. Survivorship bias 生存性偏差 （有些公司已经死掉，被排除在外了）
4. Look-ahead bias 前视性偏差 （未来有些数据是看不到的，是估计出来的）
5. Time-period bias 时间段偏差（有些过久的数据现在来看没有任何意义）

### 6. 【TODO没学好，回去好好看概率论假设检验】Hypothsis testing

#### 1. Critical method 关键方法

1. ![image-20210227174108431](https://raw.githubusercontent.com/silence/blog/assets/assets/20210227174108.png)

2. State the hypothesis 设假设
   - Null hypothesis 原假设 （拒绝的、怀疑的）
   - Alternative hypothesis 备择假设 （接受的）
   - One-tailed and Two-tailed 单尾还是双尾
   
3. Test statistic 检验统计量
   - Z 分布 （总体方差已知）
     - $\text{Test Statistic} = \frac{\bar{X} - \mu_0}{\sigma/\sqrt{n}}$
   - t 分布 （总体方差未知）
     - $\text{Test Statistic} = \frac{\bar{X} - \mu_0}{s/\sqrt{n}}$
   
4. Critical value（关键值，实际就是 k 值）
   - K 值的影响因素
     - 显著性水平
     - 查的表（T or Z）
     - 单尾 or 双尾
   
5. **TODO**：假设检验的几个步骤，去看中文的吧，都学过忘了...

   > 说人话版解释 We can not reject the null hypothesis. 
   >
   > 在不能穷举的情况下，你不能靠举例子来证明一个命题是正确的，但是你可以靠举出一个反例来证明一个命题是错误的。
   >
   > 用统计的说法就是：
   >
   > 1. **不轻易拒绝原假设**
   > 2. **小概率事件发生不正常**。如果小概率事件还是发生了，那么就说明原假设有问题。
   >
   > 关于 significant level （显著水平）.
   >
   > 显著水平，用1减一下，得到的值叫做容错比例，1% 的显著水平，变成了 99% 的容错比例，如 1 % 的显著水平，因为其容错更高，自然是更容易接受原假设。
   >
   > 显著这个词，其实就是**有足够强的证据的意思**。例如：**这个结果显著**，翻译成人话，就是我们有足够强的证据表明这个结果超过了容错范围，所以我们要 `reject` 掉原假设

   - 判断对什么参数检验
   - 成对数检验 t分布
   - 1个方差进行检验：卡方分布
   - 2个方差进行检验：F 分布

6. 如何选择采用哪种假设检验

   - **Z检验**：**一般用于大样本（即样本容量大于30）平均值差异性检验的方法（总体的方差已知）**。它是用标准正态分布的理论来推断差异发生的概率，从而比较两个平均数的差异是否显著。在国内也被称为 **u 检验**

     > 例：一种原件，要求使用寿命不低于1000小时，现从一批这种原件中抽取25件，测得其使用寿命的平均值为950小时，已知该原件服从标准差S=100小时的正太分布，试在显著性水平α=0.05下确定这批原件是否合格

   - **T检验**：**主要用于样本含量较小（例如 n < 30），总体标准差 $\sigma$ 未知的正态分布。**T检验是用 t 分布理论来推论差异发生的概率，从而比较两个平均数的差异是否显著。

     > 1. 单样本 t 检验：
     >
     > 已知某班的一次数学测验成绩复查正态分布，现从全班中抽取16人，测得这些人成绩是[50,44,91,90,74,72,89,81,65,62,68,74,63,61,33,47]，问在α=0.05下，是否可以认为全体考生的平均分是70分？
     >
     > 2. 配对样本 t 检验：
     >
     > 在针织品漂白工艺过程中， 要考虑温度对针织品断裂强力（主要质量指标）的影响。为了比较70℃与80℃的影响有无差别，在这两个温度下，分别重复做了8次试验，强力数据如下。问在70℃时的平均断裂强力与80℃时的平均断裂强力间是否有显著差别？ 假定断裂强力服从正态分布(α=0.05)
     >
     > 3. 双独立样本 t 检验：
     >
     > 甲乙两台机床加工螺丝帽，螺丝帽的半径都服从正态分布，为验证两台机床加工的螺丝帽半径是否相等，分别取两台机床加工的8、7枚螺丝帽进行测量，分别测得[20.5,19.8,19.7,20.4,20.1,20.0,19.0,19.9]\[20.7,19.8,19.5,20.8,20.4,19.6,20.2] 问两台机器生产的螺丝帽半径是否有差异（α=0.05）

   - **卡方检验**：检验两个变量之间有没有关系。属于**非参数检验**，主要是比较两个及两个以上样本率（构成比）以及两个分类变量的关联性分析。根本思想在于比较理论频数和实际频数的吻合程度或者拟合优度问题。

     > 例1：有 AB 两种药可以治疗某种疾病，问两种药物的疗效是否相同？
     >
     > 例2：探究死亡年龄和居住地、性别是否有关？

   - **F检验**：F 检验法是检验两个正态随机变量的总体方差是否相等的一种假设检验方法。

     > 例：存在两组数据，需要验证这两组数据的方差齐性。
     >
     > ```python
     > from scipy.stats import levene
     > 
     > x = [20.5, 18.8, 19.8, 20.9, 21.5, 19.5, 21.0, 21.2]
     > y = [17.7, 20.3, 20.0, 18.8, 19.0, 20.1, 20.0, 19.1]
     > 
     > f_val, p = levene(x, y)
     > print(f_val, p)
     > # 0.008805 0.926569
     > ```
     >
     > 结论：p值=0.93 大于 0.05，认为两个总体不具有方差齐性。

#### 2. A/B test AB测试 【CFA里没有，此处误入关联一下】

1. 什么是A/B测试

   在对产品进行A/B测试时，我们可以为同一个优化目标（例如优化购买转化率）制定两个方案（比如两个页面），**让一部分用户使用 A 方案，同时另一部分用户使用 B 方案**，**统计并对比不同方案的转化率、点击量、留存率等指标**，以判断不同方案的优劣并进行决策，从而提升转化率。

2. A/B测试的最佳实践

   - **确定优化目标**，例：“通过优化注册流程，将注册转化率提升20%”。
   - **分析数据**，通过数据分析，找到现有产品中可能存在的问题。
   - **提出想法**，例：“假设把注册流程中的图片校验码方式，改成短信校验码的方式，我们的注册转化率可能提升 10%”。
   - **重要性排序**，根据重要性、潜在收益、开发成本等因素对所有想法进行优先级的排序，并选择最重要的几个想法进行 A/B 测试。
   - **实施A/B测试并分析实验结果**：无效的A/B测试对于团队来说是非常宝贵的经验，避免以后再犯同样的错。而对于有效的A/B测试来说，这时可以把优胜的版本正式推送给全部用户，以实现产品用户的有效增长。
   - **迭代整个流程**，进行下一轮A/B测试

3. 未来 - 智能优化 & A/B 测试

   1. A/B 测试为顶尖的产品经理，运营推广团队提供了从优秀到卓越的必杀密器，为产品、运营和经营管理人员，有效掌握、提升了一下这些经营核心指标：

   - 点击通过率 Click-through Rate（CTR）
   - 转化率 Conversion Rate （CR）
   - 更新率 Renewal Rate
   - 跳出率 Bounce Rate
   - 平均保留率 Average Retention
   - 平均使用量（应用，手机网站，网页，App 屏幕或游戏场景上的时间），Mean Usage（Time on app, mobile web, mobile webpage, an app screen or game scene）
   - 平均每用户事物数 Average Transactions Per User
   - 净推动者指数 Net Promoter Score (NPS)
   - 客户满意率 Customer Satisfaction Rate
   - 平均每用户收入 Average Revenue Per User (ARPU)
   - 平均订单大小 Average Order Size

   2. 未来的 A/B 测试引擎一定是越来越智能，越来越自动化：智能优化引擎应该能实时持续动态对不同产品版本进行数据分析，并进行智能流量调节，无需人工干预，自动提升转化率，降低产品试错风险。比如通过强化学习智能优化引擎，自动进行版本选优和比例调整，无人工干预，保持运营水准持续提升。

#### 3. P-Value method

1. 定义
   - The **p-value** is the smallest level of significance at which the null hypothesis can be reject.
2. 判断准则：越小越拒绝
   - P-value > 显著性水平：fail to reject
   - P-value < 显著性水平：reject

#### 4. Type I and II error

1. 定义
   - Type I 错误：原假设是错误的，最后却接受了
   - Type II 错误：原假设是正确的，最后却拒绝了
2. P (I) = 显著性水平
3. power of test = 1 - P(II)
4. P(I) & P(II) 此消彼长
5. n 增大，P(I) & P(II) 同时下降

#### 5. Nonparametric test 非参数检验

1. 适用于
   - “不满足分布的假设”
   - when data are given in **ranks** 对排名的数据检验没有意义
   - 检验的不是总体的参数









