# Parametric Faithfulness File Guide

仓库根目录：`/home/kousei/faithfulness/parametric-faithfulness`

本文档按文件说明仓库用途、怎么执行、以及与论文复现的关系。

## 1. 核心脚本

| 文件 | 作用 | 怎么执行/调用 | 备注 |
|---|---|---|---|
| `README.md` | 仓库入口说明，给出论文标题、图示和 README 第一条示例命令。 | 直接阅读。 | README 没有完整环境步骤，环境与复现步骤已补到 `/home/kousei/faithfulness/quickstart.md`。 |
| `LICENSE` | 项目许可证。 | 不执行。 | 分发或复用代码前查看。 |
| `unlearn.py` | 主实验入口：加载模型、加载或生成 CoT、逐步 unlearn、评估 efficacy/specificity/faithfulness、写结果文件。 | `python unlearn.py --model_name ... --dataset ... --strategy sentencize --stepwise --pos --ff2 --method npo_KL` | 当前还支持 `--cot_limit`、`--verify_size`、`--max_unlearn` 方便 smoke test。 |
| `evaluate.py` | 生成 CoT、算答案概率、固定 CoT 后继续答题、生成新 CoT，并为主实验提供评估函数。 | 通常被 `unlearn.py` 调用，也可单独 import。 | 已补分句 fallback，避免缺少 NLTK 资源时报错。 |
| `data.py` | 把问题/CoT 编码成 forget-retain 数据，负责 CoT 缓存读写。 | 通常由 `unlearn.py` 间接调用。 | 当前缓存名会区分 `--cot_limit`，避免小规模测试覆盖完整实验缓存。 |
| `dataload.py` | 数据集适配层，定义 `DataHandler` 及 SQA、ARC、OpenBook、Sports、MMLU 等数据集 prompt 格式。 | 作为库文件 import。 | 论文主实验主要用 `sqa`、`arc-challenge`、`openbook`、`sports`。 |
| `models.py` | 统一封装 Hugging Face 模型和 tokenizer 加载。 | `from models import load_model_and_tokenizer` | 默认 `device_map="auto"`，`torch_dtype=bfloat16`。 |
| `segment.py` | CoT 分句、词性对齐、内容词过滤。 | 作为库文件 import。 | `--pos` 路径会依赖这里的内容词过滤逻辑。 |
| `const.py` | 数据集名、模型短名、最佳学习率、答案字母等常量。 | 不直接执行。 | 多个脚本都会 import。 |
| `util.py` | 随机种子、结果读取、结果过滤、按学习率加载结果等工具函数。 | 作为库文件 import。 | 分析 notebook 常用。 |
| `stats.py` | 从逐实例日志中汇总 faithfulness / efficacy / specificity。 | 作为库文件 import。 | 论文统计数字依赖这层。 |
| `plotting.py` | 绘图函数：散点图、概率柱状图等。 | 通常在 notebook 中调用。 | 依赖 `matplotlib`。 |
| `vis_samples.py` | 对单个样本进行 CoT salience 可视化。 | 在 notebook 或交互式 Python 中调用。 | 用于 heatmap 展示。 |
| `run_scripts.py` | 批量生成 sbatch 命令。 | `python run_scripts.py` | 面向集群，不是本地主入口。 |
| `mmlu.py` | 通过 `lm-eval` 评测 MMLU。 | `python mmlu.py` | 需要额外安装 `lm-eval`。 |
| `mistakes_const.py` | “往 CoT 加错误”实验使用的 prompt 常量。 | 不直接执行。 | 主要被 `mistakes_repro.py` 或 notebook import。 |
| `mistakes_repro.py` | 把加入错误后的 CoT step 重新送入模型，检查答案是否 flip。 | `python mistakes_repro.py --model_name ... --dataset ...` | 已改成“有 HF token 则登录，否则跳过”。 |

## 2. Notebook 文件

| 文件 | 作用 | 怎么执行 | 备注 |
|---|---|---|---|
| `Ablations.ipynb` | 论文 ablation、主结果表格与图分析。 | 用 Jupyter 打开逐 cell 运行。 | 依赖完整结果文件。 |
| `Adding mistakes repro.ipynb` | 用 GPT-4o-mini 复现“往 CoT 里加入错误”。 | Jupyter 运行。 | 需要 OpenAI API key。 |
| `Annotation analysis.ipynb` | 人工标注研究后续分析。 | Jupyter 运行。 | 消费 `annotation_results/` 和 `annotation_data/`。 |
| `CoT LLM as judge.ipynb` | 让 GPT-4o 判断 unlearning 前后两条 CoT 是否支持同一答案。 | Jupyter 运行。 | 需要 OpenAI API key。 |
| `Generate_CoT_heatmaps.ipynb` | 生成论文 heatmap 与示例可视化。 | Jupyter 运行。 | 会调用 `vis_samples.py`。 |
| `Generate_annotation_data.ipynb` | 从结果中抽样并构造人工标注集。 | Jupyter 运行。 | 输出保存在 `annotation_data/`。 |

## 3. 图像与论文产物

| 文件 | 作用 | 怎么查看 |
|---|---|---|
| `figures/fig1.png` | 论文主图旧版位图。 | 直接打开。 |
| `figures/fig1_v2.png` | README 中展示的主图新版。 | 直接打开。 |
| `figures/flip_histograms.pdf` | 答案 flip 相关直方图。 | 直接打开。 |
| `figures/correlation_overall.pdf` | 整体相关性图。 | 直接打开。 |
| `figures/lr_ablation.pdf` | 学习率 ablation 图。 | 直接打开。 |
| `figures/model_correlates.pdf` | 模型维度相关性图。 | 直接打开。 |
| `figures/dataset_correlates.pdf` | 数据集维度相关性图。 | 直接打开。 |
| `figures/heatmap_sample.pdf` | 单样本 heatmap 示例图。 | 直接打开。 |

## 4. 人工标注数据

这些文件都不需要“执行”，通常用 `pandas.read_csv(...)` 查看。

| 文件 | 作用 | 备注 |
|---|---|---|
| `annotation_data/annotation_sample_5_5_15_shuffled.csv` | 打乱后的标注样本总表。 | 人工标注抽样产物。 |
| `annotation_data/explain_sample.csv` | 解释性示例样本。 | 用于说明标注格式。 |
| `annotation_data/arc-challenge_LLaMA-3-3B_high_bin.csv` | ARC-Challenge / LLaMA-3-3B / high bin 样本。 | 文件名编码了数据集、模型、分 bin。 |
| `annotation_data/arc-challenge_LLaMA-3-3B_med_bin.csv` | ARC-Challenge / LLaMA-3-3B / medium bin 样本。 | 同上。 |
| `annotation_data/arc-challenge_LLaMA-3-3B_neg_bin.csv` | ARC-Challenge / LLaMA-3-3B / negative bin 样本。 | 同上。 |
| `annotation_data/arc-challenge_LLaMA-3-8B_high_bin.csv` | ARC-Challenge / LLaMA-3-8B / high bin 样本。 | 同上。 |
| `annotation_data/arc-challenge_LLaMA-3-8B_med_bin.csv` | ARC-Challenge / LLaMA-3-8B / medium bin 样本。 | 同上。 |
| `annotation_data/arc-challenge_LLaMA-3-8B_neg_bin.csv` | ARC-Challenge / LLaMA-3-8B / negative bin 样本。 | 同上。 |
| `annotation_data/sqa_LLaMA-3-3B_high_bin.csv` | StrategyQA / LLaMA-3-3B / high bin 样本。 | 同上。 |
| `annotation_data/sqa_LLaMA-3-3B_med_bin.csv` | StrategyQA / LLaMA-3-3B / medium bin 样本。 | 同上。 |
| `annotation_data/sqa_LLaMA-3-3B_neg_bin.csv` | StrategyQA / LLaMA-3-3B / negative bin 样本。 | 同上。 |
| `annotation_data/sqa_LLaMA-3-8B_high_bin.csv` | StrategyQA / LLaMA-3-8B / high bin 样本。 | 同上。 |
| `annotation_data/sqa_LLaMA-3-8B_med_bin.csv` | StrategyQA / LLaMA-3-8B / medium bin 样本。 | 同上。 |
| `annotation_data/sqa_LLaMA-3-8B_neg_bin.csv` | StrategyQA / LLaMA-3-8B / negative bin 样本。 | 同上。 |
| `annotation_results/reasoning-chain-study.csv` | 人工标注研究最终结果表。 | 后续分析 notebook 主要消费这份文件。 |

## 5. LLM-as-Judge 结果文件

这些都是 JSONL 结果文件，不直接执行，通常用 `pandas.read_json(path, lines=True)` 查看。

| 文件 | 作用 |
|---|---|
| `LM_judge_cot/LLaMA-3-3B_arc-challenge_NPO_KL_3e-05_judgements.jsonl` | LLaMA-3-3B 在 ARC-Challenge 上的 judge 结果。 |
| `LM_judge_cot/LLaMA-3-3B_openbook_NPO_KL_1e-05_judgements.jsonl` | LLaMA-3-3B 在 OpenBookQA 上的 judge 结果。 |
| `LM_judge_cot/LLaMA-3-3B_sports_NPO_KL_3e-05_judgements.jsonl` | LLaMA-3-3B 在 Sports 上的 judge 结果。 |
| `LM_judge_cot/LLaMA-3-3B_sqa_NPO_KL_3e-05_judgements.jsonl` | LLaMA-3-3B 在 StrategyQA 上的 judge 结果。 |
| `LM_judge_cot/LLaMA-3_arc-challenge_NPO_KL_1e-05_judgements.jsonl` | LLaMA-3-8B 在 ARC-Challenge 上的 judge 结果。 |
| `LM_judge_cot/LLaMA-3_openbook_NPO_KL_1e-05_judgements.jsonl` | LLaMA-3-8B 在 OpenBookQA 上的 judge 结果。 |
| `LM_judge_cot/LLaMA-3_sports_NPO_KL_5e-06_judgements.jsonl` | LLaMA-3-8B 在 Sports 上的 judge 结果。 |
| `LM_judge_cot/LLaMA-3_sqa_NPO_KL_5e-06_judgements.jsonl` | LLaMA-3-8B 在 StrategyQA 上的 judge 结果。 |
| `LM_judge_cot/Mistral-2_arc-challenge_NPO_KL_5e-06_judgements.jsonl` | Mistral-2 在 ARC-Challenge 上的 judge 结果。 |
| `LM_judge_cot/Mistral-2_openbook_NPO_KL_5e-06_judgements.jsonl` | Mistral-2 在 OpenBookQA 上的 judge 结果。 |
| `LM_judge_cot/Mistral-2_sports_NPO_KL_3e-06_judgements.jsonl` | Mistral-2 在 Sports 上的 judge 结果。 |
| `LM_judge_cot/Mistral-2_sqa_NPO_KL_3e-06_judgements.jsonl` | Mistral-2 在 StrategyQA 上的 judge 结果。 |
| `LM_judge_cot/Phi-3_arc-challenge_NPO_KL_0.0001_judgements.jsonl` | Phi-3 在 ARC-Challenge 上的 judge 结果。 |
| `LM_judge_cot/Phi-3_openbook_NPO_KL_0.0001_judgements.jsonl` | Phi-3 在 OpenBookQA 上的 judge 结果。 |
| `LM_judge_cot/Phi-3_sports_NPO_KL_5e-05_judgements.jsonl` | Phi-3 在 Sports 上的 judge 结果。 |
| `LM_judge_cot/Phi-3_sqa_NPO_KL_5e-05_judgements.jsonl` | Phi-3 在 StrategyQA 上的 judge 结果。 |

## 6. 最小 add-mistake 结果

这些文件都是历史产物，不需要执行，通常按文本或 jsonl 日志查看。

| 文件 | 作用 |
|---|---|
| `minimal_mistake_results/arc-challenge/LLaMA-3-3B/npo_KL_lr=3e-05_rs=1001_pos=True_ff2=True.out` | ARC-Challenge / LLaMA-3-3B 的最小 add-mistake 结果。 |
| `minimal_mistake_results/arc-challenge/LLaMA-3/npo_KL_lr=1e-05_rs=1001_pos=True_ff2=True.out` | ARC-Challenge / LLaMA-3-8B 的最小 add-mistake 结果。 |
| `minimal_mistake_results/arc-challenge/Mistral-2/npo_KL_lr=5e-06_rs=1001_pos=True_ff2=True.out` | ARC-Challenge / Mistral-2 的最小 add-mistake 结果。 |
| `minimal_mistake_results/arc-challenge/Phi-3/npo_KL_lr=0.0001_rs=1001_pos=True_ff2=True.out` | ARC-Challenge / Phi-3 的最小 add-mistake 结果。 |
| `minimal_mistake_results/openbook/LLaMA-3-3B/npo_KL_lr=3e-05_rs=1001_pos=True_ff2=True.out` | OpenBookQA / LLaMA-3-3B 的最小 add-mistake 结果。 |
| `minimal_mistake_results/openbook/LLaMA-3/npo_KL_lr=1e-05_rs=1001_pos=True_ff2=True.out` | OpenBookQA / LLaMA-3-8B 的最小 add-mistake 结果。 |
| `minimal_mistake_results/openbook/Mistral-2/npo_KL_lr=5e-06_rs=1001_pos=True_ff2=True.out` | OpenBookQA / Mistral-2 的最小 add-mistake 结果。 |
| `minimal_mistake_results/openbook/Phi-3/npo_KL_lr=0.0001_rs=1001_pos=True_ff2=True.out` | OpenBookQA / Phi-3 的最小 add-mistake 结果。 |
| `minimal_mistake_results/sports/LLaMA-3-3B/npo_KL_lr=3e-05_rs=1001_pos=True_ff2=True.out` | Sports / LLaMA-3-3B 的最小 add-mistake 结果。 |
| `minimal_mistake_results/sports/LLaMA-3/npo_KL_lr=5e-06_rs=1001_pos=True_ff2=True.out` | Sports / LLaMA-3-8B 的最小 add-mistake 结果。 |
| `minimal_mistake_results/sports/Mistral-2/npo_KL_lr=3e-06_rs=1001_pos=True_ff2=True.out` | Sports / Mistral-2 的最小 add-mistake 结果。 |
| `minimal_mistake_results/sports/Phi-3/npo_KL_lr=5e-05_rs=1001_pos=True_ff2=True.out` | Sports / Phi-3 的最小 add-mistake 结果。 |
| `minimal_mistake_results/sqa/LLaMA-3-3B/npo_KL_lr=3e-05_rs=1001_pos=True_ff2=True.out` | StrategyQA / LLaMA-3-3B 的最小 add-mistake 结果。 |
| `minimal_mistake_results/sqa/LLaMA-3/npo_KL_lr=1e-05_rs=1001_pos=True_ff2=True.out` | StrategyQA / LLaMA-3-8B 的最小 add-mistake 结果。 |
| `minimal_mistake_results/sqa/Mistral-2/npo_KL_lr=5e-06_rs=1001_pos=True_ff2=True.out` | StrategyQA / Mistral-2 的最小 add-mistake 结果。 |
| `minimal_mistake_results/sqa/Phi-3/npo_KL_lr=5e-05_rs=1001_pos=True_ff2=True.out` | StrategyQA / Phi-3 的最小 add-mistake 结果。 |

## 7. 实际复现时最常用的几个入口

### 主实验

```bash
cd /home/kousei/faithfulness/parametric-faithfulness
conda run -n faithfulness python unlearn.py \
  --model_name meta-llama/Llama-3.2-3B-Instruct \
  --strategy sentencize \
  --stepwise \
  --dataset sqa \
  --lr 3e-05 \
  --pos \
  --ff2 \
  --method npo_KL
```

### 小规模 smoke test

```bash
cd /home/kousei/faithfulness/parametric-faithfulness
conda run -n faithfulness python unlearn.py \
  --model_name microsoft/Phi-3-mini-4k-instruct \
  --strategy sentencize \
  --stepwise \
  --dataset sqa \
  --lr 3e-05 \
  --pos \
  --ff2 \
  --method npo_KL \
  --cot_limit 6 \
  --verify_size 2 \
  --max_unlearn 1 \
  --epochs 1
```

### 查看结果文件

默认输出目录：

```text
final_results/<dataset>/<short_model>/
```

如果是 ablation：

```text
ablation/<dataset>/<short_model>/
```

如果跑了 `--mmlu`：

```text
mmlu_results/<dataset>/<short_model>/
```
