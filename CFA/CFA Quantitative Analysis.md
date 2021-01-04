CFA

- 概率论统计
  - 描述性统计
  - 推断性统计
    - 由样本估计总体
    - 假设检验
- 货币的时间价值
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
- EAR => Effective annual rate
  - 单利 仅本金计息 360天
  - 复利 利息也计息 365天
- ![EAR = (1 + \frac{r}{m})^m -1](https://render.githubusercontent.com/render/math?math=EAR%20%3D%20(1%20%2B%20%5Cfrac%7Br%7D%7Bm%7D)%5Em%20-1) 
- **r** → nominal rate per year **m** → 年计息次数
- 连续复利 EAR = e^r^ -1 
- 金融计算器
  - CHN 链式运算
  - AOS 代数运算
- amortization 分期偿还 
- ordinary annuities 后付年金
- annuity due 后付年金
- 我 outflow "-" 银行 inflow "+"
- Perpetuity 永久
- 永续年金的PV = A除以r

**Holding Period Return【HPR】 持有期收益率**

R = **收益** 除以 成本 (**不是一个年化的收益率**)

**收益**

- Capital gain 资本利得
- Income 

T-bill 

- Treasury 国债
  - 小于等于 1 年 T-bill **零息债券**
  - 1 到 10年 T-note
  - 大于10年 T-bond

Descriptive statistics 描述统计

Inferential statistics 推断统计

Population 总体

sample statics 样本统计量

Types of measurement scales: 

- Nomial scales
- Ordinal scales < >
- Interval scales < > + - 
- Ratio scales < > + - * /



## mean dispersion -CV-and-SR

1. dispersion 散布
2. cumulative 累积

#### measures of central tendency 衡量中心趋势

1. mode 众数
2. Median 中位线
3. **The Arithmetic Mean ** 算术平均
4. **The Weighted Mean** 加权平均 
5. **The Geometric Mean** 几何平均
6. **The Harmonic Mean** 调和平均
7. **A ≥ G ≥ H**
8. - 算术/加权 ：预测未来
   - 几何
     - focus on ending value
     - 历史业绩
9. Quintile 五分位数

#### Measures of Dispersion 衡量离散程度

1. Range = max - min
2. MAD (**Mean Absolute deviation**)  平均绝对偏差
3. ![image-20210101180915672](https://raw.githubusercontent.com/silence/blog/assets/assets/20210101180915.png)
4. 总体 **希腊字母** 样本 **英文字母**

#### Chebyshev's Inequality  切比雪夫不等式

1. ![P(\mu - k\sigma \le x \le \mu + k \sigma)  \ge 1-\frac{1}{k^2}](https://render.githubusercontent.com/render/math?math=P(%5Cmu%20-%20k%5Csigma%20%5Cle%20x%20%5Cle%20%5Cmu%20%2B%20k%20%5Csigma)%20%20%5Cge%201-%5Cfrac%7B1%7D%7Bk%5E2%7D)

#### CV and SR

1. **Coefficient of variation**  变异系数 又称 离散系数 为 标准差 与 平均值 之比 ![CV = \frac{S}{\bar X}](https://render.githubusercontent.com/render/math?math=CV%20%3D%20%5Cfrac%7BS%7D%7B%5Cbar%20X%7D)
   - 衡量 每单位均值的离散程度 / 每单位均值的风险
   - 性质
     - Scale - free
     - Relative 相对离散程度
2. **Sharpe ratio**
   - 业绩衡量的指标
   - ![SR = \frac{R_p - R_f}{\sigma_p}](https://render.githubusercontent.com/render/math?math=SR%20%3D%20%5Cfrac%7BR_p%20-%20R_f%7D%7B%5Csigma_p%7D) P: portfolio
   - 每单位风险的超额收益
   - excess return 超额收益

#### Skewness

1. **Positve skewed** and **negative skewed**

   - 长尾在哪边就往哪边偏

   - gain 利润
   - ![S_k= [\frac{n}{(n-1)(n-2)}]\frac{\sum_{i=1}^n(X_i-\bar X)^3}{s^3}\approx(\frac{1}{n})\frac{\sum_{i=1}^n(X_i-\bar X)^3}{s^3}](https://render.githubusercontent.com/render/math?math=S_k%3D%20%5B%5Cfrac%7Bn%7D%7B(n-1)(n-2)%7D%5D%5Cfrac%7B%5Csum_%7Bi%3D1%7D%5En(X_i-%5Cbar%20X)%5E3%7D%7Bs%5E3%7D%5Capprox(%5Cfrac%7B1%7D%7Bn%7D)%5Cfrac%7B%5Csum_%7Bi%3D1%7D%5En(X_i-%5Cbar%20X)%5E3%7D%7Bs%5E3%7D)

   - 右偏 > 0
   - 左偏 < 0

#### Kurtosis

1. L > 3 N = 3 P < 3
2. power = 4
3. 高峰肥尾 假设![\sigma^2](https://render.githubusercontent.com/render/math?math=%5Csigma%5E2)和正态分布一样 

## Basic Concepts

#### Probability Concepts

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

#### Multiplicaion and Addition Rule

1. Joint probability P(AB) 联合概率

   - Multiplication rule ![P(AB) = P(A\lvert B)\times P(B) = P(B\lvert A) \times P(A)](https://render.githubusercontent.com/render/math?math=P(AB)%20%3D%20P(A%5Clvert%20B)%5Ctimes%20P(B)%20%3D%20P(B%5Clvert%20A)%20%5Ctimes%20P(A))

   - If A and B are mutually exclusive events, then : P(AB) = 0

2. Addition rule

   - ![P(A or B) = P(A) + P(B) - P(AB)](https://render.githubusercontent.com/render/math?math=P(A%20or%20B)%20%3D%20P(A)%20%2B%20P(B)%20-%20P(AB))
   - If A and B are mutually exclusive events, then : P(AB) = P(A) + P(B)

3. 互斥事件一定不是独立事件
   
4. **Expected Value** 期望

5. Earnings per share 每股收益 

#### Covariance and Correlation

1. Covariance 协方差
   - 定义：两个随机变量变化的方向性 同向 > 0 反向 < 0
   - ![Cov_{xy} = E[(x_i-\bar x)(y_i - \bar y)]](https://render.githubusercontent.com/render/math?math=Cov_%7Bxy%7D%20%3D%20E%5B(x_i-%5Cbar%20x)(y_i%20-%20%5Cbar%20y)%5D)  范围： ![-\infin \sim + \infin](https://render.githubusercontent.com/render/math?math=-%5Cinfin%20%5Csim%20%2B%20%5Cinfin)
   - ![Cov_{xx} = \sigma_x^2](https://render.githubusercontent.com/render/math?math=Cov_%7Bxx%7D%20%3D%20%5Csigma_x%5E2) **自己和自己的协方差就是方差**
   
2. Correlation 相关系数
   - ![\rho_{xy} = \frac{Cov_{xy}}{\sigma_x\sigma_y}](https://render.githubusercontent.com/render/math?math=%5Crho_%7Bxy%7D%20%3D%20%5Cfrac%7BCov_%7Bxy%7D%7D%7B%5Csigma_x%5Csigma_y%7D)
   - **Linear** relationship **线性关系**
   - 范围： ![-1\sim +1](https://render.githubusercontent.com/render/math?math=-1%5Csim%20%2B1)
   - slope 斜率 
   - 缺陷
     - 只能衡量线性关系
     - 异常值
     - 伪相关
#### Portfolio Return and Risk

1. ![\sigma_p^2  = w_1^2\sigma_1^2 + w_2^2\sigma_2^2 + 2w_1w_2\sigma_1\sigma_2\rho_{12}](https://render.githubusercontent.com/render/math?math=%5Csigma_p%5E2%20%20%3D%20w_1%5E2%5Csigma_1%5E2%20%2B%20w_2%5E2%5Csigma_2%5E2%20%2B%202w_1w_2%5Csigma_1%5Csigma_2%5Crho_%7B12%7D)
   - 若![\rho = 1](https://render.githubusercontent.com/render/math?math=%5Crho%20%3D%201)  ![\sigma_p = w_1\sigma_1 + w_2\sigma_2](https://render.githubusercontent.com/render/math?math=%5Csigma_p%20%3D%20w_1%5Csigma_1%20%2B%20w_2%5Csigma_2)
   - 若![\rho = -1](https://render.githubusercontent.com/render/math?math=%5Crho%20%3D%20-1)  ![\sigma_p = \lvert w_1\sigma_1 - w_2\sigma_2 \lvert](https://render.githubusercontent.com/render/math?math=%5Csigma_p%20%3D%20%5Clvert%20w_1%5Csigma_1%20-%20w_2%5Csigma_2%20%5Clvert)
   - 分散化效果好

#### Bayes' Formula

#### Principles of Counting

1. 乘法法则
2. 阶乘
3. 排列组合

#### Different Types of Random Variables

1. Discrete 离散
2. Continuous 连续
3. Discrete uniform distribution 离散均匀分布
4. Binomial distribution 二项式分布
   - ![C_n^xP^x(1-p)^{n-x}](https://render.githubusercontent.com/render/math?math=C_n%5ExP%5Ex(1-p)%5E%7Bn-x%7D)
   - Expectation : np
   - Variance: np(1-p)
5. Continuous Uniform Distribution 连续均匀分布
6.  Decade 十年

## Normal Distribution

1.  ![X \sim N(\mu, \sigma^2)](https://render.githubusercontent.com/render/math?math=X%20%5Csim%20N(%5Cmu%2C%20%5Csigma%5E2))

2. skewness = 0 ; kurtosis = 3

3. 正态分布的组合依然符合正态分布

4. 概率分布同k值

   - 68% k = 1

   - 90% k = 1.65
   - 95% k = 1.96
   - 99% k = 2.58

5. 标准化

   - ![\mu = 0, \sigma^2 = 1](https://render.githubusercontent.com/render/math?math=%5Cmu%20%3D%200%2C%20%5Csigma%5E2%20%3D%201)
   - If  ![X \sim N(\mu, \sigma^2)](https://render.githubusercontent.com/render/math?math=X%20%5Csim%20N(%5Cmu%2C%20%5Csigma%5E2)) ; ![\frac{X-\mu}{\sigma} \sim N(0,1)](https://render.githubusercontent.com/render/math?math=%5Cfrac%7BX-%5Cmu%7D%7B%5Csigma%7D%20%5Csim%20N(0%2C1))

#### Safety First Ratio

1. 第一安全比率
2. allocation 分配

#### Lognormal Distribution 

1. 若 ![lnx](https://render.githubusercontent.com/render/math?math=lnx) is normal 则 x is lognormal
2. Right skewed

#### Simulation

1. 蒙特卡洛模拟
2. 历史模拟



