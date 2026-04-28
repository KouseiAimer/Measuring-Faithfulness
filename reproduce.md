# Reproduction Log

这个文件用于记录后续所有复现尝试。

## 目录约定

复现产物统一保存在：

```text
/home/kousei/faithfulness/reproduce/<scale>/<model>/
```

当前约定：

- `first_attempt/`：第一次尝试或启动验证
- `full_scale/`：后续按 README 默认规模进行的完整复现

## 记录模板

### [日期] [规模] [模型]

- Command:
- Environment:
- Status:
- Output files:
- Notes:

## 任务规模判断

### 2026-04-28 scale_estimate Llama-3.2-3B-Instruct

- Basis:
  基于 `unlearn.py` 默认参数和仓库内 `minimal_mistake_results/sqa/LLaMA-3-3B/npo_KL_lr=3e-05_rs=1001_pos=True_ff2=True.out` 的历史结果统计。
- Key numbers:
  `cot_limit=250`，`verify_size=20`，因此默认会得到 `230` 个训练样本进入 unlearning。
  `--stepwise` 模式下，历史结果显示这 `230` 个样本平均每个样本 `5.583` 个 step，最少 `1` 个，最多 `14` 个，总计约 `1284` 次 stepwise 子运行。
- Implication:
  这不是几分钟级脚本，而是一个默认就偏“完整实验”的任务。
  另外 `unlearn_single()` 每个 step 都会重新加载两份同模型（`model` 和 `oracle_model`），因此本地 8GB GPU 的可行性重点不只是“能否启动”，还包括是否会走 CPU/显存混合放置。

## 当前状态

### 2026-04-28 first_attempt Llama-3.2-3B-Instruct

- Command:
  `conda run -n faithfulness python unlearn.py --model_name meta-llama/Llama-3.2-3B-Instruct --strategy sentencize --stepwise --dataset sqa --lr 3e-05 --pos --ff2 --method npo_KL`
- Environment:
  `conda` env = `faithfulness`
- Status:
  部分成功。HF token 生效，命令已经通过 gated model 鉴权，不再报 `401 Unauthorized`；在本次第一次尝试结束时，程序还停留在 Meta-Llama-3.2-3B-Instruct 权重下载阶段，尚未进入 CoT 生成和 unlearning。
- Output files:
  `/home/kousei/faithfulness/reproduce/first_attempt/Llama-3.2-3B-Instruct/command.txt`
  `/home/kousei/faithfulness/reproduce/first_attempt/Llama-3.2-3B-Instruct/run.log`
  `/home/kousei/faithfulness/reproduce/first_attempt/Llama-3.2-3B-Instruct/summary.md`
- Notes:
  1. 这次和之前最大的区别是：现在已经能访问 `meta-llama/Llama-3.2-3B-Instruct`。
  2. 我在 2026-04-28 13:12:23 CST 停止了第一次尝试，目的是把“首次尝试结果”稳定归档，而不是让这次会话长时间挂在模型下载上。
  3. 停止时 Hugging Face 缓存目录下已经存在两个正在写入的权重分片：
     `13cbd6d16e927a0c5bad54102514e6e18b4a47b3a6eb911e39d678d328d19f55.incomplete`，约 `64M`
     `7b770216613ac5c34d7c54bdff1fa616bc4e338a9d0b20af6303e48c295ee23c.incomplete`，约 `77M`
  4. 下次继续跑同一条命令时，会优先复用现有 Hugging Face 缓存，而不是从零开始。

### 2026-04-28 full_scale attempt_01 Llama-3.2-3B-Instruct

- Command:
  `conda run -n faithfulness python unlearn.py --model_name meta-llama/Llama-3.2-3B-Instruct --strategy sentencize --stepwise --dataset sqa --lr 3e-05 --pos --ff2 --method npo_KL`
- Environment:
  `conda` env = `faithfulness`
  local GPU = `NVIDIA GeForce RTX 4060 Laptop GPU 8GB`
- Status:
  运行中。当前已经进入 `python unlearn.py ...` 主进程，但仍在补齐 Hugging Face 模型权重，尚未输出 CoT 生成或 unlearning 日志。
- Output files:
  `/home/kousei/faithfulness/reproduce/full_scale/Llama-3.2-3B-Instruct/attempt_01/command.txt`
  `/home/kousei/faithfulness/reproduce/full_scale/Llama-3.2-3B-Instruct/attempt_01/run.log`
- Notes:
  1. 当前使用 `HF_HUB_DISABLE_XET=1`，避免之前 `xet` 下载路径触发的 `416 Range Not Satisfiable` 问题。
  2. 截至记录时，模型第二个权重分片已经完整到位，首个分片仍在下载中。
  3. 一旦越过模型准备阶段，本次记录会继续补充“是否成功进入 CoT 生成”“是否成功进入 unlearning”“是否在本地 GPU 上跑通第一批 step”的结果。
