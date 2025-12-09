[根目录](../../../CLAUDE.md) > [src](../../) > [skfolio](../) > **optimization**

# optimization - 投资组合优化模块

## 模块职责

optimization 模块是 skfolio 的核心组件，提供了各种投资组合优化算法。所有优化器都遵循 scikit-learn 的 Estimator API，实现了 `fit` 和 `predict` 方法。

## 模块结构

```
optimization/
├── __init__.py              # 模块入口，导出所有优化器类
├── _base.py                 # BaseOptimization 基类
├── convex/                  # 凸优化算法
│   ├── __init__.py
│   ├── _base.py            # ConvexOptimization 基类
│   ├── _mean_risk.py       # MeanRisk 优化器
│   ├── _objective_function.py  # 目标函数定义
│   ├── _benchmark_tracker.py   # 基准跟踪优化
│   ├── _distributionally_robust_cvar.py  # 分布鲁棒 CVaR
│   ├── _maximum_diversification.py       # 最大多样化
│   └── _risk_budgeting.py               # 风险预算
├── cluster/                # 分层聚类优化
│   ├── __init__.py
│   ├── _nco.py           # Nested Clusters Optimization
│   └── hierarchical/     # 分层优化算法
│       ├── __init__.py
│       ├── _base.py     # BaseHierarchicalOptimization
│       ├── _herc.py     # Hierarchical Equal Risk Contribution
│       ├── _hrp.py      # Hierarchical Risk Parity
│       └── _schur.py    # Schur Complementary
├── ensemble/             # 集成优化方法
│   ├── __init__.py
│   ├── _base.py        # BaseComposition
│   └── _stacking.py    # StackingOptimization
└── naive/              # 简单/朴素优化策略
    ├── __init__.py
    └── _naive.py       # EqualWeighted, InverseVolatility, Random
```

## 入口与启动

模块主入口文件导出了所有优化器类：

```python
from skfolio.optimization import (
    # 基类
    BaseOptimization,

    # 凸优化
    MeanRisk,
    MaximumDiversification,
    RiskBudgeting,
    BenchmarkTracker,
    DistributionallyRobustCVaR,
    ConvexOptimization,
    ObjectiveFunction,

    # 分层聚类优化
    HierarchicalRiskParity,
    HierarchicalEqualRiskContribution,
    NestedClustersOptimization,
    SchurComplementary,

    # 集成方法
    StackingOptimization,
    BaseComposition,

    # 朴素策略
    EqualWeighted,
    InverseVolatility,
    Random,
)
```

## 对外接口

### BaseOptimization 基类

所有优化器的基类，继承自 scikit-learn 的 BaseEstimator：

```python
class BaseOptimization(BaseEstimator, TransformerMixin):
    """投资组合优化基类"""

    def fit(self, X, y=None, **fit_params):
        """拟合优化模型"""

    def predict(self, X):
        """返回优化后的投资组合"""

    def score(self, X, y=None, **fit_params):
        """计算优化得分"""
```

### MeanRisk 优化器

最常用的凸优化器，支持多种风险度量：

```python
model = MeanRisk(
    risk_measure=RiskMeasure.VARIANCE,  # 风险度量
    objective_function=ObjectiveFunction.MINIMIZE_RISK,  # 优化目标
    min_return=0.0001,  # 最小收益约束
    max_weights=0.4,    # 最大权重约束
    transaction_costs=0.001,  # 交易成本
    l2_coef=0.01,       # L2 正则化
    solver="CLARABEL",  # 求解器
)
```

### HierarchicalRiskParity (HRP)

基于层次聚类风险平分的优化器：

```python
model = HierarchicalRiskParity(
    linkage_method="ward",  # 链接方法
    distance_metric="euclidean",  # 距离度量
)
```

## 关键依赖与配置

### 外部依赖
- **cvxpy**: 凸优化求解器
- **clarabel**: 高性能凸优化求解器
- **scikit-learn**: 基础估计器和工具
- **numpy/pandas**: 数值计算

### 求解器配置
支持的求解器：
- CLARABEL（默认，推荐）
- ECOS
- SCS
- OSQP

### 风险度量支持
- 方差 (VARIANCE)
- 半方差 (SEMI_VARIANCE)
- CVaR (CONDITIONAL_VALUE_AT_RISK)
- 最大回撤 (MAX_DRAWDOWN)
- 平均回撤 (AVERAGE_DRAWDOWN)
- 等 15+ 种风险度量

## 数据模型

### 输入格式
- **X**: 资产收益矩阵，shape `(n_samples, n_assets)`
- **y**: 可选的基准收益

### 输出格式
- **Portfolio**: 单期投资组合对象
- **MultiPeriodPortfolio**: 多期投资组合对象

### 约束类型
- 权重约束：权重上下限
- 预算约束：权重和为1
- 收益约束：最小收益要求
- 风险约束：最大风险限制
- 行业/分组约束
- 基准跟踪约束
- 交易成本约束

## 测试与质量

### 测试文件
- `tests/test_optimization/test_convex/test_mean_risk.py`
- `tests/test_optimization/test_cluster/test_hierarchical/test_hrp.py`
- `tests/test_optimization/test_ensemble/test_stacking.py`
- 等 20+ 测试文件

### 测试覆盖
- ✅ 基本优化功能
- ✅ 约束处理
- ✅ 边界条件
- ✅ 异常处理
- ✅ 性能测试

### 代码质量
- 使用 Ruff 进行代码检查
- 符合 NumPy 文档字符串规范
- 完整的类型提示

## 常见问题 (FAQ)

### Q: 如何选择合适的优化器？
A:
- 对于 **均值-方差优化**：使用 `MeanRisk`
- 对于 **分层风险分配**：使用 `HierarchicalRiskParity`
- 对于 **简单基准**：使用 `EqualWeighted`
- 对于 **多模型组合**：使用 `StackingOptimization`

### Q: 优化不收敛怎么办？
A:
1. 检查约束是否矛盾
2. 尝试不同的求解器
3. 调整约束容差
4. 检查数据质量

### Q: 如何处理大规模优化？
A:
1. 使用资产预选择减少维度
2. 选择高效的求解器（如 CLARABEL）
3. 考虑使用分层优化方法

### Q: 如何集成交易成本？
A:
```python
model = MeanRisk(
    transaction_costs=0.001,  # 0.1% 交易成本
    management_fees=0.0001,   # 0.01% 管理费用
)
```

## 相关文件清单

### 核心实现
- `_base.py` - 基类定义
- `convex/_mean_risk.py` - 均值风险优化
- `convex/_objective_function.py` - 目标函数
- `cluster/hierarchical/_hrp.py` - HRP 算法

### 工具文件
- `utils/tools.py` - 优化工具函数
- `utils/validation.py` - 输入验证

### 示例文件
- `examples/mean_risk/plot_1_maximum_sharpe_ratio.py`
- `examples/clustering/plot_1_hrp_cvar.py`
- `examples/ensemble/plot_1_stacking.py`

## 变更记录 (Changelog)

### 2025-12-09 05:21:57 UTC - 模块初始化
- 📚 **创建模块文档**：详细记录了 optimization 模块的结构和功能
- 🏗️ **架构分析**：梳理了凸优化、分层优化、集成方法和朴素策略四个子模块
- 📋 **API 文档**：记录了主要优化器的使用方法和参数
- ✅ **测试覆盖**：确认了完整的测试覆盖

### 已知限制
- 部分复杂约束（如基数约束）需要 MIP 求解器
- 大规模优化可能需要调优求解器参数
- 某些风险度量的梯度计算可能较慢

---

## 使用建议

1. **从简单开始**：先用 EqualWeighted 或 MeanRisk 建立基准
2. **逐步复杂化**：根据需要添加约束和交易成本
3. **交叉验证**：使用 WalkForward 进行时间序列交叉验证
4. **性能监控**：关注求解时间和收敛性
5. **结果分析**：检查权重分布和风险贡献