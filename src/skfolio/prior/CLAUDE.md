[根目录](../../../CLAUDE.md) > [src](../../) > [skfolio](../) > **prior**

# prior - 先验信息集成模块

## 模块职责

prior 模块提供了将投资观点、均衡模型和先验信息集成到投资组合优化中的工具。这些方法允许投资者结合量化分析、市场均衡和主观观点，生成更加稳健和合理的期望收益估计，特别适用于 Black-Litterman 模型、熵池化等现代投资组合理论。

## 模块结构

```
prior/
├── __init__.py              # 模块入口
├── _base.py                # BasePrior 基类
├── _empirical.py           # 经验先验
├── _black_litterman.py     # Black-Litterman 模型
├── _entropy_pooling.py     # 熵池化方法
├── _opinion_pooling.py     # 观点池化
├── _factor_model.py        # 因子模型
└── _synthetic_data.py      # 合成数据生成
```

## 入口与启动

模块导出了所有先验模型：

```python
from skfolio.prior import (
    # 基类
    BasePrior,
    ReturnDistribution,
    BaseLoadingMatrix,

    # 先验模型
    EmpiricalPrior,           # 经验先验
    BlackLitterman,          # Black-Litterman 模型
    EntropyPooling,          # 熵池化
    OpinionPooling,          # 观点池化
    FactorModel,             # 因子模型

    # 因子模型工具
    LoadingMatrixRegression,

    # 合成数据
    SyntheticData,
)
```

## 对外接口

### BasePrior 基类

所有先验模型的基类：

```python
class BasePrior(BaseEstimator, TransformerMixin):
    """先验模型基类"""

    def fit(self, X, y=None, **fit_params):
        """拟合先验模型"""

    def set_prior_model(self, prior_model):
        """设置先验模型参数"""

    def get_prior_mu(self):
        """获取先验期望收益"""

    def get_prior_covariance(self):
        """获取先验协方差矩阵"""

    def compute_posterior(self, views):
        """计算后验分布"""
```

### EmpiricalPrior

基于历史数据的经验先验：

```python
empirical = EmpiricalPrior(
    mu_estimator=EmpiricalMu(),         # 期望收益估计器
    covariance_estimator=EmpiricalCovariance(),  # 协方差估计器
    risk_free_rate=0.0,                # 无风险利率
    min_n_samples=None,                # 最小样本数
    window_size=None,                  # 滚动窗口
)

# 拟合先验
empirical.fit(returns)
prior_mu = empirical.prior_mu_
prior_cov = empirical.prior_covariance_
```

### BlackLitterman

Black-Litterman 模型，结合市场均衡和投资者观点：

```python
bl = BlackLitterman(
    prior=EmpiricalPrior(),            # 先验模型
    risk_aversion=1.0,                 # 风险厌恶系数
    tau=0.05,                         # 不确定性标量
    equilibrium="implied",             # 均衡类型
)

# 添加观点
views = np.array([
    [1, 0, -1, 0],    # 资产1预期比资产3高2%
    [0, 1, 0, -1],    # 资产2预期比资产4高1%
])
views_confidences = np.array([0.5, 0.3])  # 观点置信度

bl.set_views(views, views_confidences)
bl.fit(returns)
```

### EntropyPooling

熵池化方法，基于信息论的一致性观点集成：

```python
ep = EntropyPooling(
    prior=EmpiricalPrior(),            # 先验分布
    confidence=0.95,                   # 置信水平
    bounds=None,                       # 边界约束
    minimize="relative_entropy",       # 最小化目标
)

# 设置线性观点
linear_views = {
    "A": np.array([[1, -1, 0, 0]]),   # 资产1 > 资产2
    "b": np.array([0.02]),            # 差值为2%
    "inequality": True,               # 不等式约束
}

ep.set_linear_views(linear_views)
ep.fit(returns)
```

### FactorModel

多因子模型，将资产收益分解为因子收益：

```python
factor_model = FactorModel(
    factor_returns=None,              # 因子收益数据
    loading_matrix=None,              # 因子载荷矩阵
    factor_prior=EmpiricalPrior(),    # 因子先验
    idiosyncratic_prior=None,         # 特异性风险先验
)

# 使用回归估计因子载荷
loading_matrix = LoadingMatrixRegression(
    method="ols",                     # 回归方法
    intercept=True,                   # 包含截距
    regularizer=None,                 # 正则化
)

factor_model.loading_matrix = loading_matrix
factor_model.fit(returns, factor_returns)
```

### OpinionPooling

观点池化方法，线性组合多个观点：

```python
op = OpinionPooling(
    priors=[EmpiricalPrior(), FactorModel()],  # 多个先验
    weights=[0.6, 0.4],                       # 组合权重
    pooling_method="linear",                   # 池化方法
)

op.fit(returns)
```

## 关键依赖与配置

### 核心依赖
- **numpy**: 数值计算
- **pandas**: 数据处理
- **scipy**: 优化和统计
- **cvxpy**: 凸优化求解

### 优化配置
```python
# Black-Litterman 优化选项
bl.optimizer_kwargs = {
    "solver": "CLARABEL",
    "verbose": True,
    "max_iters": 1000,
}
```

### 并行处理
- 支持多观点并行计算
- 批量处理多个场景

## 数据模型

### 观点矩阵格式
```python
P: np.ndarray  # shape (n_views, n_assets) - 观点载荷矩阵
q: np.ndarray  # shape (n_views,) - 观点收益向量
Omega: np.ndarray  # shape (n_views, n_views) - 观点不确定性矩阵
```

### 因子模型格式
```python
B: np.ndarray  # shape (n_assets, n_factors) - 因子载荷
F: np.ndarray  # shape (n_periods, n_factors) - 因子收益
D: np.ndarray  # shape (n_assets,) - 特异性风险
```

### 返回分布
```python
ReturnDistribution = namedtuple(
    "ReturnDistribution",
    ["mu", "covariance", "returns"]  # 期望收益、协方差、收益数据
)
```

## 先验模型对比

| 模型 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **EmpiricalPrior** | 简单直观 | 忽视先验信息 | 基准模型 |
| **BlackLitterman** | 理论完备 | 参数敏感 | 观点明确 |
| **EntropyPooling** | 灵活约束 | 计算复杂 | 复杂观点 |
| **FactorModel** | 经济含义 | 因子选择 | 多因子分析 |
| **OpinionPooling** | 组合灵活 | 权重主观 | 多源信息 |

## 测试与质量

### 测试文件
- `tests/test_prior/test_empirical.py` - 经验先验测试
- `tests/test_prior/test_black_litterman.py` - Black-Litterman 测试
- `tests/test_prior/test_entropy_pooling.py` - 熵池化测试
- `tests/test_prior/test_factor_model.py` - 因子模型测试
- `tests/test_prior/test_opinion_pooling.py` - 观点池化测试

### 测试覆盖
- ✅ 各种先验模型的正确性
- ✅ 观点集成逻辑
- ✅ 优化求解稳定性
- ✅ 边界条件
- ✅ 数值精度

### 性能优化
- 向量化实现
- 缓存机制
- 使用高效求解器

## 常见问题 (FAQ)

### Q: 如何选择风险厌恶系数？
A:
```python
# 基于 Black-Litterman 推荐
risk_aversion = 1 / (annual_return / annual_volatility**2)

# 或使用历史估计
risk_aversion = np.mean(returns) / np.var(returns) * 252
```

### Q: 如何设置观点置信度？
A:
```python
# 基于历史准确性
historical_errors = views - actual_returns
confidence = 1 / np.var(historical_errors)

# 或基于主观判断
confidence = np.array([0.8, 0.6, 0.4])  # 80%, 60%, 40% 置信度
```

### Q: 如何处理因子模型中的缺失数据？
A:
```python
# 使用 EM 算法
factor_model = FactorModel(
    loading_matrix=LoadingMatrixRegression(
        method="em",  # 期望最大化
        max_iter=100,
        tolerance=1e-6,
    )
)
```

### Q: 如何验证先验模型？
A:
```python
# 使用交叉验证
from skfolio.model_selection import WalkForward

wf = WalkForward(train_size=504, test_size=252)
results = wf.evaluate(
    X=returns,
    models=[BlackLitterman(), EmpiricalPrior()]
)
```

## 相关文件清单

### 核心实现
- `_black_litterman.py` - Black-Litterman 实现
- `_entropy_pooling.py` - 熵池化实现
- `_factor_model.py` - 因子模型实现
- `_base.py` - 基类和工具函数

### 集成文件
- `optimization/_base.py` - 优化器使用先验
- `prior/_synthetic_data.py` - 合成数据生成

### 示例文件
- `examples/mean_risk/plot_12_black_and_litterman.py` - Black-Litterman 示例
- `examples/entropy_pooling/plot_1_entropy_pooling.py` - 熵池化示例
- `examples/mean_risk/plot_13_factor_model.py` - 因子模型示例

## 变更记录 (Changelog)

### 2025-12-09 06:15:32 UTC - 模块初始化
- 📚 **创建模块文档**：完整记录了 prior 模块的功能
- 🏗️ **模型分析**：详细说明了 Black-Litterman、熵池化、因子模型等
- 📊 **参数配置**：记录了风险厌恶、观点置信度等关键参数
- ✅ **测试覆盖**：确认了完整的测试覆盖

### 最新功能
- 改进的熵池化算法
- 支持更多因子模型
- 更灵活的观点格式
- 并行计算优化

---

## 使用建议

1. **模型选择**：根据信息类型选择合适的先验模型
2. **参数校准**：使用历史数据校准关键参数
3. **观点质量**：确保观点的合理性和一致性
4. **敏感性分析**：测试参数变化的影响
5. **稳健性**：使用多种先验模型比较结果

## 扩展开发

添加自定义先验：

```python
class CustomPrior(BasePrior):
    def __init__(self, param1=1.0):
        super().__init__()
        self.param1 = param1

    def fit(self, X, y=None):
        # 实现自定义先验估计
        self.prior_mu_ = self._estimate_mu(X)
        self.prior_covariance_ = self._estimate_cov(X)
        return self
```

## 最佳实践

1. **观点来源**：确保观点有充分的数据或理论支撑
2. **置信度**：客观评估观点的可靠性
3. **参数调优**：使用交叉验证选择最优参数
4. **结果验证**：检查后验分布的合理性
5. **风险控制**：考虑模型风险和参数不确定性