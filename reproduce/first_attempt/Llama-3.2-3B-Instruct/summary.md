# First Attempt Summary

## Basic Info

- Date: `2026-04-28 13:12:23 CST`
- Scale: `first_attempt`
- Model: `meta-llama/Llama-3.2-3B-Instruct`
- Dataset: `sqa`
- Command source: README 第一条命令

## Command

```bash
conda run -n faithfulness python unlearn.py --model_name meta-llama/Llama-3.2-3B-Instruct --strategy sentencize --stepwise --dataset sqa --lr 3e-05 --pos --ff2 --method npo_KL
```

## Result

本次尝试已经成功通过 Hugging Face gated model 鉴权，和未提供 token 时不同，这次没有出现 `401 Unauthorized` / `GatedRepoError`。

在我手动停止这次第一次尝试时，程序还处于模型权重下载阶段，尚未进入：

1. `Generating new CoTs ...`
2. 逐 step unlearning
3. `final_results/...` 结果写盘

## Snapshot At Stop Time

- Python process elapsed time: about `02:00`
- Process CPU: about `24.8%`
- Process MEM: about `26.2%`
- `run.log`: still empty
- No `final_results/`, `ablation/`, or `mmlu_results/` directories had been created yet

## Hugging Face Cache Snapshot

At stop time:

```text
~/.cache/huggingface/hub/models--meta-llama--Llama-3.2-3B-Instruct/blobs/13cbd6d16e927a0c5bad54102514e6e18b4a47b3a6eb911e39d678d328d19f55.incomplete  64M
~/.cache/huggingface/hub/models--meta-llama--Llama-3.2-3B-Instruct/blobs/7b770216613ac5c34d7c54bdff1fa616bc4e338a9d0b20af6303e48c295ee23c.incomplete  77M
```

## Interpretation

这说明第一次尝试已经验证了两件关键事情：

1. 现在的 HF token 可以访问 README 指定的 Meta-Llama 模型。
2. `unlearn.py` 的原始命令已经真正开始执行，不再停在权限报错。

这次还没有走到真正的论文实验主体，所以这是一次“启动成功但未完成”的第一次尝试记录。
