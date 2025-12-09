# API参考

<cite>
**本文档引用的文件**
- [__init__.py](file://src/skfolio/__init__.py)
- [optimization/__init__.py](file://src/skfolio/optimization/__init__.py)
- [prior/__init__.py](file://src/skfolio/prior/__init__.py)
- [portfolio/__init__.py](file://src/skfolio/portfolio/__init__.py)
- [model_selection/__init__.py](file://src/skfolio/model_selection/__init__.py)
- [distribution/__init__.py](file://src/skfolio/distribution/__init__.py)
- [optimization/_base.py](file://src/skfolio/optimization/_base.py)
- [prior/_base.py](file://src/skfolio/prior/_base.py)
- [portfolio/_base.py](file://src/skfolio/portfolio/_base.py)
- [model_selection/_validation.py](file://src/skfolio/model_selection/_validation.py)
- [distribution/_base.py](file://src/skfolio/distribution/_base.py)
- [optimization/convex/_base.py](file://src/skfolio/optimization/convex/_base.py)
- [optimization/cluster/_nco.py](file://src/skfolio/optimization/cluster/_nco.py)
- [prior/_empirical.py](file://src/skfolio/prior/_empirical.py)
- [distribution/univariate/_gaussian.py](file://src/skfolio/distribution/univariate/_gaussian.py)
- [distribution/multivariate/_vine_copula.py](file://src/skfolio/distribution/multivariate/_vine_copula.py)
- [model_selection/_walk_forward.py](file://src/skfolio/model_selection/_walk_forward.py)
</cite>

## 目录
1. [优化模块](#优化模块)
2. [先验模块](#先验模块)
3. [投资组合模块](#投资组合模块)
4. [模型选择模块](#模型选择模块)
5. [分布模块](#分布模块)

## 优化模块

### BaseOptimization
```python
class BaseOptimization(skb.BaseEstimator, ABC):
    """
    skfolio中所有投资组合优化器的基类。

    参数
    ----------
    portfolio_params : dict, 可选
        在`predict`方法中传递给结果`Portfolio`的投资组合参数。
        如果未提供且估计器上可用，则以下属性将默认传播到投资组合：
        `name`、`transaction_costs`、`management_fees`、`previous_weights`和`risk_free_rate`。

    fallback : BaseOptimization | "previous_weights" | list[BaseOptimization | "previous_weights"], 可选
        当主优化在`fit`期间引发时，要尝试的后备估计器或估计器列表（按顺序）。
        或者，使用`"previous_weights"`（单独或在列表中）回退到估计器的`previous_weights`。
        当后备成功时，其拟合的`weights_`会复制回主估计器，因此`fit`仍然返回原始实例。
        为了可追溯性，`fallback_`存储成功的估计器（或字符串`"previous_weights"`），
        `fallback_chain_`存储每次尝试及其结果。

    previous_weights : float | dict[str, float] | array-like of shape (n_assets,), 可选
        资产的先前权重。一些估计器使用此来计算成本或换手率。
        此外，当`fallback="previous_weights"`时，如果提供，失败将回退到这些权重。

    raise_on_failure : bool, 默认=True
        控制拟合失败时的错误处理。
        如果为True，`fit`期间的任何失败都会立即引发，不会设置`weights_`，
        后续对`predict`的调用将引发`NotFittedError`。
        如果为False，不会引发错误；而是发出警告，`weights_`设置为`None`，
        后续对`predict`的调用将返回`FailedPortfolio`。
        当指定了后备时，此行为仅在所有后备都耗尽后适用。

    属性
    ----------
    weights_ : ndarray of shape (n_assets,) or (n_optimizations, n_assets)
        资产的权重。

    n_features_in_ : int
        `fit`期间看到的资产数量。

    feature_names_in_ : ndarray of shape (`n_features_in_`,)
        `fit`期间看到的资产名称。仅当`X`具有全为字符串的资产名称时定义。

    fallback_ : BaseOptimization | "previous_weights" | None
        产生最终结果的后备估计器实例，或字符串`"previous_weights"`。如果未使用后备，则为`None`。

    fallback_chain_ : list[tuple[str, str]] | None
        描述优化后备尝试的序列。每个元素是一对`(estimator_repr, outcome)`，
        其中`estimator_repr`是主估计器或后备的字符串表示（例如`"EqualWeighted()"`、`"previous_weights"`），
        `outcome`是`"success"`（如果该步骤产生了有效解），否则为字符串化的错误消息。
        对于没有后备的成功拟合，此值为`None`。

    error_ : str | list[str] | None
        `fit`失败时捕获的错误消息。对于多投资组合输出（`weights_`为2D），这是一个与投资组合对齐的列表。

    注意
    -----
    所有估计器都应在`__init__`中将所有参数指定为显式关键字参数（无`*args`或`**kwargs`），
    遵循scikit-learn约定。
    """
```

### BaseComposition
```python
class BaseComposition(BaseOptimization, ABC):
    """
    组合优化估计器的基类。
    """
```

### BaseHierarchicalOptimization
```python
class BaseHierarchicalOptimization(BaseOptimization, ABC):
    """
    分层优化估计器的基类。
    """
```

### BaseConvexOptimization
```python
class ConvexOptimization(BaseOptimization, ABC):
    """
    skfolio中所有凸优化估计器的基类。

    参数
    ----------
    risk_measure : RiskMeasure, 默认=RiskMeasure.VARIANCE
        优化的`RiskMeasure`。可以是以下任何一种：
            * VARIANCE
            * SEMI_VARIANCE
            * STANDARD_DEVIATION
            * SEMI_DEVIATION
            * MEAN_ABSOLUTE_DEVIATION
            * FIRST_LOWER_PARTIAL_MOMENT
            * CVAR
            * EVAR
            * WORST_REALIZATION
            * CDAR
            * MAX_DRAWDOWN
            * AVERAGE_DRAWDOWN
            * EDAR
            * ULCER_INDEX
            * GINI_MEAN_DIFFERENCE_RATIO
        默认值为`RiskMeasure.VARIANCE`。

    prior_estimator : BasePrior, 可选
        先验估计器。先验估计器用于估计包含资产预期收益、协方差矩阵、
        收益和协方差的Cholesky分解估计的`ReturnDistribution`。
        默认（`None`）是使用`EmpiricalPrior`。

    min_weights : float | dict[str, float] | array-like of shape (n_assets, ) | None, 默认=0.0
        资产权重的最小值（权重下限）。如果提供浮点数，则应用于每个资产。
        `None`等同于`-np.Inf`（无下限）。如果提供字典，其（键/值）对必须是
        （资产名称/资产最小权重），且`fit`方法的输入`X`必须是
        资产名称在列中的DataFrame。使用字典时，未提供的资产值被分配
        0.0的最小权重。默认值为0.0（无卖空）。

    max_weights : float | dict[str, float] | array-like of shape (n_assets, ) | None, 默认=1.0
        资产权重的最大值（权重大限）。如果提供浮点数，则应用于每个资产。
        `None`等同于`+np.Inf`（无上限）。如果提供字典，其（键/值）对必须是
        （资产名称/资产最大权重），且`fit`方法的输入`X`必须是
        资产名称在列中的DataFrame。使用字典时，未提供的资产值被分配
        1.0的最大权重。默认值为1.0（每个资产低于100%）。

    budget : float | None, 默认=1.0
        投资预算。它是多头头寸和空头头寸总和（所有权重的总和）。
        `None`表示无预算约束。默认值为1.0（完全投资组合）。

    min_budget : float, 可选
        最小预算。它是多头和空头头寸总和（所有权重的总和）的下限。
        如果提供，必须设置`budget=None`。默认（`None`）表示无最小预算约束。

    max_budget : float, 可选
        最大预算。它是多头和空头头寸总和（所有权重的总和）的上限。
        如果提供，必须设置`budget=None`。默认（`None`）表示无最大预算约束。

    max_short : float, 可选
        最大空头头寸。空头头寸定义为负权重的总和（绝对值）。
        默认（`None`）表示无最大空头头寸。

    max_long : float, 可选
        最大长头头寸。长头头寸定义为正权重的总和。
        默认（`None`）表示无最大长头头寸。

    cardinality : int, 可选
        指定基数约束以限制投资资产数量（非零权重）。
        此功能需要混合整数求解器。对于开源选项，我们建议使用SCIP，
        通过设置`solver="SCIP"`。要安装它，请使用：`pip install cvxpy[SCIP]`。
        对于商业求解器，支持的选项包括MOSEK、GUROBI或CPLEX。

    group_cardinalities : dict[str, int], 可选
        指定特定资产组基数约束的字典。键表示组名称（字符串），
        值指定每组允许的最大资产数量。必须使用`groups`参数提供组。
        这需要混合整数求解器（有关详细信息，请参见`cardinality`）。

    threshold_long : float | dict[str, float] | array-like of shape (n_assets, ), 可选
        指定投资组合中被视为长头头寸的资产的最小权重阈值。
        权重低于此阈值的资产将不包括在投资组合的长头头寸中。
        此约束有助于消除不重要的分配。这需要混合整数求解器
        （有关详细信息，请参见`cardinality`）。其格式与`min_weights`和`max_weights`相同。

    threshold_short : float | dict[str, float] | array-like of shape (n_assets, ), 可选
        指定投资组合中被视为空头头寸的资产的最大权重阈值。
        权重高于此阈值的资产将不包括在投资组合的空头头寸中。
        此约束有助于控制空头头寸的幅度。这需要混合整数求解器
        （有关详细信息，请参见`cardinality`）。其格式与`min_weights`和`max_weights`相同。

    transaction_costs : float | dict[str, float] | array-like of shape (n_assets, ), 默认=0.0
        资产的交易成本。用于在优化问题中添加线性交易成本：
        total_cost = sum(c_i * |w_i - w_prev_i|)
        其中c_i是资产i的交易成本，w_i是其权重，w_prev_i是其先前权重
        （在`previous_weights`中定义）。浮点数total_cost影响优化中的投资组合预期收益：
        expected_return = mu^T * w - total_cost
        其中mu是资产预期收益的向量，w是资产权重的向量。
        如果提供浮点数，则应用于每个资产。如果提供字典，其（键/值）对必须是
        （资产名称/资产成本），且`fit`方法的输入`X`必须是
        资产名称在列中的DataFrame。默认值为0.0。

    management_fees : float | dict[str, float] | array-like of shape (n_assets, ), 默认=0.0
        资产的管理费。用于在优化问题中添加线性管理费：
        total_fee = sum(f_i * w_i)
        其中f_i是资产i的管理费，w_i是其权重。浮点数total_fee影响优化中的投资组合预期收益：
        expected_return = mu^T * w - total_fee
        其中mu是资产预期收益的向量，w是资产权重的向量。
        如果提供浮点数，则应用于每个资产。如果提供字典，其（键/值）对必须是
        （资产名称/资产费），且`fit`方法的输入`X`必须是
        资产名称在列中的DataFrame。默认值为0.0。

    previous_weights : float | dict[str, float] | array-like of shape (n_assets, ), 可选
        资产的先前权重。先前权重用于计算投资组合成本和投资组合换手率。
        如果提供浮点数，则应用于每个资产。如果提供字典，其（键/值）对必须是
        （资产名称/资产先前权重），且`fit`方法的输入`X`必须是
        资产名称在列中的DataFrame。默认（`None`）表示无先前权重。
        此外，当`fallback="previous_weights"`时，如果提供，失败将回退到这些权重。

    l1_coef : float, 默认=0.0
        L1正则化系数。用于通过L1范数惩罚目标函数：
        l1_coef * ||w||_1 = l1_coef * sum(|w_i|)
        增加此系数将减少非零权重的数量（基数）。它倾向于增加稳健性
        （样本外稳定性）但减少分散化。默认值为0.0。

    l2_coef : float, 默认=0.0
        L2正则化系数。用于通过L2范数惩罚目标函数：
        l2_coef * ||w||_2^2 = l2_coef * sum(w_i^2)
        它倾向于增加稳健性（样本外稳定性）。默认值为0.0。

    mu_uncertainty_set_estimator : BaseMuUncertaintySet, 可选
        Mu不确定性集估计器。如果提供，资产预期收益将用椭球不确定性集建模。
        这称为最坏情况优化，是稳健优化的一种。它减少了由预期收益估计误差
        引起的不稳定性。默认（`None`）表示不使用不确定性集。

    covariance_uncertainty_set_estimator : BaseCovarianceUncertaintySet, 可选
        协方差不确定性集估计器。如果提供，资产协方差矩阵将用椭球不确定性集建模。
        这称为最坏情况优化，是稳健优化的一种。它减少了由协方差矩阵估计误差
        引起的不稳定性。默认（`None`）表示不使用不确定性集。

    linear_constraints : array-like of shape (n_constraints,), 可选
        线性约束。线性约束必须匹配以下任何模式：
            * "ref1 >= a"
            * "ref1 == b"
            * "ref1 >= ref1"
            * "a * ref1 + b * ref2 + c <= d * ref3"
        其中"ref1"、"ref2"...是资产名称或在`groups`参数中提供的组名称。
        如果`fit`方法的输入`X`是具有这些资产名称在列中的DataFrame，
        则资产名称可以在不使用`groups`的情况下引用。

    groups : dict[str, list[str]] or array-like of shape (n_groups, n_assets), 可选
        在`linear_constraints`中引用的资产组。如果提供字典，其（键/值）对必须是
        （资产名称/资产组），且`fit`方法的输入`X`必须是
        资产名称在列中的DataFrame。

    left_inequality : array-like of shape (n_constraints, n_assets), 可选
        线性约束A * w <= b的左侧不等式矩阵A。

    right_inequality : array-like of shape (n_constraints, ), 可选
        线性约束A * w <= b的右侧不等式向量b。

    risk_free_rate : float, 默认=0.0
        无风险利率。默认值为0.0。

    min_acceptable_return : float, 可选
        用于区分“下行”和“上行”收益以计算低阶矩的最低可接受收益：
            * 第一低阶矩
            * 半方差
            * 半偏差
        默认（`None`）是使用均值。

    cvar_beta : float, 默认=0.95
        CVaR（条件风险价值）置信水平。默认值为0.95。

    evar_beta : float, 默认=0.95
        EVaR（熵风险价值）置信水平。默认值为0.95。

    cdar_beta : float, 默认=0.95
        CDaR（条件回撤风险）置信水平。默认值为0.95。

    edar_beta : float, 默认=0.95
        EDaR（熵回撤风险）置信水平。默认值为0.95。

    add_objective : Callable[[cp.Variable], cp.Expression], 可选
        将自定义目标添加到现有目标表达式。它是一个函数，必须以权重`w`作为参数，
        并返回CVXPY表达式。

    add_constraints : Callable[[cp.Variable], cp.Expression|list[cp.Expression]], 可选
        将自定义约束或约束列表添加到现有约束。它是一个函数，必须以权重`w`作为参数，
        并返回CVXPY表达式或CVXPY表达式列表。

    overwrite_expected_return : Callable[[cp.Variable], cp.Expression], 可选
        用自定义表达式覆盖预期收益mu * w。它是一个函数，必须以权重`w`作为参数，
        并返回CVXPY表达式。

    solver : str, 默认="CLARABEL"
        要使用的求解器。默认是"CLARABEL"，用Rust编写，具有比ECOS和SCS更好的数值稳定性和性能。
        Cvxpy将在未来版本中将其默认求解器"ECOS"替换为"CLARABEL"。

    solver_params : dict, 可选
        求解器参数。例如，`solver_params=dict(verbose=True)`。
        默认（`None`）是为求解器"CLARABEL"使用`{"tol_gap_abs": 1e-9, "tol_gap_rel": 1e-9}`，
        否则使用CVXPY默认值。

    scale_objective : float, 可选
        按此值缩放每个目标元素。可用于在特定情况下提高优化准确性。
        默认（`None`）根据问题设置。

    scale_constraints : float, 可选
        按此值缩放每个约束元素。可用于在特定情况下提高优化准确性。
        默认（`None`）根据问题设置。

    save_problem : bool, 默认=False
        如果设置为True，CVXPY问题将保存在`problem_`中。默认是`False`。

    portfolio_params : dict, 可选
        投资组合参数，传递给`predict`中的结果`Portfolio`。
        如果未提供且估计器上可用，则以下属性将默认传播到投资组合：
        `name`、`transaction_costs`、`management_fees`、`previous_weights`和`risk_free_rate`。

    fallback : BaseOptimization | "previous_weights" | list[BaseOptimization | "previous_weights"], 可选
        当主优化在`fit`期间引发时，要尝试的后备估计器或估计器列表（按顺序）。
        或者，使用`"previous_weights"`（单独或在列表中）回退到估计器的`previous_weights`。
        当后备成功时，其拟合的`weights_`会复制回主估计器，因此`fit`仍然返回原始实例。
        为了可追溯性，`fallback_`存储成功的估计器（或字符串`"previous_weights"`），
        `fallback_chain_`存储每次尝试及其结果。

    raise_on_failure : bool, 默认=True
        控制拟合失败时的错误处理。
        如果为True，`fit`期间的任何失败都会立即引发，不会设置`weights_`，
        后续对`predict`的调用将引发`NotFittedError`。
        如果为False，不会引发错误；而是发出警告，`weights_`设置为`None`，
        后续对`predict`的调用将返回`FailedPortfolio`。当指定了后备时，
        此行为仅在所有后备都耗尽后适用。

    属性
    ----------
    weights_ : ndarray of shape (n_assets,) or (n_optimizations, n_assets)
        资产的权重。

    problem_values_ : dict[str, float] | list[dict[str, float]] of size n_optimizations
        从CVXPY问题中检索的表达式值。

    prior_estimator_ : BasePrior
        拟合的`prior_estimator`。

    mu_uncertainty_set_estimator_ : BaseMuUncertaintySet
        拟合的`mu_uncertainty_set_estimator`（如果提供）。

    covariance_uncertainty_set_estimator_ : BaseCovarianceUncertaintySet
        拟合的`covariance_uncertainty_set_estimator`（如果提供）。

    problem_: cvxpy.Problem
        用于优化的CVXPY问题。仅当`save_problem`设置为`True`时。

    fallback_ : BaseOptimization | "previous_weights" | None
        产生最终结果的后备估计器实例，或字符串`"previous_weights"`。如果未使用后备，则为`None`。

    fallback_chain_ : list[tuple[str, str]] | None
        描述优化后备尝试的序列。每个元素是一对`(estimator_repr, outcome)`，
        其中`estimator_repr`是主估计器或后备的字符串表示（例如`"EqualWeighted()"`、`"previous_weights"`），
        `outcome`是`"success"`（如果该步骤产生了有效解），否则为字符串化的错误消息。
        对于没有后备的成功拟合，此值为`None`。

    error_ : str | list[str] | None
        `fit`失败时捕获的错误消息。对于多投资组合输出（`weights_`为2D），这是一个与投资组合对齐的列表。

    注意
    -----
    所有估计器都应在`__init__`中将所有参数指定为显式关键字参数（无`*args`或`**kwargs`），
    遵循scikit-learn约定。
    """
```

### ObjectiveFunction
```python
class ObjectiveFunction(AutoEnum):
    """
    目标函数的枚举。

    属性
    ----------
    MINIMIZE_RISK : str
        最小化风险度量。

    MAXIMIZE_RETURN : str
        最大化预期收益。

    MAXIMIZE_UTILITY : str
        最大化效用 w^T*mu - lambda * risk(w)。

    MAXIMIZE_RATIO : str
        最大化比率 (w^T*mu - R_f) / risk(w)。
    """
```

### BenchmarkTracker
```python
class BenchmarkTracker(ConvexOptimization):
    """
    基准跟踪优化估计器。

    该估计器通过最小化投资组合收益与基准收益之间的方差来优化投资组合，
    从而跟踪基准。

    参数
    ----------
    benchmark_weights : float | dict[str, float] | array-like of shape (n_assets, ), 可选
        基准权重。如果提供，将使用这些权重计算基准收益。
        如果未提供，将使用`benchmark`参数。

    benchmark : array-like of shape (n_observations, ), 可选
        基准收益。如果提供，将使用这些收益作为基准。
        如果未提供，将使用`benchmark_weights`参数。

    **kwargs
        传递给`ConvexOptimization`父类的其他参数。
    """
```

### ConvexOptimization
```python
class ConvexOptimization(BaseOptimization, ABC):
    """
    所有凸优化估计器的基类。
    """
```

### DistributionallyRobustCVaR
```python
class DistributionallyRobustCVaR(ConvexOptimization):
    """
    分布鲁棒CVaR优化估计器。

    该估计器通过最小化分布鲁棒CVaR来优化投资组合。

    参数
    ----------
    kappa : float, 默认=1.0
        不确定性集的大小参数。

    **kwargs
        传递给`ConvexOptimization`父类的其他参数。
    """
```

### EqualWeighted
```python
class EqualWeighted(BaseOptimization):
    """
    等权重优化估计器。

    该估计器为所有资产分配相等的权重。

    参数
    ----------
    **kwargs
        传递给`BaseOptimization`父类的其他参数。
    """
```

### HierarchicalEqualRiskContribution
```python
class HierarchicalEqualRiskContribution(BaseHierarchicalOptimization):
    """
    分层等风险贡献优化估计器。

    该估计器使用分层聚类将资产分组，然后在每个组内分配相等的风险贡献。

    参数
    ----------
    risk_measure : RiskMeasure, 默认=RiskMeasure.VARIANCE
        用于风险贡献计算的风险度量。

    distance_estimator : BaseDistance, 可选
        用于计算资产间距离的距离估计器。

    clustering_estimator : BaseEstimator, 可选
        用于计算资产聚类的聚类估计器。

    **kwargs
        传递给`BaseHierarchicalOptimization`父类的其他参数。
    """
```

### HierarchicalRiskParity
```python
class HierarchicalRiskParity(BaseHierarchicalOptimization):
    """
    分层风险平价优化估计器。

    该估计器使用分层聚类将资产分组，然后在每个组内分配风险平价权重。

    参数
    ----------
    risk_measure : RiskMeasure, 默认=RiskMeasure.VARIANCE
        用于风险平价计算的风险度量。

    distance_estimator : BaseDistance, 可选
        用于计算资产间距离的距离估计器。

    clustering_estimator : BaseEstimator, 可选
        用于计算资产聚类的聚类估计器。

    **kwargs
        传递给`BaseHierarchicalOptimization`父类的其他参数。
    """
```

### InverseVolatility
```python
class InverseVolatility(BaseOptimization):
    """
    反向波动率优化估计器。

    该估计器根据资产的波动率的倒数分配权重。

    参数
    ----------
    **kwargs
        传递给`BaseOptimization`父类的其他参数。
    """
```

### MaximumDiversification
```python
class MaximumDiversification(ConvexOptimization):
    """
    最大分散化优化估计器。

    该估计器通过最大化分散化比率来优化投资组合。

    参数
    ----------
    **kwargs
        传递给`ConvexOptimization`父类的其他参数。
    """
```

### MeanRisk
```python
class MeanRisk(ConvexOptimization):
    """
    均值-风险优化估计器。

    该估计器通过最大化效用函数（预期收益减去风险惩罚）来优化投资组合。

    参数
    ----------
    risk_measure : RiskMeasure, 默认=RiskMeasure.VARIANCE
        用于优化的风险度量。

    risk_aversion : float, 默认=1.0
        风险厌恶系数。

    **kwargs
        传递给`ConvexOptimization`父类的其他参数。
    """
```

### NestedClustersOptimization
```python
class NestedClustersOptimization(BaseOptimization):
    """
    嵌套聚类优化估计器。

    嵌套聚类优化（NCO）是一种投资组合优化方法，由Marcos Lopez de Prado开发。
    它使用距离矩阵通过聚类算法（分层树聚类、KMeans等）计算聚类。
    对于每个聚类，内部聚类权重通过在整个训练数据上拟合内部估计器来计算。
    然后，外部聚类权重通过使用内部估计器的样本外估计（通过交叉验证）
    拟合外部估计器来计算。最后，最终资产权重是内部权重和外部权重的点积。

    参数
    ----------
    inner_estimator : BaseOptimization, 可选
        用于估计内部权重（也称为内部权重）的优化估计器，
        这些权重是每个聚类内的资产权重。默认`None`是使用`MeanRisk`。

    outer_estimator : BaseOptimization, 可选
        用于估计外部权重（也称为外部权重）的优化估计器，
        这些权重是应用于每个聚类的权重。默认`None`是使用`MeanRisk`。

    distance_estimator : BaseDistance, 可选
        距离估计器。距离估计器用于估计共依存性和距离矩阵，
        用于计算链接矩阵。默认（`None`）是使用`PearsonDistance`。

    clustering_estimator : BaseEstimator, 可选
        聚类估计器。必须在拟合后暴露`labels_`属性。
        聚类估计器用于基于距离矩阵计算资产的聚类。
        默认（`None`）是使用`HierarchicalClustering`。

    cv : BaseCrossValidator | BaseCombinatorialCV | int | "ignore", 可选
        确定交叉验证拆分策略。默认（`None`）是使用5折交叉验证`KFold()`。
        它应用于内部估计器。其样本外输出用于训练外部估计器。

    n_jobs : int, 可选
        并行运行`fit`的作业数。值`-1`表示使用所有处理器。
        默认（`None`）表示1，除非在`joblib.parallel_backend`上下文中。

    quantile : float, 默认=0.5
        当`cv`参数是`CombinatorialPurgedCV`交叉验证器时，
        内部估计器路径的样本外输出的分位数（针对给定度量`quantile_measure`）。
        默认值为0.5，对应于中位数度量的路径。

    quantile_measure : PerfMeasure or RatioMeasure or RiskMeasure or ExtraRiskMeasure, 默认=RatioMeasure.SHARPE_RATIO
        用于分位数路径选择的度量（参见`quantile`和`cv`）。
        默认是`RatioMeasure.SHARPE_RATIO`。

    verbose : int, 默认=0
        详细级别。默认值为0。

    portfolio_params : dict, 可选
        投资组合参数，传递给`predict`中的结果`Portfolio`。
        如果未提供且估计器上可用，则以下属性将默认传播到投资组合：
        `name`和`previous_weights`。

    fallback : BaseOptimization | "previous_weights" | list[BaseOptimization | "previous_weights"], 可选
        当主优化在`fit`期间引发时，要尝试的后备估计器或估计器列表（按顺序）。
        或者，使用`"previous_weights"`（单独或在列表中）回退到估计器的`previous_weights`。
        当后备成功时，其拟合的`weights_`会复制回主估计器，因此`fit`仍然返回原始实例。
        为了可追溯性，`fallback_`存储成功的估计器（或字符串`"previous_weights"`），
        `fallback_chain_`存储每次尝试及其结果。

    previous_weights : float | dict[str, float] | array-like of shape (n_assets,), 可选
        当`fallback="previous_weights"`时，如果提供，失败将回退到这些权重。

    raise_on_failure : bool, 默认=True
        控制拟合失败时的错误处理。
        如果为True，`fit`期间的任何失败都会立即引发，不会设置`weights_`，
        后续对`predict`的调用将引发`NotFittedError`。
        如果为False，不会引发错误；而是发出警告，`weights_`设置为`None`，
        后续对`predict`的调用将返回`FailedPortfolio`。当指定了后备时，
        此行为仅在所有后备都耗尽后适用。

    属性
    ----------
    weights_ : ndarray of shape (n_assets,)
        资产的权重。

    distance_estimator_ : BaseDistance
        拟合的`distance_estimator`。

    inner_estimators_ : list[BaseOptimization]
        拟合的`inner_estimator`列表。每个包含多个资产的聚类一个。

    outer_estimator_ : BaseOptimization
        拟合的`outer_estimator`。

    clustering_estimator_ : BaseEstimator
        拟合的`clustering_estimator`。

    n_features_in_ : int
        `fit`期间看到的资产数量。

    feature_names_in_ : ndarray of shape (`n_features_in_`,)
        `fit`期间看到的资产名称。仅当`X`具有全为字符串的资产名称时定义。

    fallback_ : BaseOptimization | "previous_weights" | None
        产生最终结果的后备估计器实例，或字符串`"previous_weights"`。如果未使用后备，则为`None`。

    fallback_chain_ : list[tuple[str, str]] | None
        描述优化后备尝试的序列。每个元素是一对`(estimator_repr, outcome)`，
        其中`estimator_repr`是主估计器或后备的字符串表示（例如`"EqualWeighted()"`、`"previous_weights"`），
        `outcome`是`"success"`（如果该步骤产生了有效解），否则为字符串化的错误消息。
        对于没有后备的成功拟合，此值为`None`。

    error_ : str | list[str] | None
        `fit`失败时捕获的错误消息。对于多投资组合输出（`weights_`为2D），这是一个与投资组合对齐的列表。

    注意
    -----
    所有估计器都应在`__init__`中将所有参数指定为显式关键字参数（无`*args`或`**kwargs`），
    遵循scikit-learn约定。

    参考
    ----------
    .. [1]  "Building diversified portfolios that outperform out of sample",
        The Journal of Portfolio Management,
        Marcos López de Prado (2016)

    .. [2]  "A robust estimator of the efficient frontier",
        SSRN Electronic Journal,
        Marcos López de Prado (2019)

    .. [3] "Machine Learning for Asset Managers",
        Elements in Quantitative Finance. Cambridge University Press,
        Marcos López de Prado (2020)
    """
```

### Random
```python
class Random(BaseOptimization):
    """
    随机优化估计器。

    该估计器为资产分配随机权重。

    参数
    ----------
    **kwargs
        传递给`BaseOptimization`父类的其他参数。
    """
```

### RiskBudgeting
```python
class RiskBudgeting(ConvexOptimization):
    """
    风险预算优化估计器。

    该估计器通过最小化风险预算与目标风险预算之间的差异来优化投资组合。

    参数
    ----------
    risk_budgets : float | dict[str, float] | array-like of shape (n_assets, ), 可选
        目标风险预算。如果提供，将使用这些预算。

    **kwargs
        传递给`ConvexOptimization`父类的其他参数。
    """
```

### SchurComplementary
```python
class SchurComplementary(BaseHierarchicalOptimization):
    """
    Schur互补优化估计器。

    该估计器使用Schur互补方法进行分层优化。

    参数
    ----------
    **kwargs
        传递给`BaseHierarchicalOptimization`父类的其他参数。
    """
```

### StackingOptimization
```python
class StackingOptimization(BaseComposition):
    """
    堆叠优化估计器。

    该估计器通过堆叠多个基础估计器的输出来优化投资组合。

    参数
    ----------
    estimators : list[BaseOptimization], 可选
        基础估计器列表。

    meta_estimator : BaseOptimization, 可选
        用于组合基础估计器输出的元估计器。

    **kwargs
        传递给`BaseComposition`父类的其他参数。
    """
```

**Section sources**
- [optimization/__init__.py](file://src/skfolio/optimization/__init__.py)
- [optimization/_base.py](file://src/skfolio/optimization/_base.py)
- [optimization/convex/_base.py](file://src/skfolio/optimization/convex/_base.py)
- [optimization/cluster/_nco.py](file://src/skfolio/optimization/cluster/_nco.py)

## 先验模块

### BasePrior
```python
class BasePrior(skb.BaseEstimator, ABC):
    """
    skfolio中所有先验估计器的基类。

    注意
    -----
    所有估计器都应在`__init__`中将所有参数指定为显式关键字参数（无`*args`或`**kwargs`），
    遵循scikit-learn约定。
    """
```

### BaseLoadingMatrix
```python
class BaseLoadingMatrix(ABC):
    """
    因子模型加载矩阵的基类。
    """
```

### BlackLitterman
```python
class BlackLitterman(BasePrior):
    """
    Black-Litterman先验估计器。

    该估计器使用Black-Litterman模型结合市场均衡收益和投资者观点来估计预期收益。

    参数
    ----------
    market_caps : array-like of shape (n_assets, ), 可选
        资产的市值。用于计算市场均衡收益。

    views : dict[str, float], 可选
        投资者观点。字典的键是资产名称，值是预期超额收益。

    view_confidences : dict[str, float], 可选
        观点置信度。字典的键是资产名称，值是置信度。

    tau : float, 默认=1.0
        观点不确定性缩放参数。

    **kwargs
        传递给`BasePrior`父类的其他参数。
    """
```

### EmpiricalPrior
```python
class EmpiricalPrior(BasePrior):
    """
    经验先验估计器。

    经验先验通过分别拟合`mu_estimator`和`covariance_estimator`来估计`ReturnDistribution`。

    参数
    ----------
    mu_estimator : BaseMu, 可选
        资产预期收益估计器。默认（`None`）是使用`EmpiricalMu`。

    covariance_estimator : BaseCovariance , 可选
        资产协方差矩阵估计器。默认（`None`）是使用`EmpiricalCovariance`。

    is_log_normal : bool, 默认=False
        如果设置为True，将在对数收益上估计矩，而不是线性收益。
        然后，对数收益的矩估计将投影到投资期限并转换以获得投资期限上线性收益的矩估计。
        如果为True，必须提供`investment_horizon`。输入`X`必须是**线性收益**。
        它们将仅在矩估计时转换为对数收益。

    investment_horizon : float, 可选
        当`is_log_normal`为`True`时，用于线性收益矩估计的投资期限。

    属性
    ----------
    return_distribution_ : ReturnDistribution
        拟合的`ReturnDistribution`，供优化估计器使用，包含资产收益分布和矩估计。

    mu_estimator_ : BaseMu
        拟合的`mu_estimator`。

    covariance_estimator_ : BaseCovariance
        拟合的`covariance_estimator`。

    n_features_in_ : int
       `fit`期间看到的资产数量。

    feature_names_in_ : ndarray of shape (`n_features_in_`,)
       `fit`期间看到的特征名称。仅当`X`具有全为字符串的特征名称时定义。

    参考
    ----------
    .. [1]  "Linear vs. Compounded Returns - Common Pitfalls in Portfolio Management".
        GARP Risk Professional.
        Attilio Meucci (2010).
    """
```

### EntropyPooling
```python
class EntropyPooling(BasePrior):
    """
    熵池化先验估计器。

    该估计器使用熵池化方法结合市场先验和投资者观点来估计预期收益。

    参数
    ----------
    market_prior : BasePrior, 可选
        市场先验估计器。默认是使用`EmpiricalPrior`。

    views : dict[str, float], 可选
        投资者观点。字典的键是资产名称，值是预期超额收益。

    view_confidences : dict[str, float], 可选
        观点置信度。字典的键是资产名称，值是置信度。

    **kwargs
        传递给`BasePrior`父类的其他参数。
    """
```

### FactorModel
```python
class FactorModel(BasePrior):
    """
    因子模型先验估计器。

    该估计器使用因子模型估计预期收益。

    参数
    ----------
    factors : array-like of shape (n_observations, n_factors), 可选
        因子收益。

    loading_matrix_estimator : BaseLoadingMatrix, 可选
        加载矩阵估计器。默认是使用`LoadingMatrixRegression`。

    **kwargs
        传递给`BasePrior`父类的其他参数。
    """
```

### LoadingMatrixRegression
```python
class LoadingMatrixRegression(BaseLoadingMatrix):
    """
    回归加载矩阵估计器。

    该估计器使用回归方法估计因子模型的加载矩阵。

    参数
    ----------
    **kwargs
        传递给`BaseLoadingMatrix`父类的其他参数。
    """
```

### OpinionPooling
```python
class OpinionPooling(BasePrior):
    """
    观点池化先验估计器。

    该估计器使用观点池化方法结合多个投资者观点来估计预期收益。

    参数
    ----------
    opinions : list[dict[str, float]], 可选
        投资者观点列表。每个字典的键是资产名称，值是预期超额收益。

    opinion_confidences : list[dict[str, float]], 可选
        观点置信度列表。每个字典的键是资产名称，值是置信度。

    **kwargs
        传递给`BasePrior`父类的其他参数。
    """
```

### ReturnDistribution
```python
@dataclass(frozen=True, eq=False)
class ReturnDistribution:
    """
    优化估计器使用的收益分布数据类。

    属性
    ----------
    mu : ndarray of shape (n_assets,)
        资产预期收益的估计。

    covariance : ndarray of shape (n_assets, n_assets)
        资产协方差矩阵的估计。

    returns : ndarray of shape (n_observations, n_assets)
        资产收益的估计。

    cholesky : ndarray, 可选
        协方差的下三角Cholesky因子。在某些情况下，可以获得比直接对协方差估计应用Cholesky分解
        得到的维度更小的Cholesky因子（例如在因子模型中）。如果提供，此Cholesky因子在某些优化中使用
        （例如均值-方差）以提高性能和收敛性。默认是`None`。

    sample_weight : ndarray of shape (n_observations,), 可选
        每个观察的样本权重。如果为None，假设相等权重。
    """
```

### SyntheticData
```python
class SyntheticData(BasePrior):
    """
    合成数据先验估计器。

    该估计器使用合成数据生成方法估计预期收益。

    参数
    ----------
    n_samples : int, 默认=1000
        生成的样本数。

    **kwargs
        传递给`BasePrior`父类的其他参数。
    """
```

**Section sources**
- [prior/__init__.py](file://src/skfolio/prior/__init__.py)
- [prior/_base.py](file://src/skfolio/prior/_base.py)
- [prior/_empirical.py](file://src/skfolio/prior/_empirical.py)

## 投资组合模块

### BasePortfolio
```python
class BasePortfolio:
    r"""
    skfolio中所有投资组合的基类。

    参数
    ----------
    returns : array-like of shape (n_observations,)
        投资组合收益的向量。

    observations : array-like of shape (n_observations,)
        投资组合观察的向量。

    name : str, 可选
        投资组合的名称。
        默认（`None`）是使用对象id。

    tag : str, 可选
        分配给投资组合的标签。
        标签用于从`Population`中操作投资组合组。

    fitness_measures : list[measures], 可选
        适应度度量列表。
        适应度度量用于计算投资组合适应度，用于计算支配关系。
        默认（`None`）是使用列表[PerfMeasure.MEAN, RiskMeasure.VARIANCE]

    annualized_factor : float, 默认=252.0
        使用平方根规则对以下度量进行年化的因子：
            * 年化均值 = 均值 * 因子
            * 年化方差 = 方差 * 因子
            * 年化半方差 = 半方差 * 因子
            * 年化标准差 = 标准差 * sqrt(因子)
            * 年化半偏差 = 半偏差 * sqrt(因子)
            * 年化夏普比率 = 夏普比率 * sqrt(因子)
            * 年化索提诺比率 = 索提诺比率 * sqrt(因子)

    risk_free_rate : float, 默认=0.0
        无风险利率。默认值是`0.0`。

    compounded : bool, 默认=False
        如果设置为True，累积收益将复合。
        默认是`False`。

    sample_weight : ndarray of shape (n_observations,), 可选
        每个观察的样本权重。权重必须总和为一。
         如果为None，假设相等权重。

    min_acceptable_return : float, 可选
        用于区分“下行”和“上行”收益以计算低阶矩的最低可接受收益：
            * 第一低阶矩
            * 半方差
            * 半偏差
        默认（`None`）是使用均值。

    value_at_risk_beta : float, 默认=0.95
        投资组合VaR（风险价值）的置信水平，表示最差(1-beta)%观察的收益。
        默认值是`0.95`。

    entropic_risk_measure_theta : float, 默认=1.0
        投资组合熵风险度量的风险厌恶水平。
        默认值是`1.0`。

    entropic_risk_measure_beta : float, 默认=0.95
        投资组合熵风险度量的置信水平。
        默认值是`0.95`。

    cvar_beta : float, 默认=0.95
        投资组合CVaR（条件风险价值）的置信水平，表示最差(1-beta)%观察的预期VaR。
        默认值是`0.95`。

    evar_beta : float, 默认=0.95
        投资组合EVaR（熵风险价值）的置信水平。
        默认值是`0.95`。

    drawdown_at_risk_beta : float, 默认=0.95
        投资组合回撤风险（DaR）的置信水平，表示最差(1-beta)%观察的回撤。
        默认值是`0.95`。

    cdar_beta : float, 默认=0.95
        投资组合CDaR（条件回撤风险）的置信水平。
        默认值是`0.95`。

    edar_beta : float, 默认=0.95
        投资组合EDaR（熵回撤风险）的置信水平。
        默认值是`0.95`。

    属性
    ----------
    n_observations : float
        观察数量。

    mean : float
        投资组合收益的均值。

    annualized_mean : float
        通过均值 * annualization_factor年化的均值

    mean_absolute_deviation : float
        平均绝对偏差。偏差是收益与最低可接受收益（`min_acceptable_return`）之间的差。

    first_lower_partial_moment : float
        第一低阶矩。第一低阶矩是低于最低可接受收益（`min_acceptable_return`）的收益的均值。

    variance : float
        方差（第二矩）

    annualized_variance : float
        通过方差 * annualization_factor年化的方差

    semi_variance : float
        半方差（第二低阶矩）。
        半方差是低于最低可接受收益（`min_acceptable_return`）的收益的方差。

    annualized_semi_variance : float
        通过半方差 * annualization_factor年化的半方差

    standard_deviation : float
        标准差（第二矩的平方根）。

    annualized_standard_deviation : float
        通过标准差 * sqrt(annualization_factor)年化的标准差

    semi_deviation : float
        半偏差（第二低阶矩的平方根）。
        半标准差是低于最低可接受收益（`min_acceptable_return`）的收益的标准差。

    annualized_semi_deviation : float
        通过半偏差 * sqrt(annualization_factor)年化的半偏差

    skew : float
        偏度。偏度是分布偏斜度的度量。
        对称分布的偏度为零。
        更高的偏度对应于更长的右尾。

    kurtosis : float
        峰度。它是分布尾部厚重的度量。
        更高的峰度对应于更大的极端偏差（厚尾）。

    fourth_central_moment : float
       第四中心矩。

    fourth_lower_partial_moment : float
        第四低阶矩。它是低于最低可接受收益（`min_acceptable_return`）的收益的下行尾部厚重的度量。
        更高的第四低阶矩对应于更大的下行极端偏差（下行厚尾）。

    worst_realization : float
        最差实现，即最差收益。

    value_at_risk : float
        历史VaR（风险价值）。
        VaR是在给定置信水平（`value_at_risk_beta`）下的最大损失。

    cvar : float
        历史CVaR（条件风险价值）。CVaR（或尾部VaR）表示在指定置信水平（`cvar_beta`）下的平均短缺。

    entropic_risk_measure : float
        历史熵风险度量。它是依赖于投资者风险厌恶（`entropic_risk_measure_theta`）的
        风险度量，通过指数效用函数在给定置信水平（`entropic_risk_measure_beta`）下。

    evar : float
         历史EVaR（熵风险价值）。它是一个一致风险度量，是VaR和CVaR的上界，
         从Chernoff不等式在给定置信水平（`evar_beta`）下获得。EVaR可以
         通过相对熵的概念表示。

    drawdown_at_risk : float
        历史回撤风险。它是在给定置信水平（`drawdown_at_risk_beta`）下的最大回撤。

    cdar : float
        在给定置信水平（`cdar_beta`）下的历史CDaR（条件回撤风险）。

    max_drawdown : float
        最大回撤。

    average_drawdown : float
        平均回撤。

    edar : float
        EDaR（熵回撤风险）。它是一个一致风险度量，是回撤风险和CDaR的上界，
        从Chernoff不等式在给定置信水平（`edar_beta`）下获得。EDaR可以
        通过相对熵的概念表示。

    ulcer_index : float
        溃疡指数

    gini_mean_difference : float
        基尼均差（GMD）。它是两个实现之间预期绝对差的度量。GMD是
        非正态分布变异性优于方差的度量。它可以用于形成二阶随机占优的必要条件，
        而方差不能。

    mean_absolute_deviation_ratio : float
        平均绝对偏差比率。
        它是超额均值（均值 - 无风险利率）除以MaD。

    first_lower_partial_moment_ratio : float
        第一低阶矩比率。
        它是超额均值（均值 - 无风险利率）除以第一低阶矩。

    sharpe_ratio : float
        夏普比率。
        它是超额均值（均值 - 无风险利率）除以标准差。

    annualized_sharpe_ratio : float
        通过sharpe_ratio * sqrt(annualization_factor)年化的夏普比率。

    sortino_ratio : float
        索提诺比率。
        它是超额均值（均值 - 无风险利率）除以下半标准差。

    annualized_sortino_ratio : float
        通过sortino_ratio * sqrt(annualization_factor)年化的索提诺比率。

    value_at_risk_ratio : float
        VaR比率。
        它是超额均值（均值 - 无风险利率）除以风险价值（VaR）。

    cvar_ratio : float
        CVaR比率。
        它是超额均值（均值 - 无风险利率）除以条件风险价值（CVaR）。

    entropic_risk_measure_ratio : float
        熵风险度量比率。
        它是超额均值（均值 - 无风险利率）除以熵风险度量。

    evar_ratio : float
        EVaR比率。
        它是超额均值（均值 - 无风险利率）除以EVaR（熵风险价值）。

    worst_realization_ratio : float
        最差实现比率。
        它是超额均值（均值 - 无风险利率）除以最差实现（最差收益）。

    drawdown_at_risk_ratio : float
        回撤风险比率。
        它是超额均值（均值 - 无风险利率）除以回撤风险。

    cdar_ratio : float
        CDaR比率。
        它是超额均值（均值 - 无风险利率）除以CDaR（条件回撤风险）。

    calmar_ratio : float
        卡尔玛比率。
        它是超额均值（均值 - 无风险利率）除以最大回撤。

    average_drawdown_ratio : float
        平均回撤比率。
        它是超额均值（均值 - 无风险利率）除以平均回撤。

    edar_ratio : float
        EDaR比率。
        它是超额均值（均值 - 无风险利率）除以EDaR（熵回撤风险）。

    ulcer_index_ratio : float
        溃疡指数比率。
        它是超额均值（均值 - 无风险利率）除以溃疡指数。

    gini_mean_difference_ratio : float
        基尼均差比率。
        它是超额均值（均值 - 无风险利率）除以基尼均差。
    """
```

### BasePortfolio
```python
class BasePortfolio:
    """
    所有投资组合的基类。
    """
```

### FailedPortfolio
```python
class FailedPortfolio(BasePortfolio):
    """
    失败投资组合类。

    当优化失败时，此投资组合类用于表示失败的投资组合。

    参数
    ----------
    name : str, 可选
        投资组合的名称。

    optimization_error : str, 可选
        优化错误消息。

    **kwargs
        传递给`BasePortfolio`父类的其他参数。
    """
```

### MultiPeriodPortfolio
```python
class MultiPeriodPortfolio:
    """
    多期投资组合类。

    该类表示多个投资组合的序列，通常用于时间序列投资组合分析。

    参数
    ----------
    portfolios : list[Portfolio]
        投资组合列表。

    name : str, 可选
        投资组合的名称。

    **kwargs
        传递给`BasePortfolio`父类的其他参数。
    """
```

### Portfolio
```python
class Portfolio(BasePortfolio):
    """
    投资组合类。

    该类表示单个投资组合。

    参数
    ----------
    weights : array-like of shape (n_assets,)
        资产权重。

    name : str, 可选
        投资组合的名称。

    **kwargs
        传递给`BasePortfolio`父类的其他参数。
    """
```

**Section sources**
- [portfolio/__init__.py](file://src/skfolio/portfolio/__init__.py)
- [portfolio/_base.py](file://src/skfolio/portfolio/_base.py)

## 模型选择模块

### BaseCombinatorialCV
```python
class BaseCombinatorialCV(sks.BaseCrossValidator, ABC):
    """
    组合交叉验证生成器的基类。
    """
```

### CombinatorialPurgedCV
```python
class CombinatorialPurgedCV(BaseCombinatorialCV):
    """
    组合净化交叉验证生成器。

    该交叉验证器生成组合净化的训练/测试拆分。

    参数
    ----------
    n_splits : int, 默认=5
        拆分数量。

    purged_size : int, 默认=0
        从每个训练集末尾排除的观察数。

    **kwargs
        传递给`BaseCombinatorialCV`父类的其他参数。
    """
```

### MultipleRandomizedCV
```python
class MultipleRandomizedCV(sks.BaseCrossValidator):
    """
    多次随机交叉验证生成器。

    该交叉验证器生成多次随机的训练/测试拆分。

    参数
    ----------
    n_splits : int, 默认=5
        拆分数量。

    n_iterations : int, 默认=10
        迭代次数。

    **kwargs
        传递给`BaseCrossValidator`父类的其他参数。
    """
```

### WalkForward
```python
class WalkForward(sks.BaseCrossValidator):
    """
    向前走交叉验证器。

    提供训练/测试索引，使用向前走逻辑拆分时间序列数据样本。

    在每个拆分中，测试索引必须高于前一个；因此，交叉验证器中的混洗是不合适的。

    与`sklearn.model_selection.TimeSeriesSplit`相比，您通过指定训练和测试样本的数量来控制训练/测试折，
    而不是拆分数量，使其更适合投资组合交叉验证。

    如果您的数据是带有DatetimeIndex索引的DataFrame，您可以使用特定的日期时间频率和偏移量拆分数据。

    参数
    ----------
    test_size : int
        每个测试集的长度。
        如果`freq`为`None`（默认），它表示观察数。
        否则，它表示由`freq`定义的周期数。

    train_size : int | pandas.offsets.DateOffset | datetime.timedelta
        每个训练集的长度。
        如果`freq`为`None`（默认），它表示观察数。
        否则，对于整数，它表示由`freq`定义的周期数；
        对于pandas DateOffset或datetime timedelta，它表示应用于每个周期开始的日期偏移。

    freq : str | pandas.offsets.DateOffset, 可选
        如果提供，它必须是频率字符串或pandas DateOffset，且返回`X`必须是带有DatetimeIndex类型索引的DataFrame。
        有关pandas频率和偏移的列表，请参见`此处 <https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html#timeseries-offset-aliases>`_。
        默认（`None`）表示`test_size`和`train_size`表示观察数。

    freq_offset : pandas DateOffset | datetime timedelta, 可选
        仅在提供`freq`时使用。通过pandas DateOffset或datetime timedelta偏移`freq`。

    previous : bool, 默认=False
        仅在提供`freq`时使用。如果设置为`True`，且周期开始或结束不在`DatetimeIndex`中，
        则使用前一个观察；否则，使用下一个观察（默认）。

    expend_train : bool, 默认=False
        如果设置为`True`，每个后续训练集（第一个之后）将使用所有过去的观察。
        默认是`False`。

    reduce_test : bool, 默认=False
        如果设置为`True`，即使测试集不完整（即包含的观察数少于`test_size`），
        也会返回最后一个训练/测试拆分；否则，将忽略它。
        默认是`False`。

    purged_size : int, 默认=0
        从每个训练集末尾排除的观察数。
        默认值是`0`。

    警告
    ------
        **执行时机和前瞻控制**
        使用`purged_size=0`：
            - 训练在当前周期结束，测试立即开始。
            - 假设您可以在同一周期内观察、计算和执行。
            - 如果观察/计算到执行的延迟不可忽略
              （提交截止、流动性不足、期末结算或无盘中报价的市场），结果可能过于乐观。
        使用`purged_size=1`：
            - 在训练和测试之间排除一个观察。
            - 在当前周期开始做出的决策将从下一个周期开始影响性能。

        经验法则：
            - 仅当您确实可以在同一周期内以最小延迟执行时，才使用`purged_size=0`。
            - 当执行延迟时使用`purged_size >= 1`（每日定价资产、流动性不足的市场、收盘后结算的期末数据）。
    """
```

### cross_val_predict
```python
def cross_val_predict(
    estimator: skb.BaseEstimator | Pipeline,
    X: npt.ArrayLike,
    y: npt.ArrayLike = None,
    cv: sks.BaseCrossValidator
    | BaseCombinatorialCV
    | MultipleRandomizedCV
    | int
    | None = None,
    n_jobs: int | None = None,
    method: str = "predict",
    verbose: int = 0,
    params: dict | None = None,
    pre_dispatch: str = "2*n_jobs",
    column_indices: np.ndarray | None = None,
    portfolio_params: dict | None = None,
) -> MultiPeriodPortfolio | Population:
    """
    生成交叉验证的`投资组合`估计。

    数据根据`cv`参数拆分。
    优化估计器在训练集上拟合，并在相应的测试集上预测投资组合。

    对于单路径交叉验证，如`KFold`或`WalkForward`，输出是`MultiPeriodPortfolio`，
    其中每个`Portfolio`对应一个训练/测试拆分（`KFold`的`k`个投资组合）。

    对于多路径交叉验证，如`CombinatorialPurgedCV`或`MultipleRandomizedCV`，
    输出是多个`MultiPeriodPortfolio`对象的`Population`（每次测试产生多个路径的集合，而不是单个路径）。

    如果管道中的最终估计器（或估计器本身）声明`needs_previous_weights=True`，
    此函数会自动在顺序CV策略（例如`WalkForward`或`MultipleRandomizedCV`）中传播`previous_weights`。

    参数
    ----------
    estimator : BaseEstimator | Pipeline
        估计器或其最后一步是优化估计器的管道。

    X : array-like of shape (n_observations, n_assets)
        资产的收益。

    y : array-like of shape (n_observations, n_targets), 可选
        目标数据（可选）。
        例如，因子的收益。

    cv : int | cross-validation generator, 可选
        确定交叉验证拆分策略。
        `cv`的可能输入为：
            * None，使用默认的5折交叉验证，
            * int，指定`(Stratified)KFold`中的折数，
            * `CV splitter`，
            * 生成（训练，测试）拆分的可迭代对象。

    n_jobs : int, 可选
        并行运行`fit`的作业数。`None`表示1，除非在`joblib.parallel_backend`上下文中。-1表示使用所有处理器。

    method : str
        调用传递的估计器的传递方法名。

    verbose : int, 默认=0
        详细级别。

    params : dict, 可选
        传递给底层估计器的`fit`和CV拆分器的参数。

    pre_dispatch : int or str, 默认='2*n_jobs'
        控制并行执行期间调度的作业数。减少此数字有助于避免
        内存消耗爆炸，当调度的作业多于CPU可处理时。
        此参数可以是：
            * None，所有作业立即创建和启动。对于轻量级和快速运行的作业，
              使用此选项以避免按需启动作业的延迟
            * int，给出确切的总作业数
            * str，给出作为n_jobs函数的表达式，如'2*n_jobs'

    column_indices : ndarray, 可选
        要交叉验证的`X`列的索引。

    portfolio_params : dict, 可选
        传递给`MultiPeriodPortfolio`的额外投资组合参数。

    返回
    -------
    predictions : MultiPeriodPortfolio | Population
        这是调用`predict`的结果。
    """
```

### optimal_folds_number
```python
def optimal_folds_number(n_observations: int, test_size: int, purged_size: int = 0) -> int:
    """
    计算给定观察数、测试大小和净化大小的最优折数。

    参数
    ----------
    n_observations : int
        观察数。

    test_size : int
        测试大小。

    purged_size : int, 默认=0
        净化大小。

    返回
    -------
    n_folds : int
        最优折数。
    """
```

**Section sources**
- [model_selection/__init__.py](file://src/skfolio/model_selection/__init__.py)
- [model_selection/_validation.py](file://src/skfolio/model_selection/_validation.py)
- [model_selection/_walk_forward.py](file://src/skfolio/model_selection/_walk_forward.py)

## 分布模块

### BaseDistribution
```python
class BaseDistribution(skb.BaseEstimator, ABC):
    """基分布估计器。

    此抽象类作为skfolio中分布模型的基础。

    参数
    ----------
    random_state : int, RandomState实例或None, 默认=None
        用于确保可重现性的种子或随机状态。
    """

    @property
    @abstractmethod
    def n_params(self) -> int:
        """模型参数数量。"""
        pass

    @property
    @abstractmethod
    def fitted_repr(self) -> str:
        """拟合模型的字符串表示。"""
        pass

    @abstractmethod
    def fit(self, X: npt.ArrayLike, y=None) -> "BaseDistribution":
        """拟合单变量分布模型。

        参数
        ----------
        X : array-like of shape (n_observations, n_features)
            输入数据。

        y : None
            忽略。为与scikit-learn API兼容而提供。

        返回
        -------
        self : BaseDistribution
            返回实例本身。
        """
        pass

    @abstractmethod
    def score_samples(self, X: npt.ArrayLike) -> np.ndarray:
        """计算每个样本的对数似然（对数pdf）。

        参数
        ----------
        X : array-like of shape (n_observations, n_features)
            输入数据。

        返回
        -------
        density : ndarray of shape (n_observations,)
            X中每个观测的对数似然值。
        """
        pass

    def sample(self, n_samples: int = 1):
        """从拟合模型生成随机样本。

        参数
        ----------
        n_samples : int, 默认=1
            要生成的样本数。

        返回
        -------
        X : array-like of shape (n_samples, 1)
            样本列表。
        """
        pass

    def score(self, X: npt.ArrayLike, y=None):
        """计算模型下的总对数似然。

        参数
        ----------
        X : array-like of shape (n_observations, n_features)
            计算总对数似然的数据点数组。

        y : None
            忽略。为与scikit-learn API兼容而提供。

        返回
        -------
        logprob : float
            总对数似然（对数pdf值的总和）。
        """
        pass

    def aic(self, X: npt.ArrayLike) -> float:
        """计算给定数据X的模型AIC。

        AIC定义为：
        AIC = -2 * log L + 2 * k
        其中
        - log L是总对数似然
        - k是模型中的参数数量

        较低的AIC值表示模型拟合和复杂性之间的更好权衡。

        参数
        ----------
        X : array-like of shape (n_observations, n_features)
            用于计算AIC的输入数据。

        返回
        -------
        aic : float
            给定数据上拟合模型的AIC。
        """
        pass

    def bic(self, X: npt.ArrayLike) -> float:
        """计算给定数据X的模型BIC。

        BIC定义为：
        BIC = -2 * log L + k * ln(n)
        其中
        - log L是（最大化）总对数似然
        - k是模型中的参数数量
        - n是观测数

        较低的BIC值表示在施加更强的模型复杂性惩罚时更好的拟合。

        参数
        ----------
        X : array-like of shape (n_observations, n_features)
            用于计算BIC的输入数据。

        返回
        -------
        bic : float
           给定数据上拟合模型的BIC。
        """
        pass
```

### BaseBivariateCopula
```python
class BaseBivariateCopula(BaseDistribution, ABC):
    """二元Copula估计器的基类。"""
```

### BaseMultivariateDist
```python
class BaseMultivariateDist(BaseDistribution, ABC):
    """多元分布估计器的基类。"""
```

### BaseUnivariateDist
```python
class BaseUnivariateDist(BaseDistribution, ABC):
    """单变量分布估计器的基类。"""
```

### ClaytonCopula
```python
class ClaytonCopula(BaseBivariateCopula):
    """Clayton Copula估计器。"""
```

### CopulaRotation
```python
class CopulaRotation(AutoEnum):
    """Copula旋转的枚举。

    属性
    ----------
    NO_ROTATION : str
        无旋转。

    CLOCKWISE : str
        顺时针旋转。

    COUNTER_CLOCKWISE : str
        逆时针旋转。

    DOUBLE : str
        双重旋转。
    """
```

### DependenceMethod
```python
class DependenceMethod(AutoEnum):
    """依赖方法的枚举。

    属性
    ----------
    KENDALL_TAU : str
        Kendall Tau。

    MUTUAL_INFORMATION : str
        互信息。

    WASSERSTEIN_DISTANCE : str
        Wasserstein距离。
    """
```

### Gaussian
```python
class Gaussian(BaseUnivariateDist):
    r"""高斯分布估计。

    此估计器将单变量正态（高斯）分布拟合到输入数据。

    概率密度函数为：
    f(x) = exp(-x^2/2) / sqrt(2*pi)

    上述概率密度以“标准化”形式定义。要移动和/或缩放分布，请使用loc和scale参数。
    具体来说，`pdf(x, loc, scale)`等价于`pdf(y) / scale`，其中`y = (x - loc) / scale`。

    有关更多信息，请参阅`scipy文档 <https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.norm.html#scipy.stats.norm>`_

    参数
    ----------
    loc : float, 可选
        如果提供，位置参数（均值）固定为此值。
        否则，从数据中估计。

    scale : float, 可选
        如果提供，尺度参数（标准差）固定为此值。
        否则，从数据中估计。

    random_state : int, RandomState实例或None, 默认=None
        用于确保可重现性的种子或随机状态。

    属性
    ----------
    loc_ : float
        拟合的分布位置（均值）。

    scale_ : float
        拟合的分布尺度（标准差）。

    示例
    --------
    >>> from skfolio.datasets import load_sp500_index
    >>> from skfolio.preprocessing import prices_to_returns
    >>> from skfolio.distribution.univariate import Gaussian
    >>>
    >>> # 加载历史价格并将其转换为收益
    >>> prices = load_sp500_index()
    >>> X = prices_to_returns(prices)
    >>>
    >>> # 初始化高斯估计器。
    >>> model = Gaussian()
    >>>
    >>> # 将高斯模型拟合到数据。
    >>> model.fit(X)
    >>>
    >>> # 显示拟合参数。
    >>> print(model.fitted_repr)
    Gaussian(0.00035, 0.0115)
    >>>
    >>> # 计算对数似然、总对数似然、CDF、PPF、AIC和BIC
    >>> log_likelihood = model.score_samples(X)
    >>> score = model.score(X)
    >>> cdf = model.cdf(X)
    >>> ppf = model.ppf(X)
    >>> aic = model.aic(X)
    >>> bic = model.bic(X)
    >>>
    >>> # 从拟合的高斯分布生成5个新样本。
    >>> samples = model.sample(n_samples=5)
    >>>
    >>> # 绘制估计的概率密度函数（PDF）。
    >>> fig = model.plot_pdf()
    >>> fig.show()
    """
```

### GaussianCopula
```python
class GaussianCopula(BaseBivariateCopula):
    """高斯Copula估计器。"""
```

### GumbelCopula
```python
class GumbelCopula(BaseBivariateCopula):
    """Gumbel Copula估计器。"""
```

### IndependentCopula
```python
class IndependentCopula(BaseBivariateCopula):
    """独立Copula估计器。"""
```

### JoeCopula
```python
class JoeCopula(BaseBivariateCopula):
    """Joe Copula估计器。"""
```

### JohnsonSU
```python
class JohnsonSU(BaseUnivariateDist):
    """Johnson SU分布估计器。"""
```

### NormalInverseGaussian
```python
class NormalInverseGaussian(BaseUnivariateDist):
    """正态逆高斯分布估计器。"""
```

### SelectionCriterion
```python
class SelectionCriterion(AutoEnum):
    """选择标准的枚举。

    属性
    ----------
    AIC : str
        赤池信息准则（AIC）

    BIC : str
        贝叶斯信息准则（BIC）
    """
```

### StudentT
```python
class StudentT(BaseUnivariateDist):
    """学生t分布估计器。"""
```

### StudentTCopula
```python
class StudentTCopula(BaseBivariateCopula):
    """学生t Copula估计器。"""
```

### VineCopula
```python
class VineCopula(BaseMultivariateDist):
    """
    正则Vine Copula估计器。

    此模型首先为每个资产拟合最佳单变量分布，通过拟合的CDF将数据转换为均匀边缘。
    然后，它通过基于给定依赖度量的最小生成树算法为Vine中的每条边从候选列表中选择最佳二元Copula来构建正则Vine Copula。

    正则Vine捕获资产收益之间的复杂、厚尾依赖和尾部共同移动。

    它还支持条件采样，通过在指定条件下生成样本来实现压力测试和情景分析。

    此外，通过将某些资产标记为中心，此新实现能够捕获聚类或C型依赖结构，
    从而更细致地表示资产之间的层次关系，并改善条件采样和压力测试。

    参数
    ----------
    fit_marginals : bool, 默认=True
        是否在构建Vine之前为每个资产拟合边缘分布。如果为True，数据将使用拟合的CDF转换为均匀边缘。

    marginal_candidates : list[BaseUnivariateDist], 可选
        用于拟合边缘的候选单变量分布估计器。
        如果为None，默认为`[Gaussian(), StudentT(), JohnsonSU()]`。

    copula_candidates : list[BaseBivariateCopula], 可选
        候选二元Copula估计器。如果为None，默认为
        `[GaussianCopula(), StudentTCopula(), ClaytonCopula(), GumbelCopula(), JoeCopula()]`。

    max_depth : int or None, 默认=4
        最大Vine深度（截断级别）。必须大于1。
        `None`表示不应用截断。默认为4。

    log_transform : bool | dict[str, bool] | array-like of shape (n_assets, ), 默认=False
        如果为True，提供的简单收益将转换为对数收益，然后再拟合Vine Copula。
        即，每个收益R通过r = log(1+R)转换。采样后，生成的对数收益使用R = exp(r) - 1转换回简单收益。

    central_assets : array-like of asset names or asset positions, 可选
        在Vine构建期间应居中的资产。
        如果为None，无资产被强制居中。
        如果提供**整数**的数组，则其值必须是资产位置。
        如果提供**字符串**的数组，则其值必须是资产名称，且`fit`方法的输入`X`必须是
        资产名称在列中的DataFrame。标记为居中的资产将被强制占据Vine中的中心位置，
        导致C型或聚类结构。这对于条件采样是必需的，其中条件资产应为中央节点。

    dependence_method : DependenceMethod, 默认=DependenceMethod.KENDALL_TAU
        用于计算MST边权重的依赖度量。
        可能的值为：
        - KENDALL_TAU
        - MUTUAL_INFORMATION
        - WASSERSTEIN_DISTANCE

    selection_criterion : SelectionCriterion, 默认=SelectionCriterion.AIC
        用于单变量和Copula选择的标准。可能的值为：
        - SelectionCriterion.AIC : 赤池信息准则
        - SelectionCriterion.BIC : 贝叶斯信息准则

    independence_level : float, 默认=0.05
        在Copula拟合期间用于Kendall tau独立性检验的显著性水平。
        如果p值超过此阈值，则接受独立性零假设，并使用`IndependentCopula()`类对配对Copula建模。

    n_jobs : int, 可选
        并行运行`fit`的作业数。值`-1`表示使用所有处理器。
        默认（`None`）表示1，除非在`joblib.parallel_backend`上下文中。

    random_state : int, RandomState实例或None, 默认=None
        用于确保可重现性的种子或随机状态。

    属性
    ----------
    trees_ : list[Tree]
        构建的Vine树列表。

    marginal_distributions_ : list[BaseUnivariateDist]
        拟合的边缘分布列表（如果fit_marginals为True）。

    n_features_in_ : int
        `fit`期间看到的资产数量。

    feature_names_in_ : ndarray of shape (`n_features_in_`,)
        `fit`期间看到的资产名称。仅当`X`具有全为字符串的资产名称时定义。

    示例
    --------
    >>> from skfolio.datasets import load_factors_dataset
    >>> from skfolio.preprocessing import prices_to_returns
    >>> from skfolio.distribution import VineCopula
    >>>
    >>> # 加载历史价格并将其转换为收益
    >>> prices = load_factors_dataset()
    >>> X = prices_to_returns(prices)
    >>>
    >>> # 实例化VineCopula模型
    >>> vine = VineCopula()
    >>> # 拟合模型
    >>> vine.fit(X)
    >>> # 显示Vine树和拟合的Copula
    >>> vine.display_vine()
    >>> # 对数似然、AIC和BIC
    >>> vine.score(X)
    >>> vine.aic(X)
    >>> vine.bic(X)
    >>>
    >>> # 从拟合的Vine Copula生成10个样本
    >>> samples = vine.sample(n_samples=10)
    >>>
    >>> # 将QUAL、SIZE和MTUM设置为中心
    >>> vine = VineCopula(central_assets=["QUAL", "SIZE", "MTUM"])
    >>> vine.fit(X)
    >>> # 通过条件化QUAL和SIZE收益进行采样
    >>> samples = vine.sample(
    ...    n_samples=4,
    ...    conditioning={
    ...        "QUAL": [-0.1, -0.2, -0.3, -0.4],
    ...        "SIZE": -0.2,
    ...        "MTUM": (None, -0.3) # MTUM在-Inf和-30%之间采样
    ...    },
    ...)
    >>> # 绘制采样收益与历史X的散点矩阵
    >>> fig = vine.plot_scatter_matrix(X=X)
    >>> fig.show()
    >>>
    >>> # 绘制采样收益与历史X的边缘分布
    >>> fig = vine.plot_marginal_distributions(X=X)
    >>> fig.show()

    参考
    ----------
    .. [1] "Selecting and estimating regular vine copulae and application to financial
        returns" Dißmann, Brechmann, Czado, and Kurowicka (2013).

    .. [2] "Growing simplified vine copula trees: improving Dißmann's algorithm"
        Krausa and Czado (2017).

    .. [3] "Pair-copula constructions of multiple dependence" Aas, Czado, Frigessi,
        Bakken (2009).

    .. [4] "Pair-Copula Constructions for Financial Applications: A Review"
        Aas and Czado (2016).

    .. [5] "Conditional copula simulation for systemic risk stress testing"
        Brechmann, Hendrich, Czado (2013)
    """
```

### compute_pseudo_observations
```python
def compute_pseudo_observations(X: npt.ArrayLike) -> np.ndarray:
    """
    计算伪观测。

    参数
    ----------
    X : array-like of shape (n_observations, n_features)
        输入数据。

    返回
    -------
    pseudo_observations : ndarray of shape (n_observations, n_features)
        伪观测。
    """
```

### empirical_tail_concentration
```python
def empirical_tail_concentration(X: npt.ArrayLike, alpha: float) -> float:
    """
    计算经验尾部集中度。

    参数
    ----------
    X : array-like of shape (n_observations, n_features)
        输入数据。

    alpha : float
        尾部水平。

    返回
    -------
    tail_concentration : float
        经验尾部集中度。
    """
```

### plot_tail_concentration
```python
def plot_tail_concentration(X: npt.ArrayLike, alphas: np.ndarray) -> go.Figure:
    """
    绘制尾部集中度图。

    参数
    ----------
    X : array-like of shape (n_observations, n_features)
        输入数据。

    alphas : ndarray of shape (n_alphas,)
        尾部水平数组。

    返回
    -------
    fig : plotly.graph_objects.Figure
        尾部集中度图。
    """
```

### select_bivariate_copula
```python
def select_bivariate_copula(
    X: npt.ArrayLike,
    copula_candidates: list[BaseBivariateCopula],
    selection_criterion: SelectionCriterion = SelectionCriterion.AIC,
    independence_level: float = 0.05,
) -> BaseBivariateCopula:
    """
    从候选列表中选择最佳二元Copula。

    参数
    ----------
    X : array-like of shape (n_observations, 2)
        输入数据。

    copula_candidates : list[BaseBivariateCopula]
        候选二元Copula估计器。

    selection_criterion : SelectionCriterion, 默认=SelectionCriterion.AIC
        选择标准。

    independence_level : float, 默认=0.05
        独立性检验的显著性水平。

    返回
    -------
    copula : BaseBivariateCopula
        选择的二元Copula。
    """
```

### select_univariate_dist
```python
def select_univariate_dist(
    X: npt.ArrayLike,
    distribution_candidates: list[BaseUnivariateDist],
    selection_criterion: SelectionCriterion = SelectionCriterion.AIC,
) -> BaseUnivariateDist:
    """
    从候选列表中选择最佳单变量分布。

    参数
    ----------
    X : array-like of shape (n_observations, 1)
        输入数据。

    distribution_candidates : list[BaseUnivariateDist]
        候选单变量分布估计器。

    selection_criterion : SelectionCriterion, 默认=SelectionCriterion.AIC
        选择标准。

    返回
    -------
    distribution : BaseUnivariateDist
        选择的单变量分布。
    """
```

**Section sources**
- [distribution/__init__.py](file://src/skfolio/distribution/__init__.py)
- [distribution/_base.py](file://src/skfolio/distribution/_base.py)
- [distribution/univariate/_gaussian.py](file://src/skfolio/distribution/univariate/_gaussian.py)
- [distribution/multivariate/_vine_copula.py](file://src/skfolio/distribution/multivariate/_vine_copula.py)