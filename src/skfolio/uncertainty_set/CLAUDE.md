[根目录](../../../CLAUDE.md) > [src](../../) > [skfolio](../) > **uncertainty_set**

# uncertainty_set - 不确定性集模块

## 模块职责

uncertainty_set 模块提供了鲁棒优化的不确定性集建模工具。不确定性集用于描述参数（如期望收益、协方差矩阵）的不确定性，通过最坏情况分析来构建更稳健的投资组合，特别适用于分布鲁棒优化和鲁棒投资组合优化。

## 模块结构

```
uncertainty_set/
├── __init__.py              # 模块入口
├── _base.py                # BaseUncertaintySet 基类
├── _empirical.py           # 经验不确定性集
└── _bootstrap.py           # 自举法不确定性集
```

## 入口与启动

```python
from skfolio.uncertainty_set import (
    BaseUncertaintySet,
    EmpiricalUncertaintySet,
    BootstrapUncertaintySet,
)
```

## 对外接口

### BaseUncertaintySet 基类

```python
class BaseUncertaintySet(BaseEstimator):
    """不确定性集基类"""

    def fit(self, X, y=None, **fit_params):
        """拟合不确定性集"""

    def get_uncertainty_bounds(self):
        """获取不确定性边界"""

    def sample(self, n_samples=1):
        """从不确定性集采样"""

    def worst_case_optimization(self, model, X):
        """最坏情况优化"""
```

### EmpiricalUncertaintySet

基于经验分布的不确定性集：

```python
empirical_set = EmpiricalUncertaintySet(
    confidence_level=0.95,    # 置信水平
    ellipsoid=True,           # 椭球不确定性集
    budget=None,              # 不确定性预算
)

empirical_set.fit(returns)
bounds = empirical_set.get_uncertainty_bounds()
```

### BootstrapUncertaintySet

基于自举法的不确定性集：

```python
bootstrap_set = BootstrapUncertaintySet(
    n_bootstrap=1000,         # 自举次数
    method="stationary",      # 自举方法
    block_size=None,          # 块大小
    alpha=0.05,              # 显著性水平
)

bootstrap_set.fit(returns)
samples = bootstrap_set.sample(n_samples=100)
```

## 关键依赖

### 核心依赖
- **numpy**: 数值计算
- **scipy**: 统计和优化
- **cvxpy**: 凸优化

## 测试与质量

### 测试文件
- `tests/test_uncertainty_set/test_empirical.py`
- `tests/test_uncertainty_set/test_bootstrap.py`

## 变更记录

### 2025-12-09 06:15:32 UTC - 模块初始化
- 📚 **创建模块文档**：记录了不确定性集模块功能
- ✅ **基础结构**：说明了不确定性集的基本概念和使用方法

---

## 使用建议

1. **鲁棒优化**：用于构建对参数不确定性稳健的组合
2. **置信水平**：根据风险偏好设置合适的置信水平
3. **计算效率**：注意不确定性集复杂度与计算成本的关系