# 🇻🇳 ViT5-Summarization: Vietnamese Text Summarization Project

This project focuses on fine-tuning the **ViT5-base** model for the Vietnamese Text Summarization task (both extractive and abstractive).

## 1. Project Goal

The main goal is to build a model capable of generating concise and accurate abstractive summaries from a long Vietnamese article or document, leveraging the power of modern Sequence-to-Sequence models.

## 2. Data

| Dataset Name | Source | Total Samples | Description |
| :--- | :--- | :--- | :--- |
| `Dataset_summarization` | twice123nd/Dataset_summarization | **2986** | Contains columns: `title`, `document` (original text), and `summary` (target summary). |

The data is split into three subsets with the following proportions:

| Data Subset | Number of Samples | Proportion |
| :--- | :--- | :--- |
| **Train** | 2388 | 80% |
| **Validation** | 299 | 10% |
| **Test** | 299 | 10% |

## 3. Model & Preprocessing

### Base Model
The project uses the **ViT5-base** model (`VietAI/vit5-base`). This is a T5 version specifically trained on a Vietnamese corpus, making it suitable for Vietnamese text generation tasks.

### Preprocessing
* **Input:** Created by concatenating the `title` and `document` (e.g., `[Title]: [Document]...`).
* **Tokenization:**
    * Max Length for Input: **1024** tokens.
    * Max Length for Target (Summary): **512** tokens.

## 4. Training Configuration (Fine-tuning)

The training process is performed using the `Seq2SeqTrainer` from the `transformers` library with the following main parameters:

* **Model:** `VietAI/vit5-base`
* **Number of Epochs:** 3
* **Learning Rate:** 2e-5
* **Batch Size:** 2 (with `gradient_accumulation_steps=4`)
* **Optimization:** Uses **FP16** (mixed precision training) for faster processing and reduced GPU memory usage.

### Training Results (Loss)

After 3 epochs, the model achieved the following initial results:

| Step | Training Loss | Validation Loss |
| :---: | :---: | :---: |
| 100 | 1.9574 | 1.7418 |
| 400 | 1.5677 | **1.6828** |
| **Final** | **1.7933** | - |

## 5. Data Augmentation Strategy

To potentially improve performance, a data augmentation strategy was implemented:

* **Synthetic Data Generation:** Used the **GPT-4o-mini** API to create an additional **500 synthetic samples** of articles and summaries.
* **Usage:** The synthetic data was combined with the original training set to form an augmented training dataset (`aug_train_dataset`).

## 6. Evaluation Methods

In addition to automatic metrics, the project uses a Large Language Model (LLM-as-a-Judge) for a comprehensive evaluation:

### Automatic Metrics
* **ROUGE:** ROUGE-1, ROUGE-2, ROUGE-L.
* **BERTScore:** Measures the semantic similarity between the predicted summary and the reference summary.

### LLM-based Evaluation (GPT-based Evaluation)
GPT-4o-mini is used to score the summaries on a 1-5 scale based on 5 criteria:
1.  **Fluency**
2.  **Relevance**
3.  **Faithfulness** (to the source document)
4.  **Conciseness**
5.  **Average** (Overall Score)