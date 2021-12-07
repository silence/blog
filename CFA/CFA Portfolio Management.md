## CFA Level I Portfolio Management

### 1. Portfolio Management: An Overview

#### 1.1 portfolio perspective

- 任何投资都要从组合的角度来看

- $\sigma_p^2  = w_1^2\sigma_1^2 + w_2^2\sigma_2^2 + 2w_1w_2\sigma_1\sigma_2\rho_{12}$

#### 1.2 Portfolio management process

- planning step
- execution step
  - Asset allocation
  - Security analysis
  - Portfolio construction 结构
- feedback step

#### 1.3 The types of investment management clients 机构投资者的特征

- Individual investors

  > DC plan: Defined Contribution，退休前每一期存的养老金是固定的

- Institutional investors

  > DB plan: Defined Benefit，退休后每一期取的养老金是固定的

  - foundation and endowment（养老保险）
  - bank
  - insurance

#### 1.4 Characteristics of different types of investors

- ![image-20210401095209040](https://raw.githubusercontent.com/silence/blog/assets/assets/20210401095318.png)
- Tolerance 容忍度

#### 1.5 Mutual funds and other forms of pooled investments

- **Mutual funds** 共同基金
  - 分为开放式基金和封闭式基金
  - 根据基金投资的产品不同，分为货币基金、股票基金、混合型基金、指数基金等等
- **Exchange traded funds** (**ETF**s)
  - 在一级市场，拿股票换基金份额
  - 在二级市场，若 < 100w，直接用**金钱**和投资者认购，> 100w，用**股票**和基金公司认购。
- Separately managed account 高净值客户
- Hedge funds
- Buyout funds
- Venture capital funds （VC：风投）
- **Comparison between Mutual funds and ETFs**
  - Expenses are lower for ETFs
  - **ETFs are constantly traded throughout the business day.**
  - For ETF dividends are paid out to the shareholders. 分红返现，mutual funds 是红利再投资
  - The minimum required investment in an ETF is usually smaller. 最小的投资认购份额，通常 ETF 更小些。
  - often cited as having **tax advantages over index mutual funds**.
- Comparison between Mutual funds and hedge funds
  - a significant amount of risk 有一定的风险
  - use of leverage and complexity 使用杠杆并且比较复杂
  - Hedge funds are exempt from (免除) many of the reporting requirements 披露的信息会比 Mutual funds 少

#### 1.6 The Asset Management Industry

- **Buy-side** firms and **sell-side** firms

  - 分为买方和卖方
  - 卖方主要是 **Broker and dealer** 券商和投行 **Investment banks** ，用来发行债券和发行股票。
  - 买方主要是机构投资者

- **Active** versus (对比) **Passive** Management

  - Active: 通过选定一个 benchmark 来投资，（主动选股）
  - Passive: 直接使用市场的 market index tracking (针对股票) 或者 market risk factor (针对债券).

- **Traditional** versus **Alternative** Asset Managers

  - **Traditional asset managers** focus on **equities** and **fixed-income** securities.
  - **Alternative asset managers** focus on asset classes such as **private equity (私募)**, **hedge funds**, **real estate (房地产)**, or **commodities (大宗商品)**.

- **Asset Management Industry Trends**

  - The market share for **passive management** has been growing over time.
  - Big data
  - Robot-Advisers (computer algorithm)

### 2. Portfolio risk and return

#### 2.1 大体框架

![image-20210411174035354](https://raw.githubusercontent.com/silence/blog/assets/assets/20210411174035.png)

#### 2.2 Return 和 Risk 的衡量

- HPR

- Average return
  - Arithmetic mean return
  - Geometric mean return
  - **Internal rate of return** （**IRR 内部收益率**）
    - When **NPV** (net present value 净现值) = 0, the discount rate
    - 深层含义：盈亏平衡点时的应得的收益率
- Other return measures

  - Gross and net return
  - Pretax nominal return
  - After-tax nominal return
  - Real return: nominal return adjusted for **inflation**.
  - Leveraged return: 杠杆收益率

- **T**ime-**w**eighted **R**ate of **R**eturn

  - 本质求的是几何平均收益率

- **M**oney-**w**eighted **R**ate of **R**eturn

  - 本质是 IRR

- The relationship between TWRR and MWRR
  - Both TWRR and MWRR are annual rates.
  - **Time-weighted return is not influenced by cash flow, but money-weighted return will be affected by cash flow.**
  - 衡量选股/业绩：TWRR
- Single asset

  ![image-20210407230912483](https://raw.githubusercontent.com/silence/blog/assets/assets/20210407230912.png)

- Two asset portfolio

  - Covariance of return
    - $Cov_{1,2}=\frac{\sum_{t=1}^{T}(R_{t,1}-\mu_1)(R_{t,2}-\mu_2)}{T} $ *(population)*
    - $Cov_{1,2}=\frac{\sum_{t=1}^{T}(R_{t,1}-\bar{R}_1)(R_{t,2}-\bar{R}_2)}{T-1} $ *(sample)*
  - Correlation of return
    - $\rho_{1,2} = \frac{Cov_{1,2}}{\sigma_1\sigma_2}$
    - $Cov_{1,2} = \rho_{1,2}\sigma_1\sigma_2$
  - Variance of return for the portfolio
    - $\begin{split} Var_{portfolio}&=\sigma_{p}^2\\ &=w_1^2\sigma_1^2+w_2^2\sigma_2^2+2w_1w_2Cov_{1,2}\\&=w_1^2\sigma_1^2+w_2^2\sigma_2^2+2w_1w_2\sigma_1\sigma_2\rho_{1,2}\end{split}$

- Variance of N-asset portfolio (**same covariance, same weight, same volatility**)

  - $\sigma_p^2=\frac{\bar{\sigma}^2}{N} + \frac{N-1}{N}\bar{Cov}$

#### 2.3 Modern portfolio theory

- **The Efficient Frontier**

  - **Risk** and **return** for different values of **correlation**

    ![image-20210408001828171](https://raw.githubusercontent.com/silence/blog/assets/assets/20210408001828.png)

  - The **Markowitz** assumptions

    - 用方差或标准差来衡量风险
    - 投资者希望在相同的 return 水平下 risk 尽可能的小

  - Minimum variance frontier

    - Minimum-variance portfolio 在 return 一定的情况下 risk 最小的那种 portfolio 的选择
    - **Minimum-variance frontier** 最小方差前沿

  - **Global minimum-variance portfolio**

    - 全局最小方差前沿点上的 portfolio，它有所有组合中的最小的方差。同时也是最小方差前沿上最左边的那个点。

  - **Efficient frontier 有效前沿**

    - Well-diversified or fully-diversified 充分分散化的投资组合

  - ![image-20210408001746286](https://raw.githubusercontent.com/silence/blog/assets/assets/20210408001746.png)

- Utility theory 效用理论

  - 投资者分类

    - Risk averse
    - Risk neutral
    - Risk seeking

  - **效用**假设为 投资带来的满足感，并假设投资者都是风险厌恶者

  - Utility function

    - 公式：$U=E(r) - \frac{1}{2}A\sigma^2$
    - U：The utility of an investment 投资的效用
    - E(r)：The expected return
    - $\sigma^2$：The variance of the investment
    - A：风险厌恶系数
      - Risk-aversion: A > 0
      - Risk-neutral: A = 0
      - Risk-seeking: A < 0

  - Indifference curve 无差异曲线

    - 一条无差异曲线上的 Utility 是相等的

    - 不同投资者类型的无差异曲线，重点是 `risk aversion` 的无差异曲线

      ![image-20210408001250427](https://raw.githubusercontent.com/silence/blog/assets/assets/20210408001250.png)

  - Convexity 凸的

  - Concave 凹的
  
- **Optimal portfolio**

  - 主观世界的最高的**无差异曲线**和客观的**有效前沿**的**切点**
  - **Tangent: 切线**

#### 2.4 Capital market theory

- **CAL (Capital allocation line) 资产配置线**

  - Two-fund separation theory 两基金分离定律
  - **Risk free** + **optimal risky asset** 无风险资产 + 有效前沿上的最优风险资产
  - Optimal CAL 
    - 与有效前沿相切的 CAL 线称为最优 CAL 线
    - 相切的切点称为 `The optimal risky asset portfolio.`
  - **CAL 线表达式**
    - $E(R_p) = R_f + \frac{R_A-R_f}{\sigma_A}·\sigma_p$

- **CML(Capital market line) 资本市场线**
  - CAL 与 CML 的区别
  
    - CAL 有无数条，CML 只有一条
    - CML 的假设：投资者的预期都相同
  
  - Market portfolio
  
    - CML 和 EF 的**切点**
    - 包含所有的 risky asset
    - 按市值权重组成
  
  - 公式
  
    - $E(R_p) = R_f + \frac{R_M-R_f}{\sigma_A}·\sigma_p$
    - 推论
      - 斜率是市场组合的 Sharpe ratio
      - **CML 线上所有点的 Sharpe ratio = 市场组合的 Sharpe ratio**
  
  - 作用：资产配置
  
    - **Passive** investment strategy
    - 都是有效的组合，只有系统性风险，没有非系统性风险
    - **Leveraged** investment 可以以无风险利率借钱投资，所以是 `leveraged` 的


  - **lending** portfolio & **borrowing** portfolio

    ![image-20210411173417381](https://raw.githubusercontent.com/silence/blog/assets/assets/20210411173417.png)

    - 若投资点位于 `Market portfolio` 左侧，则属于，一部分💰投资的是无风险资产（相同于把💰 `lend` 给银行来投资），一部分投资的组合资产。
    - 若投资点位于 `Market portfolio` 右侧，则属于，所有的💰都投资的是风险资产，并且还向银行以无风险利率 `borrow` 了无风险资产来投资。

#### 2.5 CAPM

- 风险的分类
  - Nonsystematic risk ( or idiosyncratic, diversifiable, company-specific risk ) 非系统性风险
    - 非系统性风险能被投资组合所分散化
  - Systematic risk ( or non-diversifiable, **market risk** ) 系统性风险
    - 系统性风险不能被分散化
    - Total risk = systematic risk + nonsystematic risk
    - 充分分散化的组合作定价的时候，只给系统性风险作补偿，因为它没有非系统性风险。
- $\beta$
  - 它是对系统性风险的一个衡量
  - 公式：$\beta_i = \frac{Cov_{i,mkt}}{\sigma_{mkt}^2} = (\frac{\sigma_i}{\sigma_{mkt}}\times\rho_{i,mkt})$
- Security market line ( SML ): Graphical representation of CAPM
  - 公式：$E(R_i) = R_f+\beta_i[E(R_m)-R_f]$
  - 是系统性风险的补偿
  - 斜率 = $R_m-Rf$
  - 位于 SML 线上的任何资产，$SR_i = SR_m · \rho_{i·m}$ , SR 指 Sharp Ratio
  - 作用：**定价**
    - 系统性风险补偿的合理收益率
  - 判断价格高估还是低估
    - **收益率和价格呈反比**

#### 2.6 Portfolio performance

- 4种衡量方法的计算
  - Sharpe ratio
    - 公式：$SR_p = \frac{R_p-R_f}{\sigma_p}$
    - 需要同别的组合的 SR 进行比较才能得知目前的 SR 值高低
    - 衡量的是总风险
  - M-squared
    - 公式：$(SR_p - SR_m)·\sigma_m$
  - Treynor ratio
    - 公式：$TR_p = \frac{R_p-R_f}{\beta_p}$
  - Jensen's alpha
    - 公式：$\alpha_p = Rp - [R_f+\beta_p(R_m-R_f)]$ ，中括号内为 CAPM 合理 R
- 比较
  - SR 和 M^2^ 是衡量的 total risk
  - TR 和 α 是衡量的系统性风险
  - M^2^ 和 α 可以直接判断
  - SR 和 TR 需要同别的进行比较判断

### 3. IPS (Investment policy statement) 投资规划书

#### 3.1 写 IPS 的目的

- 最高纲领
- 秋后算账

#### 3.2 Major components

- **重点**：附录里记录了所有 asset allocation 的相关信息

#### 3.3 Objective & constraints

- Objective
  - risk
    - willingness
    - ability
  - return
    - Consistent with risk objective
- Constraints
  - Liquidity
  - Time horizon
  - Tax conserns
  - Legal and reguatory factors
  - Unique circumstances

#### 3.4 Asset Allocation

- Strategic (战略性的) asset allocation
  - 大类别资产配置：例如股票、债券等。
- Tactical (战术性的) asset allocation
  - 在类别下进行资产配置：例如中行、云南白药等。
- Environmental, social, and governance (ESG) Considerations in Portfolio Planning and Construction
  - 招股说明书里的监管、环境、社会责任等。

### 4. Risk Management

#### 4.1 名称解释

- Risk
  - Risk management 不是指去最小化风险，指的是在一定的容忍程度的风险水平下，使收益或者效益最大化等。
  - Risk exposure 风险敞口
    - The extent to which an entity's value may be affected through sensitive to underlying risks.

#### 4.2 The Risk Management Framework in an Enterprise Context

![image-20210815154658506](https://raw.githubusercontent.com/silence/blog/assets/assets/20210815154658.png)

- Risk governace
  - Risk governace is the foundation for risk management.
- Risk tolerance
  - establish the organization's risk appetite.
- Risk budgeting

#### 4.3 Types of Risks

- Financial risks
  - Market risk
  - Credit risk
  - Liquidity risk
    - Liquidity risk could also be called transaction cost risk and is most associated with **a widening bid-ask spread**.
- Non-financial risks
  - Operational risk
  - Solvency risk

 #### 4.4 Risk metrics

- Standard deviation or volatility (波动性)
- beta (股票的系统性风险)、duration (久期)
- Derivative measures (衍生品衡量，如 delta、gamma、vega、rho...)
- **Tail risk**
  - **Value at risk (VaR)**
  - e.g. A VaR of \$100 at 1% for one day means it is expected to lose a minimum of \$100 in one day 1% of the time. 一天之内有 1% 的概率至少损失 100 块钱。
  - **Conditional VaR (CVaR)** is the **weighted average** of all loss outcomes in the statistical distribution that exceed the VaR loss.

#### 4.5 Risk modification

- Risk prevention and avoidance
- Risk acceptance 
  - Self-insurance: sufficient capital (充足的资本用来自保)
  - diversification: 多样化以降低风险
- Risk transfer (insurance)
- Risk shifting (derivatives): 使用衍生品来对冲 (hedge)

### 5. Technical Analysis

#### 5.1 技术分析理论部分

- priniples: 由供需决定的
- Assumption: 技术分析认为历史数据是有意义的、有效市场假说理论不完全成立

- The differences among technicians and fundamentalists
  - Fundamental analysis 关注内在价值（即通过财务报表或者其他数据分析出来的当前价值）
  - Technicians analysis 关注市场价值（即供需、市场价格和交易量等数据）
- 技术分析的优点：
  - It can be applied to the price of assets that do not produce future cash flows, such as commodities (如黄豆、大米价格)

#### 5.2 types of charts 

- line chart
- bar chart
- candlestick chart (蜡烛图)

#### 5.3 trend

- uptrend
- downtrend

#### 5.4 Technical analysis indicators

- price-based
  - moving average lines (MA 移动平均线)
  - Bollinger bands (布林格区间)
    - Moving average +/- 2$\sigma$
- momentum oscillator
  - rate of change oscillator
    - $M = (V-V_x) \times 100$
    - V = last closing price.
    - Vx = closing price x days ago, typically 10 days.
  - relative strength index (RSI)
    - RSI = 上涨天数 / 总天数
  - Stochastic Oscillator
  - Moving Average Convergence/Divergence (MACD) 平滑异同移动平均线
- sentiment (情绪性指标)
  - Put/call ratio (put 看跌期权、call 看涨期权)
  - volatility index 波动率指数/恐慌性指数
  - Margin debt (融资融券，即借钱去买股票)
  - short interest ratio (short sale 做空，ratio 指的是做空的人数)
- flow of funds
  - short-term trading index (下跌的量/上涨的量)
  - mutual fund cash position (基金的现金位置)
  - new equity issuance (新股发行 IPO)

#### 5.5 Elliott wave theory

### 6. Fintech in Investment Management

#### 6.1 Fintech 定义

- 狭义：金融服务和产品设计和交付过程中的科技创新

 #### 6.2 Big Data

- The characteristics of Big Data
  - Volume (very large)
  - Velocity (real-time or near-real-time)
  - Variety (mainly unstructured)
- 来源
- Data Science
  - Data processing (Method)
  - Data visualization

#### 6.3 Advanced analytical tools

- Artificial Intelligence (AI)
- Machine Learning (ML)
  - 分类
    - Supervised learning
    - Unsupervised learning
  - challenges
    - overfitting and underfitted
    - black box

#### 6.4 Fintech application in investment management

- Text analytics 
  - NLP
- Robo-adviser services 智能投顾
- Risk analysis
- Algorithmic trading

#### 6.5 Distributed Ledger Technology (DLT)

- DLT application

  - rytocurrencies
  - Tokenization
  - Post-trade clearing and settlement
  - Compliance (合规)

  









