[根目录](../../../CLAUDE.md) > [src](../../) > [skfolio](../) > **pre_selection**

# pre_selection - 资产预选择模块

## 模块职责

pre_selection 模块提供了在投资组合优化之前进行资产筛选的工具。预选择可以降低投资维度、减少计算复杂度、去除冗余或不理想的资产，从而提高优化效率和质量。

## 模块结构

```
pre_selection/
├── __init__.py              # 模块入口
├── _drop_correlated.py     # 剔除高相关资产
├── _drop_zero_variance.py  # 剔除零方差资产
├── _select_complete.py     # 选择数据完整的资产
├── _select_k_extremes.py   # 选择极端值资产
├── _select_non_dominated.py # 选择非支配资产
└── _select_non_expiring.py # 选择未到期资产
```

## 入口与启动

```python
from skfolio.pre_selection import (
    DropHighCorrelation,
    DropZeroVariance,
    SelectComplete,
    SelectKExtremes,
    SelectNonDominated,
    SelectNonExpiring,
)
```

## 对外接口

### DropHighCorrelation

剔除高相关性资产：

```python
selector = DropHighCorrelation(
    threshold=0.95,         # 相关性阈值
    method="pearson",       # 相关性方法
    keep_first=True,        # 保留第一个
)

X_selected = selector.fit_transform(X)
```

### SelectKExtremes

选择具有极端特征的资产：

```python
selector = SelectKExtremes(
    k=50,                   # 选择的资产数
    criterion="sharpe",     # 选择标准
    ascending=False,        # 降序选择
)

X_selected = selector.fit_transform(X)
```

## 关键依赖

### 核心依赖
- **numpy**: 数值计算
- **scikit-learn**: 转换器接口

## 测试与质量

### 测试文件
- `tests/test_pre_selection/` - 各种预选择器测试

## 变更记录

### 2025-12-09 06:15:32 UTC - 模块初始化
- 📚 **创建模块文档**：记录了 pre_selection 模块功能

---

## 使用建议

1. **降维**：大规模投资空间先进行预选择
2. **去冗余**：移除高度相关的资产
3. **质量筛选**：剔除数据质量差的资产
4. **计算效率**：减少优化问题的维度