[根目录](../../../CLAUDE.md) > [src](../../) > [skfolio](../) > **metrics**

# metrics - 评估指标模块

## 模块职责

metrics 模块提供了投资组合模型的评估和打分工具，遵循 scikit-learn 的 scorer 接口。这些指标用于交叉验证、模型选择、超参数调优和性能评估，帮助投资者选择最优的投资策略。

## 模块结构

```
metrics/
├── __init__.py              # 模块入口
└── _scorer.py              # Scorer 实现
```

## 入口与启动

```python
from skfolio.metrics import Scorer, make_scorer
```

## 对外接口

### Scorer

投资组合评分器：

```python
scorer = Scorer(
    scoring_func="sharpe_ratio",  # 评分函数
    greater_is_better=True,       # 是否越大越好
    needs_proba=False,            # 是否需要概率
)

# 在交叉验证中使用
from sklearn.model_selection import cross_val_score
scores = cross_val_score(model, X, cv=wf, scoring=scorer)
```

### make_scorer

创建自定义评分器：

```python
custom_scorer = make_scorer(
    score_func=lambda p: p.mean / p.max_drawdown,
    greater_is_better=True,
)
```

## 预定义评分指标

### 收益类指标
- **mean**: 平均收益
- **cumulative_return**: 累计收益

### 风险类指标
- **std**: 标准差（越小越好）
- **variance**: 方差（越小越好）
- **max_drawdown**: 最大回撤（越小越好）

### 风险调整收益指标
- **sharpe_ratio**: 夏普比率
- **sortino_ratio**: 索提诺比率
- **information_ratio**: 信息比率

## 关键依赖

### 核心依赖
- **numpy**: 数值计算
- **scikit-learn**: 评分器接口

## 测试与质量

### 测试文件
- `tests/test_metrics/test_scorer.py` - 评分器测试

## 变更记录

### 2025-12-09 06:15:32 UTC - 模块初始化
- 📚 **创建模块文档**：记录了 metrics 模块功能

---

## 使用建议

1. **模型选择**：使用评分器比较不同模型
2. **交叉验证**：在时间序列交叉验证中使用
3. **超参数调优**：网格搜索或随机搜索
4. **自定义指标**：创建符合特定需求的评分函数