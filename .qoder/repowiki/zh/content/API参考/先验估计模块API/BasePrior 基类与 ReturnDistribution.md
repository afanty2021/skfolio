# BasePrior 基类与 ReturnDistribution

<cite>
**本文档引用的文件**   
- [src/skfolio/prior/_base.py](file://src/skfolio/prior/_base.py)
- [src/skfolio/prior/_empirical.py](file://src/skfolio/prior/_empirical.py)
- [src/skfolio/prior/_black_litterman.py](file://src/skfolio/prior/_black_litterman.py)
- [src/skfolio/prior/_entropy_pooling.py](file://src/skfolio/prior/_entropy_pooling.py)
- [src/skfolio/optimization/convex/_base.py](file://src/skfolio/optimization/convex/_base.py)
- [src/skfolio/optimization/convex/_mean_risk.py](file://src/skfolio/optimization/convex/_mean_risk.py)
</cite>

## 目录
1. [简介](#简介)
2. [BasePrior 基类设计](#baseprior-基类设计)
3. [ReturnDistribution 数据类](#returndistribution-数据类)
4. [继承与实现示例](#继承与实现示例)
5. [在优化器中的传递机制](#在优化器中的传递机制)

## 简介
本文档详细阐述了 `skfolio` 库中先验模块的核心组件：`BasePrior` 抽象基类和 `ReturnDistribution` 数据类。`BasePrior` 作为所有先验估计器的统一接口，遵循 scikit-learn 的规范，为资产收益分布的估计提供了标准化的框架。`ReturnDistribution` 则是一个高效的数据容器，用于封装和传递资产收益的核心统计信息。本文将深入解析其设计原理、数据结构、性能优化机制以及在投资组合优化流程中的实际应用。

## BasePrior 基类设计

`BasePrior` 是 `skfolio` 中所有先验估计器的抽象基类，它继承自 `sklearn.base.BaseEstimator` 并实现了 `abc.ABC` (抽象基类)。其设计核心在于提供一个统一、兼容且可扩展的接口，确保所有具体的先验估计器（如经验估计、Black-Litterman模型等）都遵循相同的模式。

该基类定义了所有先验估计器必须实现的 `fit` 方法，该方法接收资产收益数据 `X` 并返回一个 `ReturnDistribution` 对象。这种设计使得上层的优化器可以无缝地与任何实现了 `BasePrior` 接口的估计器进行交互，极大地增强了框架的灵活性和可组合性。

**Section sources**
- [src/skfolio/prior/_base.py](file://src/skfolio/prior/_base.py#L51-L69)

## ReturnDistribution 数据类

`ReturnDistribution` 是一个使用 `@dataclass(frozen=True, eq=False)` 装饰器定义的不可变数据类，专门用于封装和传递由先验估计器计算出的资产收益分布信息。其核心字段包括：

- **mu**: 一个形状为 `(n_assets,)` 的 `ndarray`，表示对资产预期收益的估计。
- **covariance**: 一个形状为 `(n_assets, n_assets)` 的 `ndarray`，表示对资产协方差矩阵的估计。
- **returns**: 一个形状为 `(n_observations, n_assets)` 的 `ndarray`，表示对资产收益序列的估计。
- **cholesky**: 一个可选的 `ndarray`，表示协方差矩阵的下三角Cholesky分解。在某些模型（如因子模型）中，此分解可能具有比直接对协方差矩阵进行分解更低的维度，从而在优化计算中提升性能和收敛性。
- **sample_weight**: 一个可选的 `ndarray`，表示每个观测值的样本权重。

`frozen=True` 确保了该数据类的实例是不可变的，这对于缓存和保证数据一致性至关重要。`eq=False` 则禁用了基于字段值的相等性比较，转而使用对象的ID进行哈希。这种ID哈希机制对于在CVX优化模型中缓存 `ReturnDistribution` 实例至关重要，因为它避免了对大型数组进行昂贵的逐元素比较，从而显著提升了缓存性能。

**Section sources**
- [src/skfolio/prior/_base.py](file://src/skfolio/prior/_base.py#L17-L48)

## 继承与实现示例

要创建一个自定义的先验估计器，开发者需要继承 `BasePrior` 基类并实现其 `fit` 方法。以下是一个简化的示例，展示了如何创建一个继承自 `BasePrior` 的估计器：

```python
from skfolio.prior._base import BasePrior, ReturnDistribution
import numpy as np

class CustomPrior(BasePrior):
    def __init__(self):
        # 在此处初始化自定义参数
        pass

    def fit(self, X, y=None, **fit_params):
        # 在此处实现自定义的估计逻辑
        # 例如，计算 mu 和 covariance
        mu = np.mean(X, axis=0)
        covariance = np.cov(X, rowvar=False)
        returns = X

        # 将计算结果封装到 ReturnDistribution 中
        self.return_distribution_ = ReturnDistribution(mu=mu, covariance=covariance, returns=returns)
        return self
```

在 `skfolio` 的实际实现中，如 `EmpiricalPrior` 和 `BlackLitterman`，它们都遵循了这一模式。例如，`EmpiricalPrior` 在 `fit` 方法中会分别调用一个 `mu_estimator` 和一个 `covariance_estimator` 来计算 `mu` 和 `covariance`，然后将结果与 `X` 一起封装到 `ReturnDistribution` 中。

**Section sources**
- [src/skfolio/prior/_empirical.py](file://src/skfolio/prior/_empirical.py#L17-L205)
- [src/skfolio/prior/_black_litterman.py](file://src/skfolio/prior/_black_litterman.py#L22-L269)

## 在优化器中的传递机制

`ReturnDistribution` 是连接先验估计器和投资组合优化器的关键桥梁。在优化器（如 `MeanRisk`）的 `fit` 流程中，`prior_estimator` 参数会被调用其 `fit` 方法，从而生成一个 `return_distribution_` 实例。

这个 `return_distribution_` 实例随后被传递给优化器内部的CVXPY问题构建过程。例如，在 `ConvexOptimization` 类中，`_cvx_transaction_cost` 和 `_cvx_management_fee` 等方法会接收 `return_distribution` 作为参数，以获取 `returns` 的形状等信息来构建成本表达式。最终，`mu` 和 `covariance` 等核心参数被直接用于构建优化目标函数和约束条件。

这种设计实现了关注点的分离：先验估计器专注于数据建模和参数估计，而优化器则专注于求解最优权重。`ReturnDistribution` 作为标准化的数据载体，确保了整个流程的清晰和高效。

**Section sources**
- [src/skfolio/optimization/convex/_base.py](file://src/skfolio/optimization/convex/_base.py#L1225-L1299)
- [src/skfolio/optimization/convex/_mean_risk.py](file://src/skfolio/optimization/convex/_mean_risk.py#L585-L616)