# 每日作业报告（D1）

## 1. 本日问题

- 里程碑：day-01
- 学生或小组：杨涛，王艺博
- 使用者：准备历史展览、希望解释数据和模型局限的博物馆教育团队
- 真实输入：Kaggle Titanic Dataset 的 `train.csv`（891 行、12 列真实乘客记录）
- 需要的输出：多数类基线 + 固定随机种子随机森林候选在同一留出测试集上的指标，以及真实错误乘客记录
- 与使用者最相关的错误：假阴性——真实幸存（1）却被预测为未幸存（0）
- 本日产品边界：这是历史观察数据分析，不是现代救援分配工具，也不能证明某个特征导致生存

## 2. 真实数据或真实课程输入

- 所有者/发布者：Kaggle 用户 hesh97
- 标题：Titanic Dataset (train.csv)
- 原始 URL：https://www.kaggle.com/datasets/hesh97/titanicdataset-traincsv
- 许可标签或使用许可：按课程要求使用指定真实来源；未用生成数据替代
- 下载/取得日期：2026-08-17
- 预期文件与结构：`data/raw/train.csv`，891 行、12 列
- 检查命令：`python train.py --check-data`
- 实际检查结果：`REAL DATA CHECK PASSED`；rows: 891；columns: 12；survived_counts: {0: 549, 1: 342}；missing_age: 177；missing_cabin: 687；missing_embarked: 2
- 已知缺失、偏差或限制：`Age` 缺 177 条、`Cabin` 缺 687 条（未用作特征）、`Embarked` 缺 2 条；结论只覆盖这 891 条历史记录

## 3. 可复现运行

```powershell
# 当前目录：工作副本（不是课程源文件目录）
cd c:\Users\w\Desktop\ai-summer-camp-2026-main\student-work\day-01-titanic

# 安装（已用虚拟环境 c:\Users\w\Desktop\.venv）
c:\Users\w\Desktop\.venv\Scripts\python.exe -m pip install -r requirements.txt

# 数据检查（预期 REAL DATA CHECK PASSED）
c:\Users\w\Desktop\.venv\Scripts\python.exe train.py --check-data

# 主程序（生成 metrics.json 与 errors.csv）
c:\Users\w\Desktop\.venv\Scripts\python.exe train.py

# 测试
c:\Users\w\Desktop\.venv\Scripts\python.exe -m unittest discover -s tests -v
```

关键预期输出与实际输出文件位置：`metrics.json`（基线+候选指标）、`errors.csv`（57 条真实误判记录）、3 项单元测试全部 `OK`。

## 4. 基线与候选

### 简单基线

- 方法：`DummyClassifier(strategy="most_frequent")`，永远预测训练集中最常见的类别（0，未幸存）
- 为什么足够简单：不学习任何输入，只依赖训练集的众数，作为"最低要求"的比较对象
- 命令：随主程序 `python train.py` 一起运行
- 结果：accuracy 0.614；幸存者召回率 0；F1 0；混淆矩阵 [[137, 0], [86, 0]]——86 名真实幸存者全部漏掉

### 候选方法

- 学生完成的核心改动：`build_candidate()` 返回固定随机种子的 `RandomForestClassifier(n_estimators=100, random_state=SPLIT_SEED)` 流水线
- 保持不变的数据、划分、指标或参数：`FEATURES`、`SPLIT_SEED=42`、`test_size=0.25`、`stratify`、`preprocessor()` 的缺失值填充与编码策略
- 命令：`python train.py`
- 结果：accuracy 0.744；precision_survived 0.684；recall_survived 0.628；f1_survived 0.655；混淆矩阵 [[112, 25], [32, 54]]

| 项目 | 基线 | 候选 | 含义 |
| --- | ---: | ---: | --- |
| 主指标（准确率） | 0.614 | 0.744 | 全部预测里猜对的比例 |
| 幸存者召回率 | 0.000 | 0.628 | 真实幸存者中被识别的比例 |
| 幸存者 F1 | 0.000 | 0.655 | 精确率与召回率的平衡 |
| 假阴性（真实幸存→预测未幸存） | 86 | 32 | 与使用者最相关的错误 |

说明：数字均来自同一次固定划分（种子 42、223 人测试集），比较是公平且可复查的。

## 5. 一个真实失败案例

- 样本位置/编号：`errors.csv` 中 `source_row=347`（`PassengerId=348`）
- 真实结果：`Survived=1`（幸存）
- 系统输出：`predicted_survived=0`（未幸存）——假阴性
- 可以观察到什么：三等舱、女性、`Age` 缺失、`SibSp=1`、`Fare=16.1`、`Embarked=S`
- 说明的限制：模型对信息不完整、低舱位的乘客容易漏判幸存者
- 不能证明什么：不能证明"三等舱"或"年龄缺失"导致死亡
- 下一项最小检查：对 `Age` 缺失做标记或补充后重看召回率；或给假阴性更高权重（如 `class_weight`）再比较

## 6. 智能体与学生工作边界

- 智能体提出/生成/修改了什么：将 starter 复制到工作副本、说明并演示正确运行目录与命令、解释 `FileNotFoundError` 的成因（运行了源文件/目录错误）、协助整理本报告框架并核对数字
- 学生怎样核对文件、来源、输出、测试和 diff：学生已确认数据真实性（`--check-data` 输出 891 行与计数）、重新运行主程序与测试、对照 `metrics.json` / `errors.csv` 中的数字
- 学生修改或拒绝了什么建议：拒绝为 `FileNotFoundError` 修改代码（避免掩盖工作目录错误、破坏可复查性）
- 每名成员能独立解释的代码或证据：`make_split()` 的固定划分、`preprocessor()` 的缺失值填充、`build_candidate()` 的随机森林、`evaluate()` 的混淆矩阵/召回率/F1、`errors.csv` 的失败案例

## 7. 结论与限制

1. 我们只用了 Kaggle `train.csv` 的 891 条历史乘客记录；所有结论只覆盖这 891 人的这份历史数据，很难外推到其他海难或现代人群。
2. `errors.csv` 记录 57 条真实误判，例如 `source_row=347` 的 Davison 夫人（三等舱、年龄缺失）真实幸存却被预测未幸存（假阴性）；这证明模型对信息不完整、低舱位的乘客会漏判，但这一条只能说明该模型在此数据上的表现，不能证明"三等舱或年龄缺失导致死亡"。
3. 本程序只描述历史数据里的统计相关：乘客并没有被随机分配舱位、性别或年龄，模型发现的关联不是因果关系；因此它不是现代救援分配工具，绝不能用于决定真实海难中谁该优先获救，任何超出这 891 条历史记录的主张都应拒绝。

## 8. 提交复核

- [ ] README 从新环境可以开始运行
- [ ] 数据检查、测试和主程序重新运行
- [ ] 报告数字与保存输出一致
- [ ] `presentation.pptx` 在 3 分钟内讲完
- [ ] `submission.json` 路径正确
- [ ] 无密钥、大数据、私人信息、虚拟环境或缓存
- [ ] GitHub 网页复查并邮件发送 URL
