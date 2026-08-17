# 每日作业报告

## 1. 本日问题

- 里程碑：day-01
- 学生或小组：杨涛，王艺博
- 使用者：准备历史展览、希望解释数据和模型局限的博物馆教育团队。
- 真实输入：`data/raw/train.csv`，891 行、12 列真实乘客记录。
- 需要的输出：在同一留出测试集完成公平比较；能指出假阴性等重要错误；结论只解释历史数据中的预测表现，不作因果或现代救援主张。
- 与使用者最相关的错误：假阴性（真实生还却被判为死亡）
- 本日产品边界：这是历史观察数据分析，不是现代救援分配工具，也不能证明某个特征导致生存。

## 2. 真实数据或真实课程输入

- 所有者/发布者：Kaggle 用户 hesh97
- 标题：Titanic-Dataset (train.csv)
- 原始 URL：https://www.kaggle.com/datasets/hesh97/titanicdataset-traincsv
- 许可标签或使用许可：CC0: Public Domain
- 下载/取得日期：2026年8月17日
- 预期文件与结构：`data/raw/train.csv`，12 列，891 行
- 检查命令：`python train.py --check-data`
- 实际检查结果：
REAL DATA CHECK PASSED
rows: 891
columns: 12
survived_counts: {0: 549, 1: 342}
missing_age: 177
missing_cabin: 687
missing_embarked: 2
- 已知缺失、偏差或限制：Age 缺 177 个、Cabin 缺 687 个、Embarked 缺 2 个

Day 0 请把“真实输入”写成实际课程仓库、Day 1 文件和 `learning-plan.json`，并记录路径检查。

## 3. 可复现运行

```powershell
# 当前目录
cd d:\Ai-summer-camp-2026\ai-summer-camp-2026-main\student-work\day-01-titanic
# 安装
python -m pip install -r requirements.txt
# 数据检查
python train.py --check-data
# 主程序
python train.py
# 测试
```
python -m unittest discover -s tests -v
写出关键预期输出与实际输出文件位置。不要只写“运行成功”。
关键输出与文件位置：
`python train.py --check-data` → 打印 REAL DATA CHECK PASSED 及上方统计
`python train.py` → 生成 `metrics.json`（基线与候选指标）和 `errors.csv`（被分错的乘客记录），并在终端打印 JSON
`python -m unittest discover -s tests -v` → 3 个测试全部通过
## 4. 基线与候选

### 简单基线

- 方法：`DummyClassifier(strategy="most_frequent")`
- 为什么足够简单：不学习特征，只输出训练集中最常见的类（死亡）
- 命令：`python train.py`
- 结果：（metrics.json 中 baseline）：
accuracy: 0.6143
precision_survived: 0.0
recall_survived: 0.0
f1_survived: 0.0
confusion_matrix: [[137, 0], [86, 0]]

### 候选方法

- 学生完成的核心改动：在 `build_candidate` 中，在现有预处理流水线后加入固定随机种子的 `RandomForestClassifier`
- 保持不变的数据、划分、指标或参数：数据、`train_test_split`、指标、缺失值/类别特征预处理均未改动
- 命令：`python train.py`
- 结果：（metrics.json 中 candidate）：
accuracy: 0.7444
precision_survived: 0.6835
recall_survived: 0.6279
f1_survived: 0.6545
confusion_matrix: [[112, 25], [32, 54]]

| 项目 | 基线 | 候选 | 含义 |
| --- | ---: | ---: | --- |
| 主指标 accuracy| 0.6143 | 0.7444 | 候选明显高于基线 |
| 重要错误 假阴性 | 86人全部遗漏| 32 人漏 | 候选漏掉生还者大幅减少|

## 5. 一个真实失败案例

- 样本位置/编号：`errors.csv` 中 `source_row=549`，`PassengerId=550`
- 真实结果：survived = 1（生还）
- 系统输出：predicted_survived = 0（死亡）
- 可以观察到什么：候选模型把一位真实生还的儿童乘客判成死亡，属于假阴性
- 说明的限制：模型只使用 Pclass/Sex/Age 等有限特征，无法利用全部历史信息
- 不能证明什么：不能证明“年龄/舱位导致生存”，只是历史记录中的预测表现
- 下一项最小检查：统计全部 32 个假阴性的共同特征

## 6. 智能体与学生工作边界

- 智能体提出/生成/修改了什么：
- 学生怎样核对文件、来源、输出、测试和 diff：
- 学生修改或拒绝了什么建议：
- 每名成员能独立解释的代码或证据：

## 7. 结论与限制

用 5–8 句写证据支持的最小结论。至少写一个数据限制、一个方法限制和一个不能用于真实决策的边界。

## 8. 提交复核

- [ ] README 从新环境可以开始运行
- [ ] 数据检查、测试和主程序重新运行
- [ ] 报告数字与保存输出一致
- [ ] `presentation.pptx` 在 3 分钟内讲完
- [ ] `submission.json` 路径正确
- [ ] 无密钥、大数据、私人信息、虚拟环境或缓存
- [ ] GitHub 网页复查并邮件发送 URL

