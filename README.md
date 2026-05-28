# Benchmarking Small Language Models for Turkish Open-Ended Student Answer Quality Classification

This repository contains the final project materials for the **Large Language Models** course project by **Nisanur Genç** and **Ece Yılmaz**.

The project benchmarks small and instruction-tuned language models on a Turkish open-ended answer quality classification task. The main goal is to evaluate whether small language models can distinguish between **not high-quality** and **high-quality** Turkish educational answers using zero-shot prompting and parameter-efficient fine-tuning.

## Project Overview

Open-ended student answers provide richer evidence of learning than multiple-choice responses, but evaluating them manually is time-consuming and difficult to scale. This project investigates whether small language models can support Turkish educational answer assessment under realistic hardware constraints.

The study uses the **Turkish Education Dataset**, which contains question, context, answer, and score fields. Since the original score distribution is highly concentrated around high values, especially Score 8 and Score 9, the task is reformulated as a binary classification problem.

## Dataset

The dataset contains **17,587 raw examples**.

Each example includes:

- `soru`: question
- `context`: supporting passage
- `cevap`: answer
- `kaynak`: source
- `veri türü`: data type
- `Score`: original quality score

Invalid examples with `Score = -2` were removed because they represent non-scored examples. No `Score = -1` examples were observed in the downloaded version.

After filtering invalid examples, the dataset contains **14,469 valid samples**.

## Label Construction

Two binary labeling strategies were considered.

### Strategy A

| Label | Score Range | Interpretation |
|---|---|---|
| 0 | Score <= 7 | Not high-quality |
| 1 | Score >= 8 | High-quality |

Strategy A was too imbalanced because most examples had Score 8 or above.

### Strategy B

| Label | Score Range | Interpretation |
|---|---|---|
| 0 | Score <= 8 | Not high-quality |
| 1 | Score >= 9 | High-quality |

Strategy B was selected as the main experimental setup because it provides a more balanced class distribution.

Strategy B distribution:

| Label | Count | Percentage |
|---|---:|---:|
| 0 | 8,856 | 61.21% |
| 1 | 5,613 | 38.79% |

## Preprocessing

The model input was constructed by combining the question, context, and answer into a single `input_text` field:

```text
Soru: [question text]
Bağlam: [context text]
Cevap: [answer text]
```

The question and answer fields were kept complete. The context field was truncated to the first 2,500 characters to reduce input length while preserving the main supporting information.

The main Strategy B split uses an 80/10/10 train-validation-test split with stratified sampling and a fixed random seed.

| Split | Total Examples | Label 0 | Label 1 |
|---|---:|---:|---:|
| Train | 11,575 | 7,085 | 4,490 |
| Validation | 1,447 | 886 | 561 |
| Test | 1,447 | 885 | 562 |

## Evaluated Models

The project evaluates the following small or compact instruction-tuned models:

| Model | Size | Evaluation Setting |
|---|---:|---|
| SmolLM2-360M-Instruct | 360M | Zero-shot + LoRA fine-tuning |
| TinyLlama-1.1B-Chat-v1.0 | 1.1B | Zero-shot + LoRA fine-tuning |
| Qwen2.5-1.5B-Instruct | 1.5B | Zero-shot + QLoRA fine-tuning |
| Gemma-4-E2B-it | E2B | Zero-shot + attempted QLoRA fine-tuning |

## Evaluation Metrics

Models are evaluated using:

- Accuracy
- Macro F1-score
- Weighted F1-score

Macro F1-score is treated as the primary metric because the dataset is moderately imbalanced and accuracy can be misleading when a model collapses into the majority class.

## Main Results

### Baseline

The majority class baseline always predicts label 0.

| Method | Accuracy | Macro F1 | Weighted F1 |
|---|---:|---:|---:|
| Majority Class Baseline | 0.6116 | 0.3795 | 0.4642 |

Although the baseline accuracy is relatively high, it completely fails to identify label 1 examples.

### Zero-Shot Evaluation

Zero-shot prompting showed unstable behavior across models. Some models collapsed into a single label, while TinyLlama did not reliably produce parseable binary outputs.

| Model | Sample Size | Prompt | Accuracy | Macro F1 | Weighted F1 | Notes |
|---|---:|---|---:|---:|---:|---|
| Qwen2.5-1.5B-Instruct | 100 | v3 | 0.5700 | 0.5243 | 0.5243 | Best Qwen zero-shot setting; biased toward label 1 |
| SmolLM2-360M-Instruct | 20 | v3 | 0.5000 | 0.3333 | 0.3333 | Predicted label 0 for all examples |
| TinyLlama-1.1B-Chat-v1.0 | 20 | v3 | N/A | N/A | N/A | Did not reliably return parseable labels |
| Gemma-4-E2B-it | 100 | v3 | 0.5354 | 0.5088 | 0.5100 | One output was not parseable and excluded |

### Fine-Tuning Results

| Model | Method | Accuracy | Macro F1 | Weighted F1 | Notes |
|---|---|---:|---:|---:|---|
| SmolLM2-360M-Instruct | LoRA | 0.5888 | 0.4333 | 0.4996 | Strong bias toward label 0 |
| TinyLlama-1.1B-Chat-v1.0 | LoRA | 0.5460 | 0.4924 | 0.5292 | More balanced than SmolLM2, but still under-detects label 1 |
| Qwen2.5-1.5B-Instruct | QLoRA | 0.5715 | 0.5071 | 0.5469 | Best fine-tuned macro F1 among completed runs |
| Gemma-4-E2B-it | QLoRA attempted | N/A | N/A | N/A | Fine-tuning not completed due to model architecture, memory, and PEFT compatibility limitations |

Among the completed fine-tuning runs, **Qwen2.5-1.5B-Instruct** achieved the best macro F1-score.

## Key Findings

- Accuracy alone is not sufficient for this task.
- Macro F1-score is more informative because models often collapse toward one label.
- Zero-shot prompting is unstable across small instruction-tuned models.
- Fine-tuning improves class-balanced performance compared with the majority baseline, but does not fully solve the task.
- All fine-tuned models still struggle to identify label 1 examples reliably.
- The main difficulty is the boundary between Score 8 and Score 9 answers.
- Gemma-4-E2B-it could not be fine-tuned under the available Colab T4 setup due to model compatibility and memory limitations.

## Repository Structure

```text
data/
  processed/
    strategy_a/
    strategy_b/

notebooks/
  04_zero_shot_evaluation.ipynb
  05_zero_shot_gemma.ipynb
  09_finetune_smollm2.ipynb
  10_finetune_tinyllama.ipynb
  07_finetune_qwen.ipynb
  08_finetune_gemma_qlora.ipynb

outputs/
  tables/
    final_model_comparison.csv
    smollm2_finetune_results.csv
    smollm2_test_predictions.csv
    tinyllama_finetune_results.csv
    tinyllama_test_predictions.csv
    qwen_qlora_longrun_results.csv
    qwen_qlora_longrun_test_predictions.csv
```

## Important Output Files

| File | Description |
|---|---|
| `outputs/tables/final_model_comparison.csv` | Final comparison of baseline, zero-shot, and fine-tuned models |
| `outputs/tables/smollm2_finetune_results.csv` | SmolLM2 fine-tuning metrics |
| `outputs/tables/smollm2_test_predictions.csv` | SmolLM2 test predictions |
| `outputs/tables/tinyllama_finetune_results.csv` | TinyLlama fine-tuning metrics |
| `outputs/tables/tinyllama_test_predictions.csv` | TinyLlama test predictions |
| `outputs/tables/qwen_qlora_longrun_results.csv` | Qwen QLoRA long-run metrics |
| `outputs/tables/qwen_qlora_longrun_test_predictions.csv` | Qwen QLoRA test predictions |

## Reproducibility Notes

The experiments were conducted under limited Colab GPU resources. Parameter-efficient fine-tuning methods were used instead of full fine-tuning.

Qwen2.5-1.5B-Instruct was trained with QLoRA using:

| Hyperparameter | Value |
|---|---|
| Maximum input length | 768 tokens |
| Maximum training steps | 600 |
| Training batch size | 1 |
| Gradient accumulation steps | 16 |
| Learning rate | 2e-5 |
| Evaluation interval | Every 200 steps |
| Checkpoint interval | Every 200 steps |
| Primary selection metric | Macro F1 |

Gemma-4-E2B-it fine-tuning was attempted but not completed because:

1. `AutoModelForSequenceClassification` did not support the Gemma 4 configuration.
2. Causal LM QLoRA with `prepare_model_for_kbit_training` caused CUDA out-of-memory.
3. Causal LM QLoRA without that preparation step failed because `Gemma4ClippableLinear` modules were not supported by the current PEFT LoRA injection setup.

## Limitations

- The dataset is LLM-generated and scored by an LLM, not by human expert annotators.
- The task depends on score-based thresholding.
- The boundary between Score 8 and Score 9 may be ambiguous.
- Zero-shot evaluation used balanced samples rather than always using the full test set.
- Full fine-tuning was not performed due to hardware constraints.
- Gemma fine-tuning could not be completed under the available setup.
- Human evaluation was not included.

## Authors

- Nisanur Genç - 258273002044
- Ece Yılmaz - 258273002009

## Course

Large Language Models Course - Final Project
