# 阶段五：评估、验证与诊断 / Evaluation, Validation & Diagnosis

> 把模型从"跑分漂亮"提升到"经得起审视"。包含正确的指标选择、交叉验证、调优，以及最体现严谨性的——**残差诊断主动暴露缺陷**。
>
> 输出语言提醒：全程使用用户的语言。

## 目录 / Contents
1. 分类评估指标（不平衡场景）
2. 回归评估指标
3. K 折交叉验证（检验稳定性）
4. 网格搜索调优
5. ⭐ 残差诊断（主动暴露缺陷）
6. ⭐ 数据驱动阈值发现

---

## 1. 分类评估指标（不平衡场景重点）

**为什么不能只看准确率**：不平衡数据上"全猜多数类"也能高准确率却无用。要看：
- **AUC**（ROC 曲线下面积）∈[0.5,1]：区分正负样本的综合能力。可解释为"随机取一正一负样本，模型给正样本更高分的概率"。
- **PR 曲线**（精确率 vs 召回率）：更关注少数类，不平衡数据更看重。精确率 = TP/(TP+FP)，召回率 = TP/(TP+FN)。
- **混淆矩阵 + 分类报告**：看模型把哪些分对/分错，关注少数类的 precision 与 recall 权衡。

```python
from sklearn.metrics import roc_auc_score, average_precision_score, classification_report, confusion_matrix, roc_curve, precision_recall_curve
auc = roc_auc_score(y_test, proba); ap = average_precision_score(y_test, proba)
# ROC 曲线、PR 曲线、混淆矩阵均可作图对比多个模型
```

## 2. 回归评估指标

- **R²（决定系数）**∈(-∞,1]：解释了多少目标方差，越接近 1 越好。R² = 1 − SS_res/SS_tot。
- **RMSE（均方根误差）**：误差的原始单位数值，对大误差敏感。RMSE = √(mean((y−ŷ)²))。
- **MAE（平均绝对误差）**：平均预测偏离多少，更直观。MAE = mean(|y−ŷ|)。
- 始终与"预测均值"基线对比，报告误差降低百分比。

## 3. K 折交叉验证（检验模型稳定性）

**为什么**：单次训练/测试划分可能是"幸运/不幸"的偶然。K 折把训练数据均分 K 份，轮流用 K-1 份训练、1 份验证，重复 K 次取平均。**K 个分数的标准差反映稳定性**——标准差越小越可靠。分类用分层 K 折（StratifiedKFold）保持类别比例。

```python
from sklearn.model_selection import StratifiedKFold, cross_val_score
cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(clf, X_train, y_train, cv=cv, scoring='roc_auc', n_jobs=-1)
print(f'平均 CV-AUC: {scores.mean():.4f} (±{scores.std():.4f})')
```

## 4. 网格搜索调优

**为什么**：让超参数"搜出来"而非"猜出来"。GridSearchCV 遍历参数组合，每组用 K 折评分，选最优。搜索空间要精简（避免组合爆炸）。

```python
from sklearn.model_selection import GridSearchCV
param_grid = {'n_estimators':[200,300], 'num_leaves':[31,63], 'learning_rate':[0.05,0.1]}
grid = GridSearchCV(base_model, param_grid, scoring='roc_auc', cv=cv, n_jobs=-1)
grid.fit(X_train, y_train)
print(grid.best_params_, grid.best_score_)
```

**诚实提醒**：若调优后提升甚微，要如实报告——这往往说明"模型对超参数不敏感、表现源于特征质量而非调参"，这本身是有价值的正面结论，比硬凑"提升了X%"更可信。

## 5. ⭐ 残差诊断（主动暴露模型缺陷）

**这是从"跑分漂亮"到"严肃研究"的关键一步。** 一个高 R² 的模型可能在特定区间系统性失准——汇总指标会掩盖它。残差诊断主动去查"模型在哪里失灵"。

三个检验（针对回归）：
- **向均值收缩检验**：看"残差 vs 实际值"的斜率。显著为负 = 高值低估、低值高估（强正则化模型的典型副作用）。理想斜率≈0。
- **正态性检验**：残差若不服从正态（Shapiro-Wilk 检验 p<0.05），说明仍有未捕捉的结构。
- **分组残差**：按目标高/中/低分组看平均残差，定位最不准的区间。

```python
from scipy import stats
residuals = y_test.values - pred
slope, *_ = stats.linregress(y_test.values, residuals)
print(f'向均值收缩斜率: {slope:.3f}')   # 越接近0越好
# 分组残差: pd.qcut 分高中低三组, 看各组平均残差
# 可视化: 残差vs预测值散点图 + Q-Q图
```

**为什么重要**：例如一个 R²=0.97 但"专门在高值区低估"的模型，用于预警时恰好漏掉最该预警的极端情况。诊断能让你发现并在使用时留出安全裕度。**不同模型暴露的缺陷不同**：强正则化线性模型常见向均值收缩；充足样本的树模型则近似无偏。这种对照本身很有价值。

## 6. ⭐ 数据驱动阈值发现

**精神**：用数据驱动的切分点（决策树最优分裂点）替代拍脑袋的人工阈值。决策树会自动寻找使目标区分度最大的切分位置——即数据真实的"拐点"。

```python
from sklearn.tree import DecisionTreeClassifier
dt = DecisionTreeClassifier(max_depth=2, min_samples_leaf=500, random_state=42).fit(df[[var]], df['目标'])
thresholds = sorted([dt.tree_.threshold[i] for i in range(dt.tree_.node_count) if dt.tree_.children_left[i]!=-1])
```

**关键洞察（务必理解）**：树模型（LightGBM/随机森林）在训练时**本就会自动找最优分裂点**，所以数据驱动阈值对树模型几乎无用（人工分箱是冗余的）；但对**线性模型**显著有用（线性模型不会自动找拐点，喂对阈值等于补上它学不到的非线性）。

**所以**：数据驱动阈值的价值取决于模型类型——它揭示了"特征工程的价值取决于模型类型"这一深刻规律。对树模型，保留原始连续特征即可；对线性模型，数据驱动分箱价值巨大。此外，发现的阈值本身还是很好的**业务解读工具**（如"某指标超过 X 即进入高风险区"）。
