[根目录](../../../CLAUDE.md) > [src](../../) > [skfolio](../) > **population**

# population - 投资组合种群管理模块

## 模块职责

population 模块提供了管理多个投资组合（Population）的功能。Population 是一个投资组合的集合，可以用于比较不同优化策略的结果、进行投资组合分析、风险度量和绩效评估。该模块支持单期和多期投资组合的集合操作。

## 模块结构

```
population/
├── __init__.py              # 模块入口
└── _population.py          # Population 类实现
```

## 入口与启动

```python
from skfolio.population import Population, BasePopulation
from skfolio.portfolio import Portfolio, MultiPeriodPortfolio
```

## 对外接口

### Population 类

投资组合集合的核心类：

```python
class Population(BasePopulation):
    """投资组合种群类"""

    def __init__(
        self,
        portfolios: list = None,
        name: str = None,
        tag: str = None,
        risk_free_rate: float = 0,
        annualized_factor: float = 1,
    ):
        """初始化种群"""

    # 属性
    portfolios: list          # 投资组合列表
    n_portfolios: int         # 投资组合数量

    # 方法
    def append(self, portfolio):        # 添加投资组合
    def extend(self, portfolios):       # 扩展投资组合列表
    def sort(self, measure, ascending=False):  # 排序
    def filter(self, condition):        # 过滤
    def evaluate(self, measures):       # 批量评估
    def plot_frontier(self):            # 绘制有效前沿
    def plot_distribution(self):        # 绘制分布图
    def get_best(self, measure):        # 获取最优组合
    def get_dominated(self):            # 获取被支配组合
    def get_non_dominated(self):        # 获取非支配组合
```

### 使用示例

```python
from skfolio import Population
from skfolio.optimization import MeanRisk, HierarchicalRiskParity
from skfolio.datasets import load_sp500_dataset

# 加载数据
prices = load_sp500_dataset()
returns = prices.pct_change().dropna()

# 创建多个优化器
models = [
    MeanRisk(risk_measure=RiskMeasure.VARIANCE),
    MeanRisk(risk_measure=RiskMeasure.CVAR),
    HierarchicalRiskParity(),
]

# 拟合并收集结果
portfolios = []
for model in models:
    model.fit(returns)
    portfolios.append(model.predict_)

# 创建种群
population = Population(portfolios)

# 分析种群
print(f"投资组合数量: {population.n_portfolios}")

# 按夏普比率排序
population.sort("sharpe_ratio", ascending=False)

# 获取夏普比率最高的组合
best_portfolio = population.get_best("sharpe_ratio")

# 绘制有效前沿
population.plot_frontier()
```

## 关键功能

### 1. 批量评估
```python
# 计算所有组合的多个指标
results = population.evaluate([
    "mean", "std", "sharpe_ratio", "max_drawdown",
    "variance", "cvar", "sortino_ratio"
])
```

### 2. 筛选和过滤
```python
# 筛选夏普比率大于0.5的组合
high_sharpe = population.filter(
    lambda p: p.sharpe_ratio > 0.5
)

# 筛选风险最小的10个组合
low_risk = population.sort("std")[:10]
```

### 3. 可视化
```python
# 有效前沿
population.plot_frontier(
    measure_x="std",
    measure_y="mean",
    color_map="viridis",
    show_pareto_front=True,
)

# 权重分布
population.plot_distribution(
    measure="weights",
    kind="boxplot",
)

# 风险收益散点图
population.plot_distribution(
    measure_x="std",
    measure_y="mean",
    size="sharpe_ratio",
)
```

### 4. Pareto 最优
```python
# 获取非支配组合（Pareto前沿）
pareto_front = population.get_non_dominated()

# 风险-收益 Pareto 分析
pareto_risk_return = population.get_non_dominated(
    measures=["std", "mean"]
)
```

## 关键依赖与配置

### 核心依赖
- **pandas**: 数据处理
- **numpy**: 数值计算
- **plotly**: 可视化
- **scipy**: 统计计算

### 性能优化
- 向量化计算
- 缓存机制
- 并行评估

## 测试与质量

### 测试文件
- `tests/test_population/test_population.py` - Population 类测试

### 测试覆盖
- ✅ 种群创建和管理
- ✅ 排序和筛选功能
- ✅ Pareto 最优计算
- ✅ 可视化功能
- ✅ 边界条件

## 常见问题 (FAQ)

### Q: 如何比较不同优化策略？
A:
```python
# 创建不同策略的投资组合
strategies = [
    ("Equal Weight", EqualWeighted()),
    ("Min Variance", MeanRisk(risk_measure=RiskMeasure.VARIANCE)),
    ("Min CVaR", MeanRisk(risk_measure=RiskMeasure.CVAR)),
    ("HRP", HierarchicalRiskParity()),
]

# 生成种群并比较
portfolios = []
for name, model in strategies:
    model.fit(returns)
    p = model.predict_
    p.name = name
    portfolios.append(p)

population = Population(portfolios)
population.evaluate(["mean", "std", "sharpe_ratio"])
```

### Q: 如何进行稳健性分析？
A:
```python
# 使用不同的时间窗口进行滚动分析
windows = [(0, 252), (252, 504), (504, 756)]
population_windows = []

for start, end in windows:
    window_returns = returns.iloc[start:end]
    model = MeanRisk()
    model.fit(window_returns)
    population_windows.append(model.predict_)

# 分析权重稳定性
weights_matrix = np.array([
    p.weights for p in population_windows
])
weight_volatility = np.std(weights_matrix, axis=0)
```

### Q: 如何创建投资组合集合？
A:
```python
# 方法1：从优化器创建
from sklearn.model_selection import ParameterGrid

params = {
    "risk_measure": [RiskMeasure.VARIANCE, RiskMeasure.CVAR],
    "min_return": [0.0001, 0.0002],
    "l2_coef": [0.001, 0.01],
}

portfolios = []
for param_set in ParameterGrid(params):
    model = MeanRisk(**param_set)
    model.fit(returns)
    portfolios.append(model.predict_)

population = Population(portfolios)
```

## 相关文件清单

### 核心实现
- `_population.py` - Population 类实现

### 集成文件
- `optimization/ensemble/_stacking.py` - 使用 Population
- `measures/_measures.py` - 批量评估函数

### 示例文件
- 各优化器示例中展示 Population 使用

## 变更记录 (Changelog)

### 2025-12-09 06:15:32 UTC - 模块初始化
- 📚 **创建模块文档**：完整记录了 population 模块的功能
- 🏗️ **功能分析**：详细说明了投资组合管理的各种操作
- 📊 **使用示例**：提供了典型使用场景的代码示例
- ✅ **测试覆盖**：确认了测试覆盖

---

## 使用建议

1. **比较分析**：使用 Population 比较不同策略
2. **可视化**：充分利用图表理解投资组合特征
3. **Pareto 分析**：识别最优投资组合集合
4. **稳健性测试**：通过多个样本评估策略稳定性
5. **批量操作**：利用批量评估提高效率

## 扩展开发

```python
class ExtendedPopulation(Population):
    def get_diversification_ratio(self):
        """计算多样化比率"""
        weights = np.array([p.weights for p in self.portfolios])
        # 实现多样化比率计算
        pass

    def clustering_analysis(self):
        """投资组合聚类分析"""
        # 实现 K-means 或层次聚类
        pass
```