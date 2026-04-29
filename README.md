# rstat-skill

一个 AI 驱动的 R 统计分析 Skill。在对话中直接运行真实 R 代码，完成描述性统计、回归分析、假设检验、时间序列预测等任务，并给出专业统计解读。

## 功能

- **描述统计 / EDA**: 摘要统计、分布分析、异常检测、相关性矩阵
- **回归分析**: 线性回归、逻辑回归、多重回归，含 VIF 共线性诊断、残差正态性检验
- **假设检验**: t 检验、ANOVA、卡方检验、Wilcoxon、Shapiro-Wilk 等，附效应量计算
- **时间序列**: STL 分解、auto.arima 建模、预测
- **数据可视化**: 直方图、箱线图、散点图、相关热力图、时序图（ggplot2）
- **Base R First**: 优先使用 base R，CRAN 不可用时也能运行核心分析

## 目录结构

```
rstat-skill/
├── SKILL.md                       # 主技能文件
├── reference/
│   ├── test-selection.md           # 统计检验选择决策树
│   └── visualization.md            # ggplot2 可视化模板
```

## 快速开始

1. 将 `rstat-skill` 目录放入你的 AI 工具 Skills 目录
2. 在工具中启用该 Skill
3. 上传 CSV 或粘贴数据，输入分析需求即可

## 依赖

- R (r-base)
- 可选 R 包: readr, readxl, dplyr, ggplot2, forecast, car, lmtest, psych
