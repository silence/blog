## CFA Level I Portfolio Management

### 1. Portfolio Management: An Overview

#### 1. portfolio perspective

- 任何投资都要从组合的角度来看

- ![\sigma_p^2  = w_1^2\sigma_1^2 + w_2^2\sigma_2^2 + 2w_1w_2\sigma_1\sigma_2\rho_{12}](https://render.githubusercontent.com/render/math?math=%5Csigma_p%5E2%20%20%3D%20w_1%5E2%5Csigma_1%5E2%20%2B%20w_2%5E2%5Csigma_2%5E2%20%2B%202w_1w_2%5Csigma_1%5Csigma_2%5Crho_%7B12%7D)

#### 2. Portfolio management process

- planning step
- execution step
  - Asset allocation
  - Security analysis
  - Portfolio construction 结构
- feedback step

#### 3. The types of investment management clients 机构投资者的特征

- Individual investors

  > DC plan: Defined Contribution，退休前每一期存的养老金是固定的

- Institutional investors

  > DB plan: Defined Benefit，退休后每一期取的养老金是固定的

  - foundation and endowment（养老保险）
  - bank
  - insurance

#### 4. Characteristics of different types of investors

- ![image-20210401095209040](https://raw.githubusercontent.com/silence/blog/assets/assets/20210401095318.png)
- Tolerance 容忍度

#### 5. Mutual funds and other forms of pooled investments

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

#### 6. The Asset Management Industry

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

#### 1. 大体框架

![image-20210405180916427](https://raw.githubusercontent.com/silence/blog/assets/assets/20210405180916.png)

#### 2. Return 和 Risk 的衡量

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
    - ![Cov*{1,2}=\frac{\sum*{t=1}^{T}(R*{t,1}-\mu_1)(R*{t,2}-\mu*2)}{T} ](https://render.githubusercontent.com/render/math?math=Cov*%7B1%2C2%7D%3D%5Cfrac%7B%5Csum*%7Bt%3D1%7D%5E%7BT%7D(R*%7Bt%2C1%7D-%5Cmu_1)(R*%7Bt%2C2%7D-%5Cmu*2)%7D%7BT%7D%20) *(population)\_
    - ![Cov*{1,2}=\frac{\sum*{t=1}^{T}(R*{t,1}-\bar{R}\_1)(R*{t,2}-\bar{R}\_2)}{T-1} ](https://render.githubusercontent.com/render/math?math=Cov*%7B1%2C2%7D%3D%5Cfrac%7B%5Csum*%7Bt%3D1%7D%5E%7BT%7D(R*%7Bt%2C1%7D-%5Cbar%7BR%7D%5C_1)(R*%7Bt%2C2%7D-%5Cbar%7BR%7D%5C_2)%7D%7BT-1%7D%20) _(sample)_
  - Correlation of return
    - ![\rho_{1,2} = \frac{Cov_{1,2}}{\sigma_1\sigma_2}](https://render.githubusercontent.com/render/math?math=%5Crho_%7B1%2C2%7D%20%3D%20%5Cfrac%7BCov_%7B1%2C2%7D%7D%7B%5Csigma_1%5Csigma_2%7D)
    - ![Cov_{1,2} = \rho_{1,2}\sigma_1\sigma_2](https://render.githubusercontent.com/render/math?math=Cov_%7B1%2C2%7D%20%3D%20%5Crho_%7B1%2C2%7D%5Csigma_1%5Csigma_2)
  - Variance of return for the portfolio
    - ![\begin{split} Var_{portfolio}&=\sigma_{p}^2\\ &=w_1^2\sigma_1^2+w_2^2\sigma_2^2+2w_1w_2Cov_{1,2}\\&=w_1^2\sigma_1^2+w_2^2\sigma_2^2+2w_1w_2\sigma_1\sigma_2\rho_{1,2}\end{split}](https://render.githubusercontent.com/render/math?math=%5Cbegin%7Bsplit%7D%20Var_%7Bportfolio%7D%26%3D%5Csigma_%7Bp%7D%5E2%5C%5C%20%26%3Dw_1%5E2%5Csigma_1%5E2%2Bw_2%5E2%5Csigma_2%5E2%2B2w_1w_2Cov_%7B1%2C2%7D%5C%5C%26%3Dw_1%5E2%5Csigma_1%5E2%2Bw_2%5E2%5Csigma_2%5E2%2B2w_1w_2%5Csigma_1%5Csigma_2%5Crho_%7B1%2C2%7D%5Cend%7Bsplit%7D)

- Variance of N-asset portfolio (**same covariance, same weight, same volatility**)

  - ![\sigma_p^2=\frac{\bar{\sigma}^2}{N} + \frac{N-1}{N}\bar{Cov}](https://render.githubusercontent.com/render/math?math=%5Csigma_p%5E2%3D%5Cfrac%7B%5Cbar%7B%5Csigma%7D%5E2%7D%7BN%7D%20%2B%20%5Cfrac%7BN-1%7D%7BN%7D%5Cbar%7BCov%7D)

#### 3. Modern portfolio theory

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

    - 公式：![U=E(r) - \frac{1}{2}A\sigma^2](https://render.githubusercontent.com/render/math?math=U%3DE(r)%20-%20%5Cfrac%7B1%7D%7B2%7DA%5Csigma%5E2)
    - U：The utility of an investment 投资的效用
    - E(r)：The expected return
    - ![\sigma^2](https://render.githubusercontent.com/render/math?math=%5Csigma%5E2)：The variance of the investment
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

