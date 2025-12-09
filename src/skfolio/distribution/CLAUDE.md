[根目录](../../../CLAUDE.md) > [src](../../) > [skfolio](../) > **distribution**

# distribution - 概率分布建模模块

## 模块职责

distribution 模块提供了用于资产收益率分布建模的工具，包括单变量分布、Copula 函数和多变量分布。这些工具用于捕捉金融数据的非正态性、尾部风险和资产间的复杂依赖结构，为更精确的风险度量和投资组合优化提供基础。

## 模块结构

```
distribution/
├── __init__.py              # 模块入口
├── _base.py                # BaseDistribution 基类
├── univariate/             # 单变量分布
│   ├── __init__.py
│   ├── _base.py           # BaseUnivariateDist 基类
│   ├── _gaussian.py       # 正态分布
│   ├── _student_t.py      # 学生 t 分布
│   ├── _johnson_su.py     # Johnson SU 分布
│   ├── _normal_inverse_gaussian.py  # 正态逆高斯分布
│   └── _selection.py      # 分布选择工具
├── copula/                # Copula 函数
│   ├── __init__.py
│   ├── _base.py          # BaseBivariateCopula 基类
│   ├── _gaussian.py      # Gaussian Copula
│   ├── _student_t.py     # Student t Copula
│   ├── _clayton.py       # Clayton Copula
│   ├── _gumbel.py        # Gumbel Copula
│   ├── _joe.py           # Joe Copula
│   ├── _independent.py   # Independent Copula
│   ├── _utils.py         # Copula 工具函数
│   └── _selection.py     # Copula 选择工具
└── multivariate/          # 多变量分布
    ├── __init__.py
    ├── _base.py          # BaseMultivariateDist 基类
    ├── _vine_copula.py   # Vine Copula
    └── _utils.py         # 多变量工具
```

## 入口与启动

模块导出了所有分布和 Copula 相关类：

```python
from skfolio.distribution import (
    # 基类
    BaseDistribution,
    BaseUnivariateDist,
    BaseBivariateCopula,
    BaseMultivariateDist,

    # 单变量分布
    Gaussian,              # 正态分布
    StudentT,              # 学生 t 分布
    JohnsonSU,             # Johnson SU 分布
    NormalInverseGaussian, # 正态逆高斯分布

    # Copula 函数
    GaussianCopula,        # Gaussian Copula
    StudentTCopula,        # Student t Copula
    ClaytonCopula,         # Clayton Copula
    GumbelCopula,          # Gumbel Copula
    JoeCopula,             # Joe Copula
    IndependentCopula,     # Independent Copula

    # 多变量分布
    VineCopula,            # Vine Copula

    # 工具函数
    select_univariate_dist,  # 自动选择单变量分布
    select_bivariate_copula, # 自动选择二元 Copula
    compute_pseudo_observations,  # 计算伪观测值
    empirical_tail_concentration,  # 经验尾部集中度
)
```

## 对外接口

### BaseUnivariateDist 基类

单变量分布的基类：

```python
class BaseUnivariateDist(BaseEstimator):
    """单变量分布基类"""

    def fit(self, X, y=None):
        """拟合分布参数"""

    def cdf(self, X):
        """计算累积分布函数"""

    def ppf(self, X):
        """计算分位数函数（CDF 逆函数）"""

    def pdf(self, X):
        """计算概率密度函数"""

    def log_pdf(self, X):
        """计算对数概率密度"""

    def sample(self, n_samples=1):
        """生成随机样本"""
```

### 单变量分布

#### 1. 正态分布
```python
dist = Gaussian()
dist.fit(returns)
print(f"均值: {dist.mu_}")
print(f"标准差: {dist.sigma_}")
```

#### 2. 学生 t 分布
```python
dist = StudentT()
dist.fit(returns)
print(f"自由度: {dist.df_}")
```

#### 3. Johnson SU 分布
```python
dist = JohnsonSU()
dist.fit(returns)
print(f"形状参数: {dist.gamma_}, {dist.delta_}")
```

### BaseBivariateCopula 基类

二元 Copula 的基类：

```python
class BaseBivariateCopula(BaseEstimator):
    """二元 Copula 基类"""

    def fit(self, X):
        """拟合 Copula 参数"""

    def cdf(self, X):
        """计算 Copula CDF"""

    def pdf(self, X):
        """计算 Copula PDF"""

    def simulate(self, n_samples):
        """模拟 Copula 样本"""

    def tau(self):
        """计算 Kendall's tau"""
```

### Copula 函数

#### 1. Gaussian Copula
```python
copula = GaussianCopula()
copula.fit(returns)
print(f"相关系数: {copula.correlation_}")
```

#### 2. Student t Copula
```python
copula = StudentTCopula()
copula.fit(returns)
print(f"自由度: {copula.df_}")
```

#### 3. Clayton Copula（下尾依赖）
```python
copula = ClaytonCopula()
copula.fit(returns)
```

#### 4. Gumbel Copula（上尾依赖）
```python
copula = GumbelCopula()
copula.fit(returns)
```

### VineCopula

多变量 Copula 构建器：

```python
vine = VineCopula(
    copula_types=None,      # 自动选择 Copula 类型
    dependence_method="kendall",  # 依赖度量
    selection_criterion="aic",    # 选择准则
    vine_type="regular",     # Vine 类型
)
vine.fit(returns)
```

## 关键依赖与配置

### 核心依赖
- **numpy**: 数值计算
- **scipy**: 统计函数和优化
- **scikit-learn**: 基础估计器
- **plotly**: 可视化

### 优化配置
- **数值优化器**：L-BFGS-B、Nelder-Mead
- **起始值策略**：矩估计、启发式方法
- **收敛容差**：可配置的优化容差

### 并行处理
- 支持 Copula 选择过程的并行计算
- 分布拟合的批量处理

## 数据模型

### 输入格式
```python
X: np.ndarray  # shape (n_samples,) 或 (n_samples, n_assets)
```

### Copula 旋转
```python
class CopulaRotation(Enum):
    NONE = 0      # 无旋转
    ROTATION_90 = 90   # 90度旋转
    ROTATION_180 = 180  # 180度旋转
    ROTATION_270 = 270  # 270度旋转
```

### 依赖度量
- **Kendall's tau**：秩相关
- **Spearman's rho**：等级相关
- **Pearson**：线性相关

## 测试与质量

### 测试文件
- `tests/test_distribution/test_univariate/` - 单变量分布测试
- `tests/test_distribution/test_copula/` - Copula 函数测试
- `tests/test_distribution/test_multivariate/` - 多变量分布测试

### 测试覆盖
- ✅ 各种分布的参数估计准确性
- ✅ Copula 拟合质量
- ✅ 尾部依赖特性
- ✅ 数值稳定性
- ✅ 边界条件处理

### 性能优化
- 向量化实现
- 缓存常用计算
- 使用 JIT 编译（Numba）

## 常见问题 (FAQ)

### Q: 如何选择合适的单变量分布？
A:
```python
# 自动选择最佳分布
best_dist = select_univariate_dist(returns, criteria="aic")
print(f"最佳分布: {best_dist.__class__.__name__}")
```

### Q: 如何检测尾部依赖？
A:
```python
# 使用尾部集中度图
copula = StudentTCopula()
copula.fit(returns)
plot_tail_concentration(copula, returns)
```

### Q: 如何处理高维 Copula？
A:
```python
# 使用 Vine Copula 处理高维情况
vine = VineCopula(vine_type="regular")
vine.fit(returns)
```

### Q: 如何验证 Copula 拟合质量？
A:
```python
# 使用伪观测值的均匀性检验
from scipy.stats import kstest

pseudo_obs = compute_pseudo_observations(returns, dist)
stat, p_value = kstest(pseudo_obs, 'uniform')
```

## 相关文件清单

### 核心实现
- `univariate/_base.py` - 单变量分布基类
- `copula/_base.py` - Copula 基类
- `copula/_utils.py` - Copula 工具函数
- `multivariate/_vine_copula.py` - Vine Copula 实现

### 集成文件
- `prior/_synthetic_data.py` - 使用分布生成合成数据
- `distribution/_base.py` - 分布基类和选择工具

### 示例文件
- `examples/synthetic_data/plot_1_bivariate_copulas.py` - 二元 Copula 示例
- `examples/synthetic_data/plot_2_vine_copula.py` - Vine Copula 示例

## 变更记录 (Changelog)

### 2025-12-09 06:15:32 UTC - 模块初始化
- 📚 **创建模块文档**：完整记录了 distribution 模块的功能
- 🏗️ **架构分析**：梳理了单变量分布、Copula、多变量分布三个子模块
- 📊 **功能分类**：按分布类型和 Copula 特性分类说明
- ✅ **测试覆盖**：确认了完整的测试覆盖

### 最新功能
- 新增 Johnson SU 分布
- 改进的 Vine Copula 算法
- 自动 Copula 选择
- 尾部依赖分析工具

---

## 使用建议

1. **探索性分析**：先分析数据的分布特性
2. **分布选择**：使用信息准则选择最佳分布
3. **Copula 选择**：考虑尾部依赖特性选择 Copula
4. **验证拟合**：使用多种方法验证拟合质量
5. **维度管理**：高维情况考虑 Vine Copula

## 扩展开发

添加新分布：

```python
class NewDistribution(BaseUnivariateDist):
    def __init__(self, param1=1.0):
        super().__init__()
        self.param1 = param1

    def _fit(self, X):
        # 实现参数估计
        pass

    def _cdf(self, X):
        # 实现 CDF
        pass
```

## 性能提示

1. **批量处理**：使用向量化操作
2. **缓存结果**：缓存重复计算
3. **并行计算**：利用多核加速
4. **数值稳定**：注意数值精度问题
5. **内存管理**：大数据集使用分块处理