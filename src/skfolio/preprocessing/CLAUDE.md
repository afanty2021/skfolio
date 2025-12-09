[根目录](../../../CLAUDE.md) > [src](../../) > [skfolio](../) > **preprocessing**

# preprocessing - 数据预处理模块

## 模块职责

preprocessing 模块提供了金融数据预处理的工具，包括价格到收益的转换、缺失值处理、异常值检测、标准化等。良好的预处理是投资组合优化的基础，确保数据质量和一致性。

## 模块结构

```
preprocessing/
├── __init__.py              # 模块入口
└── _returns.py             # 收益率转换
```

## 入口与启动

```python
from skfolio.preprocessing import ReturnsTransformer
```

## 对外接口

### ReturnsTransformer

价格到收益率的转换：

```python
transformer = ReturnsTransformer(
    log_returns=False,       # 对数收益还是简单收益
    drop_first=True,         # 是否丢弃第一个NaN
    fill_method="ffill",     # 缺失值填充方法
)

returns = transformer.fit_transform(prices)
```

## 关键依赖

### 核心依赖
- **pandas**: 时间序列处理
- **numpy**: 数值计算

## 测试与质量

### 测试文件
- `tests/test_preprocessing/test_returns.py` - 收益转换测试

## 变更记录

### 2025-12-09 06:15:32 UTC - 模块初始化
- 📚 **创建模块文档**：记录了 preprocessing 模块功能

---

## 使用建议

1. **数据清洗**：预处理是投资组合分析的第一步
2. **一致性**：确保所有资产使用相同的计算方法
3. **缺失值**：合理处理缺失数据
4. **异常值**：检测和处理极端值