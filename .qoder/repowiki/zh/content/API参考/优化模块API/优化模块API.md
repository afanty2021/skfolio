# 优化模块API

<cite>
**本文档引用的文件**
- [__init__.py](file://src/skfolio/optimization/__init__.py)
- [_base.py](file://src/skfolio/optimization/_base.py)
- [_mean_risk.py](file://src/skfolio/optimization/convex/_mean_risk.py)
- [_risk_budgeting.py](file://src/skfolio/optimization/convex/_risk_budgeting.py)
- [_hrp.py](file://src/skfolio/optimization/cluster/hierarchical/_hrp.py)
- [_herc.py](file://src/skfolio/optimization/cluster/hierarchical/_herc.py)
- [_nco.py](file://src/skfolio/optimization/cluster/_nco.py)
- [_naive.py](file://src/skfolio/optimization/naive/_naive.py)
</cite>

## 目录
1. [简介](#简介)
2. [BaseOptimization 基类](#baseoptimization-基类)
3. [MeanRisk 优化器](#meanrisk-优化器)
4. [RiskBudgeting 优化器](#riskbudgeting-优化器)
5. [HierarchicalRiskParity 优化器](#hierarchicalriskparity-优化器)
6. [HierarchicalEqualRiskContribution 优化器](#hierarchicalequalriskcontribution-优化器)
7. [其他优化器](#其他优化器)
8. [回退机制](#回退机制)
9. [在Pipeline中使用](#在pipeline中使用)
10. [与scikit-learn交叉验证的兼容性](#与scikit-learn交叉验证的兼容性)

## 简介
skfolio的优化模块提供了一系列用于投资组合优化的估计器。所有优化器都继承自`BaseOptimization`基类，并遵循scikit-learn的API约定。本API文档详细介绍了`BaseOptimization`基类及其所有公共子类，包括`MeanRisk`、`RiskBudgeting`、`HierarchicalRiskParity`等。文档涵盖了每个优化器类的完整参数说明、属性和方法，并特别说明了回退机制的使用。

**Section sources**
- [__init__.py](file://src/skfolio/optimization/__init__.py)

## BaseOptimization 基类
`BaseOptimization`是skfolio中所有投资组合优化算法的基类。它定义了所有优化器共有的核心参数、属性和方法。

### 参数
- **portfolio_params**: 一个字典，包含在`predict`方法中传递给结果`Portfolio`的参数。如果未提供且估计器上可用，则默认会传播以下属性：`name`、`transaction_costs`、`management_fees`、`previous_weights`和`risk_free_rate`。
- **fallback**: 当主优化在`fit`期间引发异常时，要尝试的备用估计器或估计器列表（按顺序）。也可以使用字符串`"previous_weights"`（单独或在列表中）来回退到估计器的`previous_weights`。
- **previous_weights**: 资产的先前权重。某些估计器使用此参数来计算成本或换手率。此外，当`fallback="previous_weights"`时，如果提供了此参数，失败将回退到这些权重。
- **raise_on_failure**: 一个布尔值，控制拟合失败时的错误处理。如果为`True`，则在`fit`期间的任何失败都会立即引发，不会设置`weights_`，后续对`predict`的调用将引发`NotFittedError`。如果为`False`，则不会引发错误；而是发出警告，`weights_`设置为`None`，后续对`predict`的调用将返回一个`FailedPortfolio`。

### 属性
- **weights_**: 形状为(n_assets,)或(n_optimizations, n_assets)的ndarray，表示资产的权重。
- **n_features_in_**: 在`fit`期间看到的资产数量。
- **feature_names_in_**: 在`fit`期间看到的资产名称的ndarray，仅当`X`具有所有为字符串的资产名称时定义。
- **fallback_**: 产生最终结果的备用估计器实例或字符串`"previous_weights"`。如果没有使用备用，则为`None`。
- **fallback_chain_**: 描述优化备用尝试的序列。每个元素是一个元组`(estimator_repr, outcome)`，其中`estimator_repr`是主估计器或备用的字符串表示，`outcome`是`"success"`（如果该步骤产生了有效解决方案），否则是字符串化的错误消息。对于没有备用的成功拟合，此值为`None`。
- **error_**: 当`fit`失败时捕获的错误消息。对于多投资组合输出（`weights_`为2D），这是一个与投资组合对齐的列表。

### 方法
- **fit(X, y=None, **fit_params)**: 拟合优化器。
- **predict(X)**: 基于拟合的`weights`在`X`上预测`Portfolio`或`Portfolio`的`Population`。
- **score(X, y=None)**: 使用夏普比率计算预测分数。
- **fit_predict(X)**: 在`X`上执行`fit`并返回基于拟合`weights`的预测`Portfolio`或`Portfolio`的`Population`。

**Section sources**
- [_base.py](file://src/skfolio/optimization/_base.py)

## MeanRisk 优化器
`MeanRisk`优化器实现了均值-风险优化框架，可以最小化风险、最大化预期收益、最大化效用或最大化比率。

### 参数
- **objective_function**: 优化目标函数，可以是`MINIMIZE_RISK`、`MAXIMIZE_RETURN`、`MAXIMIZE_UTILITY`或`MAXIMIZE_RATIO`。
- **risk_measure**: 优化的风险度量，支持多种风险度量，如方差、CVaR、最大回撤等。
- **risk_aversion**: 效用函数的风险厌恶因子，仅在`objective_function=MAXIMIZE_UTILITY`时使用。
- **prior_estimator**: 用于估计资产预期收益、协方差矩阵和收益的先验估计器。
- **efficient_frontier_size**: 如果提供，表示要计算的有效前沿上帕累托最优投资组合的数量。
- **min_weights**: 资产权重的最小值（权重下限）。
- **max_weights**: 资产权重的最大值（权上限）。
- **budget**: 投资预算，即多头和空头头寸的总和（所有权重的总和）。
- **transaction_costs**: 资产的交易成本，用于在优化问题中添加线性交易成本。
- **management_fees**: 资产的管理费，用于在优化问题中添加线性管理费。
- **previous_weights**: 资产的先前权重，用于计算投资组合成本和换手率。
- **target_weights**: 资产的目标权重，当提供时，风险度量在这些目标权重的偏差上计算。
- **l1_coef**: L1正则化系数，用于惩罚目标函数。
- **l2_coef**: L2正则化系数，用于惩罚目标函数。
- **mu_uncertainty_set_estimator**: 用于对资产预期收益建模的椭球不确定性集估计器。
- **covariance_uncertainty_set_estimator**: 用于对资产协方差矩阵建模的椭球不确定性集估计器。
- **linear_constraints**: 线性约束。
- **groups**: `linear_constraints`中引用的资产组。
- **left_inequality**: 线性约束`A · w ≤ b`的左侧不等式矩阵`A`。
- **right_inequality**: 线性约束`A · w ≤ b`的右侧不等式向量`b`。
- **risk_free_rate**: 无风险利率。
- **max_tracking_error**: 跟踪误差的上限约束。
- **max_turnover**: 换手率的上限约束。
- **solver**: 要使用的求解器，默认为"CLARABEL"。
- **solver_params**: 求解器参数。

### 属性
- **problem_values_**: 从CVXPY问题中检索的表达式值。
- **prior_estimator_**: 拟合的`prior_estimator`。
- **mu_uncertainty_set_estimator_**: 拟合的`mu_uncertainty_set_estimator`（如果提供）。
- **covariance_uncertainty_set_estimator_**: 拟合的`covariance_uncertainty_set_estimator`（如果提供）。
- **problem_**: 用于优化的CVXPY问题，仅当`save_problem`设置为`True`时。

**Section sources**
- [_mean_risk.py](file://src/skfolio/optimization/convex/_mean_risk.py)

## RiskBudgeting 优化器
`RiskBudgeting`优化器解决了风险预算优化问题，其中每个资产被分配一个风险预算。

### 参数
- **risk_measure**: 优化的风险度量。
- **risk_budget**: 分配给每个资产的风险预算。如果为`None`，则使用单位向量，将风险预算减少为风险平价（每个资产对总风险的贡献相等）。
- **prior_estimator**: 用于估计资产预期收益、协方差矩阵和收益的先验估计器。
- **min_weights**: 资产权重的最小值（权重下限）。
- **max_weights**: 资产权重的最大值（权上限）。
- **transaction_costs**: 资产的交易成本。
- **management_fees**: 资产的管理费。
- **previous_weights**: 资产的先前权重。
- **linear_constraints**: 线性约束。
- **groups**: `linear_constraints`中引用的资产组。
- **left_inequality**: 线性约束`A · w ≤ b`的左侧不等式矩阵`A`。
- **right_inequality**: 线性约束`A · w ≤ b`的右侧不等式向量`b`。
- **risk_free_rate**: 无风险利率。
- **min_return**: 预期收益的下限约束。
- **solver**: 要使用的求解器，默认为"CLARABEL"。
- **solver_params**: 求解器参数。

### 属性
- **problem_values_**: 从CVXPY问题中检索的表达式值。
- **prior_estimator_**: 拟合的`prior_estimator`。
- **problem_**: 用于优化的CVXPY问题，仅当`save_problem`设置为`True`时。

**Section sources**
- [_risk_budgeting.py](file://src/skfolio/optimization/convex/_risk_budgeting.py)

## HierarchicalRiskParity 优化器
`HierarchicalRiskParity`优化器使用距离矩阵计算层次聚类，然后通过递归二分法分配权重。

### 参数
- **risk_measure**: 优化的风险度量。
- **prior_estimator**: 用于估计资产预期收益、协方差矩阵和收益的先验估计器。
- **distance_estimator**: 用于估计代码依赖性和距离矩阵的距离估计器。
- **hierarchical_clustering_estimator**: 用于计算链接矩阵和资产层次聚类的层次聚类估计器。
- **min_weights**: 资产权重的最小值（权重下限）。
- **max_weights**: 资产权重的最大值（权上限）。
- **transaction_costs**: 资产的交易成本。
- **management_fees**: 资产的管理费。
- **previous_weights**: 资产的先前权重。

### 属性
- **distance_estimator_**: 拟合的`distance_estimator`。
- **hierarchical_clustering_estimator_**: 拟合的`hierarchical_clustering_estimator`。

**Section sources**
- [_hrp.py](file://src/skfolio/optimization/cluster/hierarchical/_hrp.py)

## HierarchicalEqualRiskContribution 优化器
`HierarchicalEqualRiskContribution`优化器使用距离矩阵计算层次聚类，然后通过自上而下的递归划分分配权重。

### 参数
- **risk_measure**: 优化的风险度量。
- **prior_estimator**: 用于估计资产预期收益、协方差矩阵和收益的先验估计器。
- **distance_estimator**: 用于估计代码依赖性和距离矩阵的距离估计器。
- **hierarchical_clustering_estimator**: 用于计算链接矩阵和资产层次聚类的层次聚类估计器。
- **min_weights**: 资产权重的最小值（权重下限）。
- **max_weights**: 资产权重的最大值（权上限）。
- **transaction_costs**: 资产的交易成本。
- **management_fees**: 资产的管理费。
- **previous_weights**: 资产的先前权重。
- **solver**: 用于权重约束优化的求解器。
- **solver_params**: 求解器参数。

### 属性
- **distance_estimator_**: 拟合的`distance_estimator`。
- **hierarchical_clustering_estimator_**: 拟合的`hierarchical_clustering_estimator`。

**Section sources**
- [_herc.py](file://src/skfolio/optimization/cluster/hierarchical/_herc.py)

## 其他优化器
除了上述优化器外，skfolio还提供了其他几种优化器：

- **NestedClustersOptimization**: 嵌套聚类优化器，使用聚类算法将资产分组，然后在每个聚类内和聚类间分别优化权重。
- **EqualWeighted**: 等权重优化器，每个资产的权重相等。
- **InverseVolatility**: 反波动率优化器，每个资产的权重与其波动率的倒数成正比。
- **Random**: 随机权重优化器，资产权重从狄利克雷分布中抽取。

**Section sources**
- [_nco.py](file://src/skfolio/optimization/cluster/_nco.py)
- [_naive.py](file://src/skfolio/optimization/naive/_naive.py)

## 回退机制
回退机制允许在主优化失败时尝试备用优化器或回退到先前的权重。

### 配置备用优化器链
可以通过`fallback`参数配置备用优化器链。例如：
```python
fallback_chain = [MeanRisk(), EqualWeighted()]
model = MeanRisk(fallback=fallback_chain)
```
当主优化失败时，将按顺序尝试链中的每个备用优化器。

### 'previous_weights' 字符串参数
可以使用字符串`"previous_weights"`作为备用，以回退到估计器的`previous_weights`。例如：
```python
model = MeanRisk(fallback="previous_weights", previous_weights=[0.5, 0.5])
```
如果优化失败，将回退到提供的`previous_weights`。

**Section sources**
- [_base.py](file://src/skfolio/optimization/_base.py)

## 在Pipeline中使用
优化器可以在scikit-learn的Pipeline中使用，以组合多个预处理步骤和优化步骤。

### 示例
```python
from sklearn.pipeline import Pipeline
from skfolio.preprocessing import prices_to_returns
from skfolio.optimization import MeanRisk

pipeline = Pipeline([
    ('returns', prices_to_returns),
    ('optimization', MeanRisk())
])
pipeline.fit(prices)
```

**Section sources**
- [plot_1_maximum_sharpe_ratio.py](file://examples/mean_risk/plot_1_maximum_sharpe_ratio.py)

## 与scikit-learn交叉验证的兼容性
优化器与scikit-learn的交叉验证工具兼容，可以使用`cross_val_score`等函数评估模型性能。

### 示例
```python
from sklearn.model_selection import cross_val_score
from skfolio.optimization import MeanRisk

model = MeanRisk()
scores = cross_val_score(model, X_train, cv=5)
```

**Section sources**
- [plot_1_maximum_sharpe_ratio.py](file://examples/mean_risk/plot_1_maximum_sharpe_ratio.py)