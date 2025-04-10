

### ✅ Part 1: Calculating BLEU, ROUGE, METEOR Scores

You have a dataset with the following columns:
- `question`
- `ground_truth` (reference text)
- `llm_answers` (generated text by the language model)

To evaluate how close the generated answers are to the ground truth, we use these common NLP metrics:

---

#### 🧮 Sample Python Code to Compute BLEU, ROUGE, METEOR

```python
from nltk.translate.bleu_score import sentence_bleu, SmoothingFunction
from nltk.translate.meteor_score import meteor_score
from rouge import Rouge
import pandas as pd

# Load your dataset
df = pd.read_csv("your_dataset.csv")  # Update with your path

# Initialize ROUGE
rouge = Rouge()
smooth = SmoothingFunction().method4

# Containers for storing scores
bleu_scores = []
meteor_scores = []
rouge_1 = []
rouge_2 = []
rouge_l = []

for _, row in df.iterrows():
    reference = row['ground_truth']
    hypothesis = row['llm_answers']
    
    # BLEU (use list of reference words and list of hypothesis words)
    bleu = sentence_bleu(
        [reference.split()],
        hypothesis.split(),
        smoothing_function=smooth
    )
    bleu_scores.append(bleu)

    # METEOR
    meteor = meteor_score([reference], hypothesis)
    meteor_scores.append(meteor)

    # ROUGE
    rouge_scores = rouge.get_scores(hypothesis, reference)[0]
    rouge_1.append(rouge_scores['rouge-1']['f'])
    rouge_2.append(rouge_scores['rouge-2']['f'])
    rouge_l.append(rouge_scores['rouge-l']['f'])

# Add the scores to the dataframe
df['bleu'] = bleu_scores
df['meteor'] = meteor_scores
df['rouge-1'] = rouge_1
df['rouge-2'] = rouge_2
df['rouge-l'] = rouge_l

# Save results
df.to_csv("scored_output.csv", index=False)
```

---

### ✅ Part 2: What the Scores Mean

| **Metric** | **Range** | **Meaning** |
|------------|-----------|-------------|
| **BLEU**   | 0 to 1    | Measures exact n-gram matches. Higher is better. BLEU ~ 0.35 = 35% match on average n-grams. Often lower for open-ended tasks. |
| **ROUGE-1**| 0 to 1    | Overlap of **unigrams** (single words). ROUGE-1 = 1 means perfect overlap in words. |
| **ROUGE-2**| 0 to 1    | Overlap of **bigrams** (pairs of words). Harder to match. |
| **ROUGE-L**| 0 to 1    | Longest Common Subsequence. Captures fluency and sequence similarity. |
| **METEOR** | 0 to 1    | More flexible than BLEU. Considers synonyms, stemming. Often more correlated with human judgment. |

---

### ⚠️ Notes on Interpretation

- **BLEU ~ 0.35** is decent for generation tasks. It's usually **< 0.5** in open-ended NLP tasks like QA or summarization.
- **ROUGE-1 = 1** suggests exact match in words — maybe short answers or yes/no style.
- **ROUGE-2** and **ROUGE-L** give insight into fluency and structural similarity.
- METEOR is often considered better aligned with human judgment than BLEU.

---

If you'd like, you can share a few samples or scores and I can help you interpret or improve them further.
