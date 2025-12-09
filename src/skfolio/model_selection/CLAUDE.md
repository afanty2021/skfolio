[根目录](../../../CLAUDE.md) > [src](../../) > [skfolio](../) > **model_selection**

# model_selection - 模型选择与验证模块

## 模块职责

model_selection 模块提供了专门针对投资组合优化的模型选择和交叉验证工具。考虑到金融时间序列的特殊性（非独立性、结构断裂、前视偏差等），这些工具提供了适应金融数据的验证策略，帮助评估和选择最优的投资组合模型。

## 模块结构

```
model_selection/
├── __init__.py              # 模块入口
├── _base.py                # 基础工具函数
├── _validation.py          # 交叉验证工具
├── _walk_forward.py        # Walk Forward 分析
├── _combinatorial.py       # 组合交叉验证
└── _multiple_randomized_cv.py  # 多重随机化交叉验证
```

## 入口与启动

模块导出了所有模型选择工具：

```python
from skfolio.model_selection import (
    # 交叉验证策略
    WalkForward,                     # 滚动窗口交叉验证
    CombinatorialPurgedCV,          # 组合纯化交叉验证
    MultipleRandomizedCV,           # 多重随机化交叉验证

    # 工具函数
    cross_val_predict,              # 交叉验证预测
    optimal_folds_number,           # 最优折数计算
    BaseCombinatorialCV,            # 组合交叉验证基类
)
```

## 对外接口

### WalkForward

滚动窗口时间序列交叉验证，适合动态模型评估：

```python
wf = WalkForward(
    test_size=252,          # 测试集大小（1年交易日）
    train_size=504,         # 训练集大小（2年）
    step=126,              # 步长（6个月）
    refit=False,           # 是否重新拟合
    exploit_train_fold=True,  # 是否使用训练折
    purge_size=21,         # 纯化窗口（1个月）
    embargo_size=5,        # 禁运期（1周）
)

# 使用示例
models = [MeanRisk(), HierarchicalRiskParity()]
cv_results = wf.evaluate(X=returns, models=models)
```

### CombinatorialPurgedCV

组合纯化交叉验证，最大化数据使用效率：

```python
cpcv = CombinatorialPurgedCV(
    n_folds=5,             # 折数
    n_test_folds=2,        # 测试折数
    purge_size=21,         # 纯化窗口
    embargo_size=5,        # 禁运期
    shuffle=False,         # 是否打乱顺序
)

# 分割数据
splits = list(cpcv.split(returns))
```

### MultipleRandomizedCV

多重随机化交叉验证，提供稳健的性能估计：

```python
mrcv = MultipleRandomizedCV(
    n_trials=100,          # 试验次数
    train_ratio=0.7,       # 训练集比例
    test_ratio=0.3,        # 测试集比例
    purge_size=21,         # 纯化窗口
    embargo_size=5,        # 禁运期
    random_state=42,       # 随机种子
)

# 评估模型
results = mrcv.evaluate(X=returns, models=models)
```

### cross_val_predict

交叉验证预测工具：

```python
# 使用 Walk Forward 进行预测
predictions = cross_val_predict(
    estimator=MeanRisk(),
    X=returns,
    cv=WalkForward(train_size=504, test_size=252),
    n_jobs=-1,             # 并行计算
)

# 预测结果是多期投资组合
print(f"预测的投资组合数: {len(predictions.portfolios)}")
```

## 关键依赖与配置

### 核心依赖
- **numpy**: 数值计算
- **pandas**: 时间序列处理
- **scikit-learn**: 基础验证工具
- **joblib**: 并行计算

### 时间序列特性
- **时序保持**：确保训练数据在测试数据之前
- **纯化窗口**：避免训练和测试间的信息泄露
- **禁运期**：考虑交易延迟

### 并行配置
```python
# 使用多核加速
WalkForward(..., n_jobs=-1)  # 使用所有核心
CombinatorialPurgedCV(..., n_jobs=4)  # 使用4个核心
```

## 数据模型

### 时间序列索引
```python
X: pd.DataFrame  # 索引为时间戳，shape (n_periods, n_assets)
```

### 分割格式
- **训练集**：连续的时间段
- **验证集**：训练集后的时间段
- **测试集**：验证集后的时间段

### 纯化和禁运
```
训练集 | 纯化窗口 | 验证集 | 禁运期 | 测试集
```

## 验证策略对比

| 策略 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **Walk Forward** | 模拟实盘交易 | 计算密集 | 动态策略评估 |
| **CombinatorialPurgedCV** | 数据利用率高 | 实现复杂 | 模型选择 |
| **MultipleRandomizedCV** | 统计稳健 | 计算成本高 | 稳健性验证 |

## 测试与质量

### 测试文件
- `tests/test_model_selection/test_walk_forward.py` - Walk Forward 测试
- `tests/test_model_selection/test_combinatorial.py` - 组合交叉验证测试
- `tests/test_model_selection/test_multiple_randomized_cv.py` - 随机化交叉验证测试
- `tests/test_model_selection/test_validation.py` - 验证工具测试

### 测试覆盖
- ✅ 各种分割策略的正确性
- ✅ 时序依赖处理
- ✅ 纯化和禁运逻辑
- ✅ 并行计算
- ✅ 边界条件

### 性能优化
- 向量化分割
- 内存优化
- 并行计算支持

## 常见问题 (FAQ)

### Q: 如何选择交叉验证策略？
A:
- **动态策略**：使用 Walk Forward
- **模型选择**：使用 CombinatorialPurgedCV
- **稳健性验证**：使用 MultipleRandomizedCV

### Q: 如何设置纯化和禁运参数？
A:
```python
# 短期策略（日频）
purge_size=5,      # 1周
embargo_size=1,     # 1天

# 长期策略（月频）
purge_size=21,     # 1个月
embargo_size=5,     # 1周
```

### Q: 如何处理不平衡数据？
A:
```python
# 使用分层采样
wf = WalkForward(
    train_size=504,
    test_size=252,
    stratify=market_regimes,  # 根据市场状态分层
)
```

### Q: 如何加速交叉验证？
A:
```python
# 1. 使用并行计算
wf = WalkForward(n_jobs=-1)

# 2. 减少试验次数
mrcv = MultipleRandomizedCV(n_trials=50)

# 3. 使用预计算
precomputed_cov = CovarianceEstimator().fit_transform(returns)
```

## 相关文件清单

### 核心实现
- `_walk_forward.py` - Walk Forward 实现
- `_combinatorial.py` - 组合交叉验证实现
- `_multiple_randomized_cv.py` - 随机化交叉验证实现
- `_validation.py` - 验证工具函数

### 集成文件
- `optimization/_base.py` - 优化器使用交叉验证
- `population/_population.py` - Population 使用交叉验证
- `examples/model_selection/` - 交叉验证示例

### 示例文件
- `examples/model_selection/plot_1_multiple_randomized_cv.py` - 多重随机化交叉验证示例

## 变更记录 (Changelog)

### 2025-12-09 06:15:32 UTC - 模块初始化
- 📚 **创建模块文档**：完整记录了 model_selection 模块的功能
- 🏗️ **策略分析**：详细说明了 Walk Forward、组合交叉验证、随机化交叉验证
- 📊 **参数配置**：记录了纯化窗口、禁运期等关键参数的设置方法
- ✅ **测试覆盖**：确认了完整的测试覆盖

### 最新功能
- 改进的 Walk Forward 实现
- 支持并行计算
- 更灵活的分割策略
- 性能优化

---

## 使用建议

1. **策略匹配**：根据模型特性选择合适的验证策略
2. **参数调优**：根据交易频率调整纯化和禁运参数
3. **避免过拟合**：使用多重验证确保结果稳健
4. **计算效率**：合理使用并行计算
5. **结果分析**：检查验证结果的统计显著性

## 扩展开发

创建自定义验证策略：

```python
class CustomCV(BaseCrossValidator):
    def __init__(self, param1=1.0):
        super().__init__()
        self.param1 = param1

    def split(self, X, y=None, groups=None):
        # 实现自定义分割逻辑
        pass

    def get_n_splits(self, X=None, y=None, groups=None):
        # 返回分割数
        pass
```

## 最佳实践

1. **时间序列**：始终保持时序顺序
2. **信息泄露**：使用纯化和禁运避免
3. **样本外**：确保真正的样本外测试
4. **多次验证**：使用多次运行评估稳定性
5. **交易成本**：在验证中考虑交易成本