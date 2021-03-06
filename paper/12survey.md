<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

# An Experimental Evaluation of Pointofinterest Recommendation in Locationbased Social Networks
## Point-of-Interest(POI)系统面临的挑战
1. 丰富的上下文信息
- 用户在一定地理范围内运动
- 用户容易重复访问地点
- 用户行为与时间段有关联
- 用户行为收到社交群体影响
2. 数据缺失问题
- 被访问POI只占很少的比例

## POI推荐
1. 空间影响（spatial influence）
2. 社交影响（social influence）
3. 时间影响（temporal influence）

## 评估模型
- 四种推荐技术
- 五种上下文信息

### 矩阵分解模型
与矩阵分解类似，在共现矩阵看作用户向量与POI向量（而非Item向量）乘积的结果。目标方程为：

$$
\min _{\mathbf{U}, \mathbf{L}}|| \mathbf{C}-\mathbf{U} \mathbf{L}^{\top}\left\|_{F}^{2}+\lambda_{1}\right\| \mathbf{U}\left\|_{F}^{2}+\lambda_{2}\right\| \mathbf{L} \|_{F}^{2}
$$

1.LRT
加入了时间因素，将用户因向量再分为24小时中不同的向量，并在目标方程中增加时间相似度的函数

$$
\sum_{t=1}^{T} \sum_{i=1}^{m} \psi_{i}(t, t-1)\left\|\mathbf{u}_{i}^{(t)}-\mathbf{u}_{i}^{(t-1)}\right\|_{2}^{2}
$$

2.IRenMF
基于加权矩阵分解，背后的原理：
- 在相邻POIs中用户行为相似（本地级别）
- 在相同地区用户行为相似（地区级别）
除了考虑用户与POI的关系，也要考虑与POI邻居们的关系，POI关系前的参数alpha即是权重

$$
\hat{C}_{i j}=\alpha \mathbf{u}_{i} \mathbf{l}_{j}^{\top}+(1-\alpha) \frac{1}{Z\left(l_{j}\right)} \sum_{l_{l_{k} \in \mathcal{N}\left(l_{i}\right)}} \operatorname{Sim}\left(l_{j}, l_{k}\right) \mathbf{u}_{i} \mathbf{l}_{k}^{\top}
$$

3.GeoMF
基于加权矩阵分解,GeoMF通过对用户活动区域的建模和对地理空间的影响传播来整合地理影响。
地理影响：地区被分为若干区域，某个区域中的POI会吸引周围区域中的用户
所以目标函数除了考虑**用户向量**和**POI向量**，同时也要考虑**用户活动区域向量X**和**POI影响向量Y**

$$
\hat{C}_{i j}=\mathbf{u}_{i} \mathbf{l}_{j}^{\top}+\mathbf{x}_{i} \mathbf{y}_{j}^{\top}
$$

4.RankGeoFM
基于排名的矩阵分解模型：
- 为POIs学习用户喜好排名，U1
- 包含了邻居POIs的地理影响，U2

$$
\hat{C}_{i j}=\mathbf{u}_{i}^{(1)} \mathbf{l}_{j}^{\top}+\mathbf{u}_{i}^{(2)}
$$

5.ASMF
ASMF是一个两步POI推荐框架
- 学习用户潜在朋友
- 将潜在地点包含到加权矩阵分解中解决冷启动问题
（1）社交影响：每个用户在每个潜在地点有三种朋友，社交朋友、地点朋友、邻居朋友
（2）分类影响：计算推荐分数时，ASMF根据用户对某地区隶属分类的喜爱程度作为权重计算目标函数$$\hat{C}_{i j}=\left(Q_{i c_{j}}+\epsilon\right) \mathbf{u}_{i}\mathbf{l}_{j}^{\top}$$(3）地理影响：将距离加入到目标函数计算中$$\hat{C}_{i j} \propto p_{i j}^{G} \times \hat{C}_{i j}$$

### 泊松因子模型（Poisson Factor Models）
Poisson Factor Model (PFM)是一个概率模型，将用户和POI相乘的共现矩阵以泊松分布的形式展现。

1.MGMPFM
MGMPFM是一个融合模型，由PFM的输出和地理模型Multi-center Gaussian Model (MGM)
（1）地理影响：根据用户活动区域（学校，家），MGM学习到**多重高斯分布**
（2）推荐：根据区域R计算POI的分布，再计算POI与用户共现矩阵的分布

$$
P_{i j}=\&P\left(C_{i j}\right) \cdot P\left(l_{j}\left|R_{i}\right|\right)
$$

2.GeoPFM
- 同时考虑用户的地理偏好和兴趣偏好
- 潜在地理位置用二维高斯分布被整合到PFM模型中
- 用户则表现为一个位置中的多项分布

### 基于链接的模型
LFBCA使用图链接用户和关系。在图中用户喜好和社交影响展现为不同的边。

### 混合模型（Hybrid Models）
混合模型综合了不同的方法，方法分别对用户偏好或者一种上下文信息建模。

1.USG
- 同步对用户偏好，社交影响和地理影响建模，然后推荐POI
- 用户偏好：基于用户的协同过滤
- 社交影响：基于朋友的协同过滤 Friend-based Collaborative Filtering (FCF)
- 地理影响：用户访问某个POI的几率基于用户与该POI在一定距离内的其他POI的访问历史

2.iGSLR
iGSLR挖掘地理偏好和社交因素
- 社交影响：基于朋友的协同过滤，朋友的定义为居住地相近
- 地理影响：学习用户访问距离的分布，Kernel Density Estimation(KDE).

3.LORE
除了社交和地理因素，还考虑连续动作影响
- 社交影响：FCF
- 地理影响：2维KDE分布
- 连续行动：使用additive Markov chain (AMC)挖掘POI间联系。个别用户再POI之间的动作概率服从总体用户的动作规律

4.GeoSoCa
对社交，地理以及分类进行建模。
- 地理影响：2-维 KDE。与LORE不同的是，参数sigma并非被所有用户共享，而是用户独有
- 社交影响：用户的朋友登记到POI几率服从幂律分布，将其累积分布作为社交影响
- 分类影响：用户对于某个分类访问几率服从幂律分布，其累积分布作为分类影响

[返回首页](https://666cocohappy.github.io/note/)
