# 阶段二：探索性数据分析 (EDA) / Exploratory Data Analysis

> 建模前彻底理解"数据里发生了什么、什么驱动了目标"。EDA 的洞察直接指导特征工程（该造什么特征）和结果解读（模型学到的规律对不对）。
>
> 输出语言提醒：图表标题、坐标轴、图例全部用用户的语言。务必先配置好中文字体（见末尾），否则中文显示为方框。

## 目录 / Contents
1. EDA 的四个分析角度
2. 目标变量分布
3. 单变量与分组分析
4. 相关性分析
5. 时间/空间/类别规律
6. ⭐ 特征与目标的关系（最有价值）
7. 中文字体配置（避免图表乱码）

---

## 1. EDA 的四个分析角度

无论什么领域的表格数据，EDA 通常从四个角度切入：
- **目标变量分布**：要预测的东西长什么样（分类的类别比例、回归的取值范围）。
- **单变量/分组分析**：各特征自身的分布，以及在不同条件下的差异。
- **相关性**：特征与目标、特征之间的关联强度。
- **领域规律**：时间规律、空间规律、类别规律等（视数据而定）。

## 2. 目标变量分布

**为什么先看目标**：决定任务类型与处理方式。
- 分类目标：看类别比例。若某类占比很低（如 <10%），是**类别不平衡**，建模和评估要特殊处理（见阶段四、五）。
- 回归目标：看取值范围、均值、是否偏态。

```python
# 分类目标
print(df['目标'].value_counts())
print(f'正类占比: {df["目标"].mean()*100:.1f}%')
# 回归目标
print(df['目标'].describe().round(2))
```

## 3. 单变量与分组分析

用分组聚合快速看"不同条件下目标的差异"，这是发现规律最快的方式。

```python
# 按某类别列分组看目标均值
df.groupby('类别列')['目标'].mean().sort_values(ascending=False)
```

可视化用柱状图（类别）、直方图（数值分布）、折线图（趋势）。

## 4. 相关性分析

量化数值特征与目标、特征之间的线性关联。Pearson 相关系数 r ∈ [-1,1]，绝对值越大线性相关越强。

```python
import seaborn as sns, matplotlib.pyplot as plt
num_cols = ['特征1','特征2','特征3','目标']
corr = df[num_cols].corr()
fig, ax = plt.subplots(figsize=(10, 8))
sns.heatmap(corr, annot=True, fmt='.2f', cmap='RdBu_r', center=0, square=True, ax=ax)
plt.tight_layout(); plt.savefig('fig_corr.png', bbox_inches='tight'); plt.show()
```

**解读**：关注各特征与目标的相关性（哪些是强信号），以及特征间是否有极端冗余（|r|>0.9 的特征对，建模时可保留其一）。

## 5. 时间/空间/类别规律

视数据类型选择性分析：
- **时间规律**：按小时/星期/月分组看目标，发现周期性、高峰时段。
- **空间规律**：按地区/位置类型分组，发现地理分异。
- **类别规律**：不同类别下目标水平的差异。

散点图看连续变量关系时，数据量大要抽样避免过密：
```python
sample = df.sample(min(2000, len(df)), random_state=42)
ax.scatter(sample['特征'], sample['目标'], alpha=0.3, s=15)
```

## 6. ⭐ 特征与目标的关系（EDA 中最有价值的部分）

**对分类任务**：对比"正类 vs 负类"在各特征上的差异。差异越大的特征，预测价值越高——这是后续"关键影响因素"结论的量化依据。
```python
comparison = df.groupby('目标')[num_feats].mean().T
comparison.columns = ['负类','正类']
comparison['差异'] = (comparison['正类'] - comparison['负类'])
print(comparison.round(2))
# 配分组直方图: 正类/负类在关键特征上的分布叠加对比
```

**对分类任务的另一利器——分组"事件率"**：按类别算目标发生率，直接定位高风险条件。
```python
df.groupby('类别列')['目标'].mean().sort_values(ascending=False)  # 各类别的正类率
```

**运行案例（城市噪音）**：分析发现交通指数与噪音强正相关（r≈0.85）；主干路超标率(32%)是支路(5%)的六倍；超标记录的交通指数显著高于未超标记录——这些既验证数据可信，也预告了模型会重点依赖的特征。

## 7. ⭐ 中文字体配置（避免图表乱码 □□□）

如果用户语言是中文，必须先配置中文字体，否则图表中文全是方框。**最稳健的方法是按文件路径加载字体，而非靠字体名匹配**（很多中文字体打包成 .ttc 合集，靠名字匹配可能误命中日文子字体）。

**关键坑**：`sns.set_style()` 会重置字体！所以字体设置必须放在 seaborn 样式设置**之后**。

```python
import matplotlib as mpl, matplotlib.pyplot as plt
import matplotlib.font_manager as fm
import seaborn as sns, os

sns.set_style('whitegrid')   # 先设 seaborn 样式（它会重置字体）

def setup_chinese_font():
    font_files = [
        '/usr/share/fonts/truetype/wqy/wqy-zenhei.ttc',          # Linux 文泉驿
        '/usr/share/fonts/opentype/noto/NotoSansCJK-Regular.ttc', # Linux Noto
        'C:/Windows/Fonts/msyh.ttc',                              # Windows 微软雅黑
        '/System/Library/Fonts/PingFang.ttc',                     # Mac 苹方
    ]
    for fp in font_files:
        if os.path.exists(fp):
            fm.fontManager.addfont(fp)
            name = fm.FontProperties(fname=fp).get_name()
            mpl.rcParams['font.sans-serif'] = [name]
            return name
    return None

setup_chinese_font()
mpl.rcParams['font.family'] = 'sans-serif'   # 关键：让 matplotlib 用上面设的字体
mpl.rcParams['axes.unicode_minus'] = False   # 正常显示负号
```

若本地无中文字体：Windows/Mac 一般自带（代码会自动找到）；Linux 执行 `sudo apt install fonts-wqy-zenhei` 后清缓存 `rm -rf ~/.cache/matplotlib` 再重启。
