[根目录](../../../CLAUDE.md) > [src](../../) > [skfolio](../) > **moments**

# moments - 统计矩估计模块

## 模块职责

moments 模块提供了用于估计投资组合关键统计矩（协方差矩阵和期望收益）的工具。这些估计器是投资组合优化的基础，提供了多种方法来处理数据噪声、提高估计稳定性并集成先验信息。

## 模块结构

```
moments/
├── __init__.py              # 模块入口
├── covariance/              # 协方差矩阵估计
│   ├── __init__.py
│   ├── _base.py            # BaseCovariance 基类
│   ├── _empirical_covariance.py    # 经验协方差
│   ├── _ledoit_wolf.py     # Ledoit-Wolf 收缩估计
│   ├── _oas.py             # Oracle 近似收缩
│   ├── _shrunk_covariance.py    # 收缩协方差
│   ├── _ew_covariance.py   # 指数加权协方差
│   ├── _gerber_covariance.py    # Gerber 统计协方差
│   ├── _denoise_covariance.py   # 去噪协方差
│   ├── _detone_covariance.py    # 去调协方差
│   ├── _graphical_lasso_cv.py   # 图套索交叉验证
│   └── _implied_covariance.py   # 隐含协方差
└── expected_returns/       # 期望收益估计
    ├── __init__.py
    ├── _base.py            # BaseMu 基类
    ├── _empirical_mu.py    # 经验期望收益
    ├── _ew_mu.py          # 指数加权期望收益
    ├── _shrunk_mu.py      # 收缩期望收益
    └── _equilibrium_mu.py # 均衡期望收益
```

## 入口与启动

模块导出了所有协方差和期望收益估计器：

```python
from skfolio.moments import (
    # 协方差估计器
    EmpiricalCovariance,      # 经验协方差
    LedoitWolf,              # Ledoit-Wolf 收缩
    OAS,                     # Oracle 近似收缩
    ShrunkCovariance,        # 通用收缩估计器
    EWCovariance,            # 指数加权协方差
    GerberCovariance,        # Gerber 统计协方差
    DenoiseCovariance,       # 去噪协方差
    DetoneCovariance,        # 去调协方差
    GraphicalLassoCV,        # 图套索交叉验证
    ImpliedCovariance,       # 隐含协方差

    # 期望收益估计器
    EmpiricalMu,             # 经验期望收益
    EWMu,                    # 指数加权期望收益
    ShrunkMu,                # 收缩期望收益
    EquilibriumMu,           # 均衡期望收益
)
```

## 对外接口

### BaseCovariance 基类

所有协方差估计器的基类，遵循 scikit-learn API：

```python
class BaseCovariance(BaseEstimator, TransformerMixin):
    """协方差估计基类"""

    def fit(self, X, y=None):
        """拟合协方差矩阵"""

    def transform(self, X):
        """转换数据"""

    def fit_transform(self, X, y=None):
        """拟合并转换"""

    def score(self, X, y=None):
        """计算似然得分"""
```

### 协方差估计器

#### 1. 经验协方差
```python
estimator = EmpiricalCovariance(
    assume_centered=False,  # 是否假设数据中心化
)
```

#### 2. Ledoit-Wolf 收缩
```python
estimator = LedoitWolf(
    assume_centered=False,  # 是否假设数据中心化
    block_size=1000,       # 块大小（内存优化）
)
```

#### 3. 指数加权协方差
```python
estimator = EWCovariance(
    halflife=252,          # 半衰期（日）
    decay=0.94,            # 衰减因子
    assume_centered=False,
)
```

#### 4. Gerber 统计协方差
```python
estimator = GerberCovariance(
    threshold=0.5,         # 阈值
    window_size=None,      # 窗口大小
    demean=True,           # 是否去均值
)
```

### BaseMu 基类

期望收益估计器的基类：

```python
class BaseMu(BaseEstimator, TransformerMixin):
    """期望收益估计基类"""

    def fit(self, X, y=None):
        """拟合期望收益"""

    def predict(self, X=None):
        """预测期望收益"""
```

### 期望收益估计器

#### 1. 经验期望收益
```python
estimator = EmpiricalMu(
    window_size=None,      # 滚动窗口大小
    keep_memory=False,     # 是否保留历史
)
```

#### 2. 指数加权期望收益
```python
estimator = EWMu(
    halflife=252,         # 半衰期
    decay=0.94,           # 衰减因子
    window_size=None,     # 窗口大小
)
```

#### 3. 收缩期望收益
```python
estimator = ShrunkMu(
    prior_estimator=EmpiricalMu(),  # 先验估计器
    shrinkage_factor=0.5,           # 收缩因子
    delta=0.5,                      # 平滑参数
)
```

## 关键依赖与配置

### 核心依赖
- **numpy**: 数值计算
- **scikit-learn**: 基础估计器
- **scipy**: 统计函数和优化

### 内存优化
- 支持块式计算处理大规模数据
- 可配置的窗口大小限制内存使用
- 稀疏矩阵支持

### 并行处理
- 支持多核并行计算
- joblib 并行后端

## 数据模型

### 输入格式
```python
X: np.ndarray  # shape (n_samples, n_features) - 资产收益矩阵
```

### 输出格式
- **协方差矩阵**: shape (n_features, n_features)
- **期望收益**: shape (n_features,)

### 估计方法分类

#### 传统方法
- 经验估计：直接计算样本统计量
- 指数加权：给予近期数据更高权重

#### 收缩方法
- Ledoit-Wolf：最优收缩因子
- Oracle 近似收缩 (OAS)
- 通用收缩框架

#### 结构化方法
- Gerber 统计：基于相关性阈值
- 图套索：稀疏逆协方差估计
- 隐含协方差：从期权价格提取

## 测试与质量

### 测试文件
- `tests/test_moment/test_covariance/test_covariance.py` - 协方差估计器测试
- `tests/test_moment/test_expected_returns/test_expected_returns.py` - 期望收益测试
- `tests/test_moment/test_covariance/test_implied_covariance.py` - 隐含协方差测试

### 测试覆盖
- ✅ 各种估计器的准确性
- ✅ 数值稳定性测试
- ✅ 边界条件处理
- ✅ 内存效率测试
- ✅ 并行计算测试

### 性能优化
- 向量化实现
- 缓存机制
- 块式计算

## 常见问题 (FAQ)

### Q: 如何选择协方差估计器？
A:
- **数据充足**：使用 EmpiricalCovariance
- **维度诅咒**：使用 LedoitWolf 或 OAS
- **时变数据**：使用 EWCovariance
- **噪声数据**：使用 DenoiseCovariance

### Q: 如何处理大规模协方差矩阵？
A:
```python
# 使用块大小限制内存使用
estimator = LedoitWolf(block_size=1000)

# 或使用稀疏方法
estimator = GraphicalLassoCV()
```

### Q: 如何集成先验信息？
A:
```python
# 使用收缩方法结合先验
estimator = ShrunkCovariance(
    shrinkage_factor=0.3,  # 0.3 的收缩强度
)
```

### Q: 期望收益估计的最佳实践？
A:
```python
# 使用指数加权处理非平稳性
mu_estimator = EWMu(halflife=126)  # 6个月半衰期

# 或使用收缩估计降低噪声
mu_estimator = ShrunkMu(
    prior_estimator=EquilibriumMu(),
    shrinkage_factor=0.5
)
```

## 相关文件清单

### 核心实现
- `covariance/_base.py` - 协方差基类
- `expected_returns/_base.py` - 期望收益基类
- `covariance/_ledoit_wolf.py` - Ledoit-Wolf 实现

### 集成文件
- `optimization/convex/_base.py` - 优化器使用矩估计
- `prior/_empirical.py` - 先验估计使用
- `model_selection/_validation.py` - 交叉验证使用

### 示例文件
- `examples/mean_risk/plot_13_factor_model.py` - 因子模型示例
- `examples/mean_risk/plot_12_black_and_litterman.py` - Black-Litterman 示例

## 变更记录 (Changelog)

### 2025-12-09 06:15:32 UTC - 模块初始化
- 📚 **创建模块文档**：完整记录了 moments 模块的功能
- 🏗️ **架构分析**：梳理了协方差和期望收益两个子模块
- 📊 **估计器分类**：按传统方法、收缩方法、结构化方法分类说明
- ✅ **测试覆盖**：确认了完整的测试覆盖

### 最新功能
- 新增 Gerber 统计协方差估计
- 改进的隐含协方差计算
- 更高效的块式计算
- 支持更多分布假设

---

## 使用建议

1. **数据质量**：检查并处理异常值和缺失值
2. **估计器选择**：根据数据特征选择合适的估计器
3. **参数调优**：使用交叉验证选择最优参数
4. **稳定性监控**：定期检查估计的稳定性
5. **组合使用**：可以链式使用多个估计器

## 扩展开发

添加新估计器：

```python
class NewEstimator(BaseCovariance):
    def __init__(self, param1=1.0, param2=True):
        super().__init__()
        self.param1 = param1
        self.param2 = param2

    def fit(self, X, y=None):
        # 实现拟合逻辑
        return self
```

## 性能提示

1. **维度控制**：对于高维数据使用收缩方法
2. **更新策略**：使用增量更新处理流数据
3. **并行计算**：利用多核加速计算
4. **缓存结果**：缓存重复使用的估计结果