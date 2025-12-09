[根目录](../../../CLAUDE.md) > [src](../../) > [skfolio](../) > **measures**

# measures - 风险与绩效度量模块

## 模块职责

measures 模块提供了投资组合分析所需的各种风险度量和绩效指标。这些度量函数用于评估投资组合的风险、收益和风险调整后收益，是优化和决策的重要依据。

## 模块结构

```
measures/
├── __init__.py          # 模块入口，导出所有度量函数
├── _enums.py           # 度量类型枚举定义
└── _measures.py        # 具体度量函数实现
```

## 入口与启动

模块导出了所有度量函数和枚举：

```python
from skfolio.measures import (
    # 枚举类型
    RiskMeasure,         # 风险度量枚举
    PerfMeasure,         # 绩效度量枚举
    RatioMeasure,        # 比率度量枚举
    ExtraRiskMeasure,    # 额外风险度量

    # 风险度量函数
    variance,            # 方差
    standard_deviation,  # 标准差
    semi_variance,       # 半方差
    cvar,               # 条件风险价值
    value_at_risk,      # 风险价值
    max_drawdown,       # 最大回撤

    # 绩效度量函数
    mean,               # 均值
    get_cumulative_returns,  # 累计收益

    # 比率度量函数
    sharpe_ratio,       # 夏普比率
    sortino_ratio,      # 索提诺比率

    # 其他功能函数
    get_drawdowns,      # 获取回撤序列
    effective_number_assets,  # 有效资产数量
)
```

## 对外接口

### 风险度量函数

#### 1. 方差和标准差
```python
def variance(returns: np.ndarray, annualized_factor: float = 1) -> float:
    """计算收益的方差"""

def standard_deviation(returns: np.ndarray, annualized_factor: float = 1) -> float:
    """计算收益的标准差"""
```

#### 2. 下行风险度量
```python
def semi_variance(returns: np.ndarray, min_acceptable_return: float = 0) -> float:
    """计算半方差（只考虑负偏差）"""

def cvar(returns: np.ndarray, level: float = 0.95) -> float:
    """计算条件风险价值（CVaR）"""

def value_at_risk(returns: np.ndarray, level: float = 0.95) -> float:
    """计算风险价值（VaR）"""
```

#### 3. 回撤度量
```python
def max_drawdown(returns: np.ndarray) -> float:
    """计算最大回撤"""

def average_drawdown(returns: np.ndarray) -> float:
    """计算平均回撤"""

def get_drawdowns(returns: np.ndarray) -> np.ndarray:
    """获取完整的回撤序列"""
```

### 绩效度量函数

```python
def mean(returns: np.ndarray, annualized_factor: float = 1) -> float:
    """计算算术平均收益"""

def get_cumulative_returns(returns: np.ndarray) -> np.ndarray:
    """计算累计收益序列"""
```

### 比率度量函数

```python
def sharpe_ratio(returns: np.ndarray, risk_free_rate: float = 0) -> float:
    """计算夏普比率"""

def sortino_ratio(returns: np.ndarray, risk_free_rate: float = 0) -> float:
    """计算索提诺比率（使用下行风险）"""

def information_ratio(returns: np.ndarray, benchmark_returns: np.ndarray) -> float:
    """计算信息比率"""
```

## 关键依赖与配置

### 核心依赖
- **numpy**: 数值计算
- **scipy**: 统计函数
- **pandas**: 时间序列处理

### 年化因子配置
```python
# 常用的年化因子
ANNUALIZATION_FACTORS = {
    "daily": 252,      # 日收益年化
    "weekly": 52,      # 周收益年化
    "monthly": 12,     # 月收益年化
    "quarterly": 4,    # 季度年化
}
```

## 数据模型

### 输入格式
```python
returns: np.ndarray  # shape (n_periods,) 或 (n_periods, n_portfolios)
```

### 输出格式
- 标量值：对于单一投资组合
- 向量：对于多个投资组合

### 度量分类

#### 风险度量 (RiskMeasure)
- **离散度度量**：方差、标准差、平均绝对偏差
- **下行风险**：半方差、一阶下偏矩
- **尾部风险**：VaR、CVaR、EVaR
- **回撤风险**：最大回撤、平均回撤、CDaR
- **其他**：Gini 均差、Ulcer 指数

#### 绩效度量 (PerfMeasure)
- **收益度量**：均值、累计收益
- **高阶矩**：偏度、峰度

## 测试与质量

### 测试文件
- `tests/test_measures/test_measures.py` - 所有度量函数的测试

### 测试覆盖
- ✅ 各种度量函数的准确性
- ✅ 边界条件处理
- ✅ 缺失值处理
- ✅ 不同输入维度
- ✅ 数值稳定性

### 性能优化
- 向量化实现
- 避免重复计算
- 使用 Numba 加速关键函数

## 常见问题 (FAQ)

### Q: 如何选择合适的风险度量？
A:
- **传统投资**：使用方差或标准差
- **对下行风险敏感**：使用半方差或 CVaR
- **关注极端损失**：使用最大回撤或 VaR
- **考虑尾部风险**：使用 EVaR 或 Gini 均差

### Q: 如何年化度量的结果？
A:
```python
from skfolio.measures import standard_deviation, mean

# 日收益数据
daily_returns = ...

# 年化标准差
annual_vol = standard_deviation(daily_returns, annualized_factor=np.sqrt(252))

# 年化收益
annual_return = mean(daily_returns, annualized_factor=252)
```

### Q: 如何处理缺失值？
A:
```python
# 大多数度量函数会自动跳过 NaN
# 但建议先清理数据
returns_clean = returns[~np.isnan(returns)]
result = standard_deviation(returns_clean)
```

### Q: 如何计算自定义风险度量？
A:
```python
def custom_risk(returns, threshold=0.02):
    """自定义风险度量：超过阈值的概率"""
    return np.mean(returns > threshold)

# 使用在优化中
from skfolio.optimization import MeanRisk
model = MeanRisk(
    risk_measure=RiskMeasure.CUSTOM_RISK,
    custom_risk_func=custom_risk
)
```

## 相关文件清单

### 核心实现
- `_measures.py` - 所有度量函数的实现
- `_enums.py` - 度量类型枚举定义

### 集成文件
- `portfolio/_portfolio.py` - Portfolio 类使用度量函数
- `optimization/_base.py` - 优化器使用度量
- `population/_population.py` - Population 类使用度量

### 示例文件
- `examples/mean_risk/*.py` - 展示各种风险度量的使用
- `examples/model_selection/*.py` - 使用度量进行模型选择

## 变更记录 (Changelog)

### 2025-12-09 05:21:57 UTC - 模块初始化
- 📚 **创建模块文档**：完整记录了 measures 模块的功能
- 📊 **度量分类**：将度量函数按风险、绩效、比率分类说明
- 📝 **使用示例**：提供了各种度量函数的使用示例
- ✅ **性能考虑**：记录了性能优化建议

### 最新功能
- 新增 Entropic Risk Measure (ERM)
- 改进的尾部风险度量
- 更高效的回撤计算
- 支持自定义度量

---

## 使用建议

1. **选择合适的度量**：根据投资目标选择合适的风险度量
2. **注意年化**：正确使用年化因子
3. **数据预处理**：处理缺失值和异常值
4. **组合使用**：同时使用多个度量全面评估
5. **基准比较**：使用信息比率等相对度量

## 扩展开发

添加新度量：

```python
def new_measure(returns: np.ndarray, **kwargs) -> float:
    """新的度量函数"""
    # 实现逻辑
    return result

# 添加到枚举
class RiskMeasure(Enum):
    # 现有度量...
    NEW_MEASURE = "new_measure"
```