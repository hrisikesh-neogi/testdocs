
# ✅ Evaluating Generated Text: BLEU, ROUGE, METEOR in a RAG Pipeline

In Retrieval-Augmented Generation (RAG) systems, evaluating the quality of generated answers is essential, especially when **ground truth** answers are available. This evaluation allows us to measure how closely the generated responses align with expected outputs. Common metrics used in this context are **BLEU**, **ROUGE**, and **METEOR**.

---

## 🔹 1. BLEU (Bilingual Evaluation Understudy)

### 📘 Definition:
BLEU is a precision-based metric that evaluates how many **n-gram overlaps** exist between the generated and reference text. Originally designed for machine translation, it's widely adopted in other generation tasks.

### 📈 Score Interpretation:
| BLEU Score | Interpretation |
|------------|----------------|
| 0.0–0.2    | Very low overlap – potential hallucination |
| 0.2–0.4    | Partial match, but significant paraphrasing or missing details |
| 0.4–0.6    | Reasonable n-gram overlap |
| 0.6–1.0    | High overlap – mostly or exactly matches the reference |

> **Example:**
- **Reference**: "Insulin resistance leads to Type 2 diabetes."
- **Generated**: "Type 2 diabetes is caused by insulin resistance."
- **BLEU**: ~0.3–0.4 (due to different word order and some paraphrasing)

### ⚠️ Limitations:
- Only considers **exact word matches** (e.g., “rain” ≠ “raining”).
- Ignores **synonyms**, **semantics**, or **word order** importance.
- Penalizes correct paraphrases and flexible language usage.
- Focuses on **precision** without considering **recall** (i.e., doesn’t check if all important reference tokens were captured).

### 📌 When to Use in RAG:
Best suited for evaluating **factual or short-form answers** where precision of terms is crucial (e.g., definitions, numbers, names, etc.).

---

## 🔹 2. ROUGE (Recall-Oriented Understudy for Gisting Evaluation)

### 📘 Definition:
ROUGE focuses on **recall** by measuring how much of the reference text is present in the generated answer. It includes:
- **ROUGE-1**: Unigram overlap
- **ROUGE-2**: Bigram overlap
- **ROUGE-L**: Longest common subsequence (LCS)

### 📈 Score Interpretation:
| ROUGE Score | Interpretation |
|-------------|----------------|
| ~0.0–0.3     | Limited content overlap |
| ~0.3–0.6     | Moderate coverage |
| ~0.6–1.0     | Strong alignment with reference content |

> **Example:**
- **Reference**: "Insulin resistance and high blood sugar lead to Type 2 diabetes."
- **Generated**: "Type 2 diabetes is primarily caused by insulin resistance and elevated glucose."
- **ROUGE-1**: High (~0.7–0.8)
- **ROUGE-L**: Also relatively high, due to similar phrase order

### ⚠️ Limitations:
- Sensitive to **stopwords** and common words.
- Doesn’t differentiate between important and less relevant tokens.
- ROUGE-2 tends to be low for paraphrased or short outputs.
- Can be inflated by repeated words or long answers.

### 📌 When to Use in RAG:
Helpful in **evaluating coverage** and structure in **long-form responses or summaries**. Suitable when assessing whether the generated answer captures all **key concepts** from the reference.

---

## 🔹 3. METEOR (Metric for Evaluation of Translation with Explicit ORdering)

### 📘 Definition:
METEOR evaluates based on both **precision and recall**, but with enhancements. It matches not just exact words but also considers:
- **Stemming**
- **Synonyms**
- **Paraphrasing**
- **Word order**

### 📈 Score Interpretation:
| METEOR Score | Interpretation |
|--------------|----------------|
| < 0.3        | Weak similarity |
| 0.3–0.5      | Moderate similarity, with partial semantic alignment |
| 0.5–0.7      | Strong similarity, with good paraphrasing or synonym use |
| > 0.7        | High-quality semantic match |

> **Example:**
- **Reference**: "Type 2 diabetes is caused by insulin resistance and high sugar levels."
- **Generated**: "Insulin resistance and increased blood sugar are key reasons for Type 2 diabetes."
- **METEOR**: ~0.6–0.7 (captures paraphrasing and synonyms)

### ⚠️ Limitations:
- Slower to compute than BLEU or ROUGE.
- Slightly more sensitive to sentence structure variation.
- May produce similar scores for outputs with different human-perceived quality.

### 📌 When to Use in RAG:
Ideal for **open-ended**, **semantically-rich** answers. Especially useful when paraphrasing and flexible language are expected.

---

## ✅ Which Metric to Use in RAG (With Ground Truth)?

| Metric  | Strength | Best Use Case in RAG |
|---------|----------|----------------------|
| **BLEU** | Measures **exact** n-gram precision | When factual correctness and exact terminology are critical (e.g., dates, codes, definitions) |
| **ROUGE** | Captures **coverage** and **sequence similarity** | When you want to ensure the generated text includes all necessary concepts (e.g., summaries, long answers) |
| **METEOR** | Recognizes **semantic similarity**, **synonyms**, and **order** | Best for **paraphrased** or **human-like generation**; reflects human judgment more closely |

---

## 🔬 Example Scenario in a RAG Workflow

- **Question**: "What causes Type 2 diabetes?"
- **Ground Truth**: "Type 2 diabetes is primarily caused by insulin resistance and high blood sugar levels."
- **Generated Answer**: "Insulin resistance and elevated glucose are the main factors behind Type 2 diabetes."

| Metric | Score Range | Insight |
|--------|-------------|---------|
| BLEU   | ~0.3–0.4    | Lower due to word order and paraphrasing |
| ROUGE-1 | ~0.75–0.85 | High unigram overlap |
| ROUGE-L | ~0.6–0.7   | Moderate structural similarity |
| METEOR | ~0.65–0.75  | High semantic alignment; synonyms recognized |

---

## 🛠️ Python Snippet to Compute the Scores

```python
from nltk.translate.bleu_score import sentence_bleu, SmoothingFunction
from nltk.translate.meteor_score import meteor_score
from rouge import Rouge
import pandas as pd

df = pd.read_csv("your_dataset.csv")
rouge = Rouge()
smooth = SmoothingFunction().method4

bleu_scores, meteor_scores = [], []
rouge_1, rouge_2, rouge_l = [], [], []

for _, row in df.iterrows():
    ref = row['ground_truth']
    hypo = row['llm_answers']
    
    bleu = sentence_bleu([ref.split()], hypo.split(), smoothing_function=smooth)
    bleu_scores.append(bleu)
    
    meteor_scores.append(meteor_score([ref], hypo))
    
    r_scores = rouge.get_scores(hypo, ref)[0]
    rouge_1.append(r_scores['rouge-1']['f'])
    rouge_2.append(r_scores['rouge-2']['f'])
    rouge_l.append(r_scores['rouge-l']['f'])

df['bleu'] = bleu_scores
df['meteor'] = meteor_scores
df['rouge-1'] = rouge_1
df['rouge-2'] = rouge_2
df['rouge-l'] = rouge_l

df.to_csv("scored_output.csv", index=False)
```

---

## 📊 Conclusion

- Choose **BLEU** for **exact factual** accuracy.
- Use **ROUGE** when **coverage** and **content presence** matter.
- Prefer **METEOR** for measuring **human-like, semantically aligned responses**.

A combination of these metrics offers a more comprehensive picture of RAG system performance, especially in critical evaluation or fine-tuning workflows.

---
