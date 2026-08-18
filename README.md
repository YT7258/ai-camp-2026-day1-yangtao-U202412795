# D1 · Titanic 历史数据预测（多数类基线 vs 随机森林）

> 海事博物馆教育团队教学展项：只用乘客记录中的信息（船票等级、性别、年龄、票价、同行家属数量），
> 模型能否区分历史记录中的“幸存 / 未幸存”？并诚实解释模型错在哪里。

- 里程碑：day-01
- 成员：杨涛、王艺博
- 数据：Kaggle Titanic Dataset `train.csv`（891 行、12 列真实乘客记录）

## 问题与使用者

- **使用者**：准备历史展览、希望解释数据和模型局限的博物馆教育团队；
- **任务**：二分类——对留出乘客记录预测 `Survived`（1 幸存 / 0 未幸存）；
- **输入特征**：`Pclass`、`Sex`、`Age`、`SibSp`、`Parch`、`Fare`、`Embarked`；
- **最关心错误**：假阴性（真实幸存却预测未幸存）；
- **边界**：历史观察数据分析，不是现代救援分配工具，也不能证明某个特征导致生存。

## 环境要求

- Python 3.11+
- 依赖见 `requirements.txt`：`pandas>=2.2,<4`、`scikit-learn>=1.5,<2`

```powershell
python -m pip install -r requirements.txt
```

## 数据准备

从指定页面下载真实数据，把 `train.csv` 放到 `data/raw/train.csv`（不改名、不生成替代数据）：

https://www.kaggle.com/datasets/hesh97/titanicdataset-traincsv

## 运行

```powershell
# 1. 校验真实数据（预期 REAL DATA CHECK PASSED；891 行；{0: 549, 1: 342}）
python train.py --check-data

# 2. 运行基线 + 候选模型，生成 metrics.json 与 errors.csv
python train.py

# 3. 运行单元测试
python -m unittest discover -s tests -v
```

## 预期输出

- `metrics.json`：数据集信息、基线指标、候选指标（accuracy / precision / recall / F1 / 混淆矩阵）；
- `errors.csv`：候选模型在留出测试集上的真实误判乘客记录（57 条）。

实测结果（固定划分：种子 42、25% 留出、223 人测试集）：

| 指标 | 基线（多数类） | 候选（随机森林） |
| --- | ---: | ---: |
| 准确率 | 0.614 | 0.744 |
| 幸存者召回率 | 0.000 | 0.628 |
| 幸存者 F1 | 0.000 | 0.655 |
| 假阴性 | 86 | 32 |

基线召回率为 0：86 名真实幸存者全部漏掉；候选在同一测试集上明显更有用。

## 关键实现

- `make_split()`：固定随机种子 42、25% 留出、分层抽样；
- `preprocessor()`：数值列用训练集中位数填充、类别列用最频繁值填充 + OneHot 编码；
- `build_baseline()`：`DummyClassifier(strategy="most_frequent")`（多数类基线）；
- `build_candidate()`：`RandomForestClassifier(n_estimators=100, random_state=42)` 流水线；
- `evaluate()`：准确率、精确率、召回率、F1、混淆矩阵。

## 限制

- 结论只覆盖这份 891 条历史记录；`Age` 缺失 177 条、`Cabin` 缺失 687 条（未用作特征）；
- 数字来自单次固定划分，不代表模型固有精度；
- 模型发现的是统计相关，不是因果关系；本程序不能用于真实救援决策。

## 交付物

- `train.py`、`tests/`：可复查的源代码与测试；
- `report.md`：书面报告（数据、方法、结果、失败案例、限制）；
- `presentation.pptx`：3 分钟答辩幻灯片（`make_presentation.py` 可重新生成）；
- `submission.json`：提交元数据；
- `README-course-notes.md`：课程 starter 学习说明备份。
