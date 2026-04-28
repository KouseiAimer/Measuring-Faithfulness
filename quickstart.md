# Parametric Faithfulness Quickstart

验证日期：2026-04-28  
工作目录：`/home/kousei/faithfulness/parametric-faithfulness`

## 1. 本次实际做了什么

我先检查了仓库入口、README 和 `unlearn.py`，确认 README 的第一条命令是：

```bash
python unlearn.py --model_name meta-llama/Llama-3.2-3B-Instruct --strategy sentencize --stepwise --dataset sqa --lr 3e-05 --pos --ff2 --method npo_KL
```

随后确认了两件现实约束：

1. 这条命令使用的 `meta-llama/Llama-3.2-3B-Instruct` 是 Hugging Face gated model。
2. 原始代码里有几个会直接影响首次复现的启动问题，需要先修。

## 2. 代码层面的必要修复

我对仓库做了以下最小、向后兼容的修复：

1. 修复 `--stepwise` 参数方向。
   原始代码把 `--stepwise` 写成了 `store_false`，导致 README 的命令语义反了。

2. 补上 `--atomic` 参数。
   `unlearn.py` 在 `main()` 里使用了 `args.atomic`，但 parser 里没有定义。

3. 把空字符串 Hugging Face 登录改成“有 token 就登录，没有就跳过”。
   否则 `login("")` 会让运行逻辑非常脆弱。

4. 给分句逻辑加了 fallback。
   原始代码依赖 `nltk.sent_tokenize()`，但很多环境没有 `punkt_tab`；现在缺资源时会退回规则分句，不再直接报错。

5. 新增了几个 smoke test 友好的参数，默认值不改变原始实验。
   `--cot_limit`
   `--verify_size`
   `--max_unlearn`

## 3. 环境配置

### 3.1 本机实际采用的方式

这台机器原本已经有一个成熟的 GPU 环境 `llm-26-gpu`，而同名环境 `faithfulness` 只是一个几乎空白的环境。  
为了避免重新下载整套 CUDA/PyTorch 栈，我实际采用了：

```bash
conda env remove -n faithfulness -y
conda create -n faithfulness --clone llm-26-gpu -y
```

然后补了仓库额外需要的包：

```bash
conda install -n faithfulness -y seaborn notebook sentencepiece
conda run -n faithfulness pip install openai
```

说明：

1. `lm-eval` 是可选依赖，只在 `mmlu.py` 或 `unlearn.py --mmlu/--gsm` 路线里需要。
2. 我把本次已验证过的依赖版本写到了 `requirements-faithfulness.txt`。

### 3.2 如果你的机器没有 `llm-26-gpu`

可以从零开始：

```bash
conda create -n faithfulness python=3.12 -y
conda install -n faithfulness -y pytorch torchvision torchaudio pytorch-cuda=12.4 -c pytorch -c nvidia
conda install -n faithfulness -y seaborn notebook sentencepiece
conda run -n faithfulness pip install transformers datasets accelerate huggingface_hub spacy nltk matplotlib pandas scipy openai
```

可选：

```bash
conda run -n faithfulness pip install lm-eval
```

## 4. 环境验证

我在 `faithfulness` 里确认了这些版本：

```text
torch==2.6.0+cu124
torchvision==0.21.0+cu124
torchaudio==2.6.0+cu124
transformers==5.3.0
datasets==4.6.1
accelerate==1.13.0
huggingface_hub==1.6.0
spacy==3.8.2
nltk==3.9.4
sentencepiece==0.2.1
matplotlib==3.10.8
pandas==2.3.3
scipy==1.17.1
seaborn==0.13.2
openai==2.32.0
notebook==7.5.3
ipykernel==7.2.0
```

并确认：

```bash
conda run -n faithfulness python -c "import torch; print(torch.cuda.is_available(), torch.cuda.device_count())"
```

输出为：

```text
True 1
```

## 5. README 第一条命令的真实结果

我按原样执行了：

```bash
conda run -n faithfulness python unlearn.py --model_name meta-llama/Llama-3.2-3B-Instruct --strategy sentencize --stepwise --dataset sqa --lr 3e-05 --pos --ff2 --method npo_KL
```

结果是：

```text
401 Unauthorized / GatedRepoError
Cannot access gated repo: meta-llama/Llama-3.2-3B-Instruct
```

这不是本地环境问题，而是模型权限问题。  
也就是说，这条 README 命令想成功跑起来，必须先给当前 shell 提供一个有 Meta-Llama 访问权限的 Hugging Face token。

推荐做法：

```bash
export HF_TOKEN=你的_huggingface_token
conda run -n faithfulness python unlearn.py --model_name meta-llama/Llama-3.2-3B-Instruct --strategy sentencize --stepwise --dataset sqa --lr 3e-05 --pos --ff2 --method npo_KL
```

## 6. 为了看 `unlearn.py` 的效果，我做了一个公开模型 smoke test

因为 README 原命令被 gated model 挡住，我改用公开模型 `microsoft/Phi-3-mini-4k-instruct` 做同路线验证：

```bash
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

这条命令已经成功进入模型下载阶段，说明：

1. `unlearn.py` 的参数解析已经正常。
2. 修复后的 `stepwise/sentencize/pos/ff2/npo_KL` 主路径可以启动。
3. Hugging Face 公共模型下载逻辑正常。

我停止它的原因不是报错，而是为了不把这次会话全部耗在等待大模型权重下载上。  
当前缓存目录里已经能看到 Phi-3 的下载痕迹：

```text
~/.cache/huggingface/hub/models--microsoft--Phi-3-mini-4k-instruct/
```

下次继续跑，会从缓存续传。

## 7. 建议你下一步怎么跑

### 方案 A：你有 Meta-Llama 权限

直接：

```bash
export HF_TOKEN=你的_huggingface_token
cd /home/kousei/faithfulness/parametric-faithfulness
conda run -n faithfulness python unlearn.py --model_name meta-llama/Llama-3.2-3B-Instruct --strategy sentencize --stepwise --dataset sqa --lr 3e-05 --pos --ff2 --method npo_KL
```

### 方案 B：你先看公开模型上的完整启动效果

先跑一个小规模 smoke test：

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

如果你想跑论文式的完整规模，再把这些 smoke test 参数去掉。

## 8. 结果文件会写到哪里

默认主实验：

```text
final_results/<dataset>/<short_model>/
```

如果用了 `--ablation`：

```text
ablation/<dataset>/<short_model>/
```

如果用了 `--mmlu`：

```text
mmlu_results/<dataset>/<short_model>/
```

## 9. 文件说明入口

仓库文件逐项说明已经写到：

```text
/home/kousei/faithfulness/settings.md
```
