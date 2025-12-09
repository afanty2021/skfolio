[根目录](../../../CLAUDE.md) > [src](../../) > [skfolio](../) > **portfolio**

# portfolio - 投资组合表示模块

## 模块职责

portfolio 模块定义了投资组合的数据结构和相关操作。Portfolio 和 MultiPeriodPortfolio 对象是优化算法的返回结果，提供了丰富的分析方法和可视化功能。

## 模块结构

```
portfolio/
├── __init__.py              # 模块入口
├── _base.py                # BasePortfolio 基类
├── _portfolio.py           # Portfolio 单期投资组合
├── _multi_period_portfolio.py  # MultiPeriodPortfolio 多期投资组合
└── _failed_portfolio.py    # FailedPortfolio 失败的投资组合
```

## 入口与启动

模块主入口导出了所有投资组合类：

```python
from skfolio.portfolio import (
    BasePortfolio,           # 基类
    Portfolio,              # 单期投资组合
    MultiPeriodPortfolio,    # 多期投资组合
    FailedPortfolio,        # 失败的投资组合
)
```

## 对外接口

### Portfolio 类

单期投资组合的核心类，包含权重、收益、风险等属性：

```python
class Portfolio(BasePortfolio):
    """单期投资组合类"""

    def __init__(self, weights=None, assets=None, **kwargs):
        """初始化投资组合"""

    # 属性
    weights: np.ndarray      # 资产权重
    assets: list[str]        # 资产名称
    returns: pd.DataFrame    # 收益数据
    mean: float             # 期望收益
    risk: float             # 风险
    sharpe_ratio: float     # 夏普比率

    # 方法
    def plot_composition(self):     # 绘制组合构成
    def plot_cumulative_returns(self):  # 绘制累计收益
    def get_perfect_weights(self):  # 获取理想权重
```

### MultiPeriodPortfolio 类

多期投资组合，包含多个 Portfolio 对象：

```python
class MultiPeriodPortfolio(BasePortfolio):
    """多期投资组合类"""

    def __init__(self, portfolios=None, **kwargs):
        """初始化多期投资组合"""

    # 属性
    portfolios: list[Portfolio]  # 投资组合列表
    n_portfolios: int           # 投资组合数量

    # 方法
    def rollforward(self, X):    # 滚动预测
    def rebalance(self):         # 再平衡
    def plot_frontier(self):     # 绘制有效前沿
```

### BasePortfolio 基类

定义了所有投资组合的通用接口和属性：

```python
class BasePortfolio:
    """投资组合基类"""

    # 通用属性
    name: str
    tag: str
    annualized: bool

    # 计算方法
    def compose(self, X):        # 计算组合收益
    def evaluate(self, measures):  # 计算风险和绩效指标
```

## 关键依赖与配置

### 核心依赖
- **pandas**: 数据处理和分析
- **numpy**: 数值计算
- **plotly**: 交互式可视化
- **scipy**: 统计计算

### 可视化配置
- 支持静态图表和交互式图表
- 自定义颜色主题和样式
- 支持导出为 HTML/PNG

## 数据模型

### 权重数据结构
```python
weights: np.ndarray  # shape (n_assets,)
# 示例: [0.3, 0.2, 0.25, 0.25] 表示 4 个资产的权重
```

### 收益数据结构
```python
returns: pd.DataFrame  # shape (n_periods, n_assets)
# 索引: 时间戳
# 列: 资产名称
```

### 风险度量
- **标准差**: 基础风险度量
- **VaR**: 风险价值
- **CVaR**: 条件风险价值
- **最大回撤**: 最大损失
- **夏普比率**: 风险调整收益

### 绩效指标
- **累计收益**: 投资期间总收益
- **年化收益**: 年化平均收益
- **波动率**: 收益标准差
- **信息比率**: 超额收益/跟踪误差
- **Sortino 比率**: 下行风险调整收益

## 测试与质量

### 测试文件
- `tests/test_portfolio/test_portfolio.py` - Portfolio 类测试
- `tests/test_portfolio/test_multi_period_portfolio.py` - MultiPeriodPortfolio 测试
- `tests/test_portfolio/test_failed_portfolio.py` - FailedPortfolio 测试

### 测试覆盖
- ✅ 权重计算和验证
- ✅ 收益计算
- ✅ 风险和绩效指标
- ✅ 可视化功能
- ✅ 边界条件处理

### 性能优化
- 向量化计算提高性能
- 缓存机制避免重复计算
- 内存优化的数据结构

## 常见问题 (FAQ)

### Q: 如何创建自定义投资组合？
A:
```python
from skfolio.portfolio import Portfolio
import numpy as np

# 创建等权重组合
weights = np.array([0.25, 0.25, 0.25, 0.25])
assets = ["AAPL", "MSFT", "GOOGL", "AMZN"]
portfolio = Portfolio(weights=weights, assets=assets)
```

### Q: 如何计算自定义指标？
A:
```python
# 使用 evaluate 方法
custom_measures = {
    "custom_ratio": lambda p: p.mean / (p.risk + 0.01)
}
results = portfolio.evaluate(custom_measures)
```

### Q: 如何处理缺失数据？
A:
```python
# Portfolio 会自动处理缺失值
# 可以设置填充策略
portfolio = Portfolio(
    weights=weights,
    assets=assets,
    fill_missing="forward"  # 前向填充
)
```

### Q: 如何导出投资组合？
A:
```python
# 导出为 DataFrame
df = portfolio.to_dataframe()

# 导出权重
weights_df = portfolio.weights.to_dataframe()

# 保存为文件
portfolio.save("portfolio.json")
```

## 相关文件清单

### 核心实现
- `_base.py` - 基类定义，包含通用方法
- `_portfolio.py` - Portfolio 类实现
- `_multi_period_portfolio.py` - MultiPeriodPortfolio 实现
- `_failed_portfolio.py` - 失败处理

### 集成文件
- `measures/_measures.py` - 风险和绩效指标计算
- `utils/validation.py` - 输入验证
- `utils/figure.py` - 可视化工具

### 示例文件
- `examples/mean_risk/*.py` - 展示 Portfolio 使用
- `examples/ensemble/plot_1_stacking.py` - 多期组合示例

## 变更记录 (Changelog)

### 2025-12-09 05:21:57 UTC - 模块初始化
- 📚 **创建模块文档**：完整记录了 Portfolio 模块的功能
- 🏗️ **类结构分析**：梳理了 BasePortfolio、Portfolio、MultiPeriodPortfolio 的继承关系
- 📊 **数据模型文档**：详细说明了权重、收益、风险等数据结构
- 🎨 **可视化功能**：记录了绘图和展示功能

### 最新功能
- 支持更多自定义指标
- 改进的可视化选项
- 更好的缺失数据处理
- 性能优化

---

## 使用建议

1. **数据验证**：创建 Portfolio 前验证权重和为 1
2. **风险分析**：使用多种风险度量全面评估
3. **可视化**：利用丰富的图表功能理解组合特性
4. **性能监控**：关注计算时间和内存使用
5. **结果验证**：检查计算结果的合理性

## 扩展开发

如需扩展 Portfolio 功能：

1. **自定义指标**：继承 BasePortfolio 添加新指标
2. **自定义可视化**：使用 plotly 创建新图表
3. **性能优化**：使用 numba 加速计算
4. **数据源集成**：添加新的数据源支持