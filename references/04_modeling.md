# 阶段四：建模（分类与回归）/ Modeling (Classification & Regression)

> 把特征工程的成果变成"可预测能力"。覆盖分类与回归两类任务，采用"基线→集成"的递进策略。
>
> 输出语言提醒：全程使用用户的语言。

## 目录 / Contents
1. 确定任务类型
2. 划分训练/测试集（防泄露）
3. 基线→集成的递进策略
4. 分类建模
5. 回归建模
6. 类别不平衡处理

---

## 1. 确定任务类型

- **目标是类别**（是/否、几个等级）→ **分类**。
- **目标是连续数值**（金额、分贝、温度）→ **回归**。
- 一份数据可能同时支持两种（如既预测"是否超标"分类，又预测"声级值"回归），可都做以覆盖两类任务。

## 2. 划分训练/测试集（防泄露）

**为什么**：模型不能在"见过的数据"上评估，否则高估性能。

```python
from sklearn.model_selection import train_test_split
# 分类用 stratify 保持类别比例一致
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, random_state=42, stratify=y)   # 回归任务去掉 stratify
```

**时序数据的特殊划分**：若是预测未来，必须按时间切分（用过去训练、预测未来），绝不能随机划分——否则用未来信息预测过去，是时序泄露。例：用前 7 天行为构造特征，预测后 2 天是否发生某事件。

## 3. 基线→集成的递进策略

**为什么要多模型递进对比**：先用简单模型建立性能下限(baseline)，再上复杂模型，证明复杂度确实带来提升、而非过拟合或噱头。这是严谨建模的标准做法。

顺序：**线性模型（基线）→ 随机森林 → 梯度提升（主力）**。

## 4. 分类建模

**基线：逻辑回归**。简单、可解释、训练快。对量纲敏感需先标准化。
```python
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import roc_auc_score
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)   # 只在训练集 fit，防泄露
X_test_s  = scaler.transform(X_test)
lr = LogisticRegression(max_iter=1000, class_weight='balanced', random_state=42)
lr.fit(X_train_s, y_train)
lr_auc = roc_auc_score(y_test, lr.predict_proba(X_test_s)[:,1])
```

**随机森林**：多棵树的 Bagging 集成，自动捕捉非线性与交互，对量纲不敏感。
```python
from sklearn.ensemble import RandomForestClassifier
rf = RandomForestClassifier(n_estimators=200, max_depth=12,
                            class_weight='balanced', random_state=42, n_jobs=-1)
rf.fit(X_train, y_train)
```

**主力：LightGBM**（梯度提升树）。表格数据上通常表现最佳。
```python
import lightgbm as lgb
spw = (y_train==0).sum() / (y_train==1).sum()   # 不平衡处理
clf = lgb.LGBMClassifier(n_estimators=300, learning_rate=0.05, num_leaves=31,
        subsample=0.8, colsample_bytree=0.8, scale_pos_weight=spw,
        random_state=42, n_jobs=-1, verbose=-1)
clf.fit(X_train, y_train, eval_set=[(X_test, y_test)], eval_metric='auc',
        callbacks=[lgb.early_stopping(30, verbose=False)])   # 提前停止防过拟合
```

## 5. 回归建模

**基线：预测均值 + 线性回归**。任何回归模型都必须打败"预测均值"这个最弱基线。
```python
import numpy as np
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error, r2_score
baseline_rmse = np.sqrt(mean_squared_error(y_test, np.full(len(y_test), y_train.mean())))
linr = LinearRegression().fit(X_train, y_train)
```

**主力：LightGBM 回归**。原理同分类，损失换成回归用的均方误差。
```python
reg = lgb.LGBMRegressor(n_estimators=400, learning_rate=0.05, num_leaves=31,
        subsample=0.8, colsample_bytree=0.8, random_state=42, n_jobs=-1, verbose=-1)
reg.fit(X_train, y_train, eval_set=[(X_test, y_test)], eval_metric='rmse',
        callbacks=[lgb.early_stopping(30, verbose=False)])
```

## 6. 类别不平衡处理

当正类占比很低（如超标率 23%、流失率 5%），模型容易"全猜多数类"也得高准确率却毫无用处。处理方式：
- **类别加权**：逻辑回归/随机森林用 `class_weight='balanced'`；LightGBM 用 `scale_pos_weight = 负样本数/正样本数`。
- **评估指标**：绝不能只看准确率，要看 AUC、PR 曲线、混淆矩阵（详见阶段五）。
- 极端不平衡（<1%）还可考虑重采样（SMOTE 等），但加权通常已足够。

**模型选择回顾**：以上以大/中样本表格数据为前提，故主力用 LightGBM。若是小样本（几十到几百行），应改用强正则化的简单模型（Ridge/逻辑回归），并严控特征数、用交叉验证——切勿在小样本上套用 LightGBM。方法服从数据规模约束。
