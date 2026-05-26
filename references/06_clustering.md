# 阶段六：聚类（无监督补充视角）/ Clustering

> 从"预测"切换到"分群"的视角。把记录（客户、门店、传感器、用户等）按其画像自然分成几类，支持差异化决策。是有监督建模的有力补充，也是商业分析复用率极高的技能。
>
> 输出语言提醒：全程使用用户的语言（含图表）。

## 目录 / Contents
1. 聚类的定位与价值
2. 构造对象画像特征
3. 确定聚类数 K（肘部法则）
4. 执行 KMeans 并解读
5. ⭐ 高维聚类的正确可视化（PCA + 热力图）
6. 从聚类到差异化策略

---

## 1. 聚类的定位与价值

聚类是**无监督**任务（无既定目标），换一个视角回答："这些对象能否按特征自然分成几类？" 价值在于：
- **用户/客户分层**（如 RFM：最近一次、频次、金额）→ 差异化营销。
- **门店/站点分群** → 分类施策、功能区划分。
- 与有监督模型形成"预测 + 分群"的完整闭环，叙事更顺。

## 2. 构造对象画像特征

按对象聚合，为每个对象计算刻画其特性的指标。

```python
# 通用模式：按对象ID聚合出画像
profile = df.groupby('对象ID').agg(
    指标1=('列A', 'mean'),
    指标2=('列B', 'std'),
    指标3=('目标', 'mean'),
).reset_index()
# 可派生更多: 如时间型对象的"昼夜差""波动性"等
```

**RFM 模型**（经典用户价值模型）：Recency（最近一次距今天数）、Frequency（频次）、Monetary（金额/价值）。

## 3. 确定聚类数 K（肘部法则）

尝试不同 K，画"簇内平方和(inertia)随 K 变化"曲线，拐点(肘部)处的 K 通常合理。KMeans 对量纲敏感，**必须先标准化**。

```python
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans
X_cluster = StandardScaler().fit_transform(profile[cluster_feats])
inertias = [KMeans(n_clusters=k, random_state=42, n_init=10).fit(X_cluster).inertia_ for k in range(2,8)]
# 画 inertias 折线图找拐点
```

## 4. 执行 KMeans 并解读

用选定的 K 聚类，计算每类的特征均值，给出有业务含义的命名。

```python
km = KMeans(n_clusters=3, random_state=42, n_init=10)
profile['聚类'] = km.fit_predict(X_cluster)
cluster_profile = profile.groupby('聚类')[cluster_feats].mean()   # 各类画像
```

## 5. ⭐ 高维聚类的正确可视化（PCA + 热力图）

**这是聚类中最容易出错、也最能体现专业度的地方。** 聚类常用多个维度（如 5 个特征），但简单的二维散点图只能展示 2 个维度——**会产生误导**：某些点在"看得见的 2 维"上看似归属异常，实则在"看不见的维度"上与所属类一致。

**解法一：PCA 降维散点图**。用主成分分析把多维压缩成 2 维（前两个主成分综合了全部特征信息），此时图上的远近才真正对应聚类所用的距离。检查累计解释方差（>85% 说明二维图能较好代表原结构）。
```python
from sklearn.decomposition import PCA
pca = PCA(n_components=2); coords = pca.fit_transform(X_cluster)
print('累计解释方差:', pca.explained_variance_ratio_.sum())
# 用 coords 作散点图,按聚类着色
```

**解法二：聚类特征热力图**。直接展示每个聚类在各维度上的（标准化）均值，颜色深浅一眼读出每类的特征画像，是解读聚类含义最清晰的方式。
```python
import seaborn as sns
cluster_mean = profile.groupby('聚类')[cluster_feats].mean()
cluster_mean_z = (cluster_mean - cluster_mean.mean()) / cluster_mean.std()
sns.heatmap(cluster_mean_z.T, annot=cluster_mean.T.round(2), fmt='g', cmap='RdBu_r', center=0)
```

**运行案例（城市噪音）**：15 个监测点按 5 维画像聚成 3 类。某监测点在"平均声级×超标率"二维图上看似混入了另一类，但 PCA 图（保留 93% 方差）显示它通过第二主成分正确归位——因为它在"波动性、昼夜差"这两个隐藏维度上与所属类一致。热力图则清晰显示该类在这两维上数值最高。**务必用 PCA 图而非简单二维散点图展示聚类结果。**

## 6. 从聚类到差异化策略

聚类的落点是**可落地的差异化决策**。为每类对象命名（如"高价值型""流失风险型""安静达标型"），并给出对应策略。这种"按类型分类施策"的思路，把数据分析直接转化为管理/运营对策，是报告/论文中管理建议章节的有力支撑。
