# Relationship Flag Classifier

## Project Overview

This project builds a text classification system that classifies posts from the Reddit community `r/relationship_advice` into three categories:

- **Green Flag**
- **Yellow Flag**
- **Red Flag**

The goal was to compare a zero-shot LLM baseline against a fine-tuned text classification model and evaluate how well each model could distinguish between different levels of relationship concerns.

The project focused on data collection, label design, annotation consistency, prompt-based classification, model fine-tuning, and error analysis.

---

## Community and Task

### Community: `r/relationship_advice`

I chose `r/relationship_advice` because the community contains many posts describing interpersonal situations, relationship conflicts, boundaries, communication issues, and supportive relationship behaviors. This makes it a useful dataset for a classification task based on the type and severity of behavior described in a post.

The task is challenging because the difference between a Green Flag, Yellow Flag, and Red Flag can depend heavily on context. Some situations are clearly positive or clearly concerning, while others fall near the boundary between categories.

---

# Dataset

## Data Source

The dataset was collected from posts from the Reddit community `r/relationship_advice`.

The final dataset contains **200 labeled examples**.

### Label Distribution

| Label | Number of Examples |
|---|---:|
| Green Flag | 67 |
| Yellow Flag | 67 |
| Red Flag | 66 |
| **Total** | **200** |

The dataset was intentionally kept approximately balanced so that no single class dominated the training data.

### Data Collection

Examples were collected from `r/relationship_advice` and labeled according to the definitions established in `planning.md`.

The dataset contains the following columns:

- `text` — the relationship-advice post or situation
- `label` — Green Flag, Yellow Flag, or Red Flag
- `source_url` — the Reddit source when available

---

# Label Definitions

### Green Flag

The situation does not present a significant concern, and the behavior described is generally normal, healthy, respectful, or supportive.

**Example 1:**

> "My partner listens when I bring up a concern, communicates openly, and tries to understand my perspective instead of becoming defensive."

**Example 2:**

> "My partner respects my friendships and supports my personal goals without expecting me to give them up for the relationship."


### Yellow Flag

The situation is concerning or unusual, but it is not necessarily a serious problem and may be resolved through communication, additional context, or a reasonable change in behavior.

**Example 1:**

> "My partner is uncomfortable with my friendship with an ex, but they have not tried to control the friendship and are willing to discuss reasonable boundaries."

**Example 2:**

> "My partner and I have different expectations about how often we should communicate during the day, and we are trying to find a compromise that works for both of us."


### Red Flag

The situation involves seriously concerning behavior, such as a major boundary violation, dishonesty, disrespect, manipulation, coercive control, repeated harmful behavior, or behavior that warrants addressing the situation directly.

**Example 1:**

> "My partner secretly checks my private messages after I explicitly told them that doing so is not okay."

**Example 2:**

> "My partner uses repeated threats, insults, and control to get their way."

---

## Error Analysis

The fine-tuned model made 10 incorrect predictions out of 30 test examples. I selected three errors for closer analysis because they represent different types of classification mistakes.

### Error 1: Yellow Flag → Red Flag

**Post:**

> "My partner has recently become more short-tempered and sometimes blows up over small disagreements, but usually apologizes afterward."

- **True label:** Yellow Flag
- **Predicted label:** Red Flag
- **Confidence:** 0.34

**Analysis:**  
This example is difficult because it falls near the boundary between Yellow and Red. The partner's behavior is concerning because they become short-tempered and sometimes blow up over small disagreements. However, the additional context that the partner usually apologizes makes the situation less clearly severe than a Red Flag. The model appears to have focused more heavily on the negative behavior than on the mitigating context. This suggests that the model may have difficulty distinguishing concerning behavior that may be addressed through communication from behavior that represents a serious or repeated harmful pattern.

**Potential improvement:**  
Include more Yellow Flag training examples where concerning behavior is paired with mitigating context, such as apologies, willingness to communicate, or attempts to change.

---

### Error 2: Green Flag → Red Flag

**Post:**

> "My partner understands that wanting private space does not mean I love them less."

- **True label:** Green Flag
- **Predicted label:** Red Flag
- **Confidence:** 0.34

**Analysis:**  
This example represents a different type of error because the post describes a healthy approach to personal boundaries. The partner understands that wanting private space does not mean the relationship is less important. This aligns with the project's Green Flag definition of healthy, respectful, and supportive behavior. However, the model predicted Red Flag. This suggests that the fine-tuned model did not consistently learn the positive characteristics that distinguish Green Flags from concerning relationship behavior.

**Potential improvement:**  
Add more diverse Green Flag examples involving healthy boundaries, independence, communication, and respect for personal space.

---

### Error 3: Red Flag → Green Flag

**Post:**

> "My partner uses repeated threats, insults, and control to get their way."

- **True label:** Red Flag
- **Predicted label:** Green Flag
- **Confidence:** 0.35

**Analysis:**  
This example is particularly important because the behavior directly matches several parts of the project's Red Flag definition, including threats, disrespect, and controlling behavior. Despite this, the model predicted Green Flag. Unlike the first example, this is not primarily an ambiguous Green/Yellow or Yellow/Red boundary case. Instead, it demonstrates that the fine-tuned model sometimes failed to recognize clearly severe behavior. This suggests that the model's learned decision boundary did not consistently associate strong indicators such as threats and control with the Red Flag category.

**Potential improvement:**  
Include more clearly labeled Red Flag examples involving threats, coercive control, manipulation, and repeated harmful behavior so that these behaviors are more strongly represented during training.
---

# Model Approaches

## Baseline: Zero-Shot Classification

The baseline used a prompt-based classification approach with the Groq API.

The model was instructed to classify each `r/relationship_advice` post into exactly one of the three labels.

The prompt included:

SYSTEM_PROMPT = """You are classifying posts from r/relationship_advice into one of three
relationship-behavior categories.

Assign each post to exactly ONE category.

Green Flag: The situation does not present a significant concern, and the
behavior described is generally normal, healthy, respectful, or supportive.

Example: "My partner listens when I bring up a concern, communicates openly,
and tries to understand my perspective instead of becoming defensive."

Yellow Flag: The situation is concerning or unusual, but it is not necessarily
a serious problem and may be resolved through communication, additional
context, or a reasonable change in behavior.

Example: "My partner is uncomfortable with my friendship with an ex, but they
have not tried to control the friendship and are willing to discuss reasonable
boundaries."

Red Flag: The situation involves seriously concerning behavior, such as a
major boundary violation, dishonesty, disrespect, manipulation, coercive
control, repeated harmful behavior, or behavior that warrants addressing the
situation directly.

Example: "My partner secretly checks my private messages after I explicitly
told them that doing so is not okay."

When classifying, focus on the behavior and situation described. Do not rely
only on keywords such as "toxic," "red flag," or "jealous." Consider the
context provided and do not assume information that is not stated.

IMPORTANT OUTPUT RULE:
Your response MUST be exactly ONE of these three labels:

Green Flag
Yellow Flag
Red Flag

Do not provide an explanation.
Do not provide reasoning.
Do not add punctuation.
Do not use quotation marks.
Do not write anything before or after the label.

Example of valid output:
Yellow Flag

Example of invalid output:
The situation is a Yellow Flag because...

Now classify the post."""

The baseline model used was:

`openai/gpt-oss-120b`

The model was evaluated on a 30-example test set.

All 30 responses were successfully parsed.

---

# Fine-Tuning Pipeline

## Base Model

The fine-tuned model was:

**DistilBERT**

Training was performed using:

**Google Colab with a T4 GPU**

The fine-tuning process trained the model for **3 epochs**.

### Training Results

| Epoch | Training Loss | Validation Loss | Validation Accuracy |
|---|---:|---:|---:|
| 1 | No log | 1.101644 | 33.3% |
| 2 | 1.108051 | 1.089369 | 43.3% |
| 3 | 1.095776 | 1.068744 | 73.3% |

The validation accuracy increased substantially during training, from 33.3% after the first epoch to 73.3% after the third epoch.

The validation loss also decreased from 1.101644 to 1.068744.

### Key Training Decision

The model was trained for **3 epochs**.

3 is a good default for small datasets. More epochs risk overfitting on 200 examples.

Other training settings:

- Learning rate: 2e-5 is the standard starting point for fine-tuning BERT-family models. Lower → slower but more stable.
- Batch size: 16

---

# Evaluation

Both models were evaluated on the **same 30-example test set** so that their performance could be compared directly.

## Overall Results

| Model | Test Accuracy |
|---|---:|
| Zero-shot baseline (Groq) | **90.0%** |
| Fine-tuned DistilBERT | **66.7%** |

The fine-tuned model performed **23.3 percentage points lower** than the zero-shot baseline on the test set.

The comparison was:

> 0.667 − 0.900 = −0.233

Therefore, fine-tuning resulted in a **23.3 percentage-point regression** on this particular test set.

---

# Baseline Per-Class Metrics

| Class | Precision | Recall | F1-Score | Support |
|---|---:|---:|---:|---:|
| Green Flag | 0.77 | 1.00 | 0.87 | 10 |
| Yellow Flag | 1.00 | 0.70 | 0.82 | 10 |
| Red Flag | 1.00 | 1.00 | 1.00 | 10 |
| **Macro Average** | **0.92** | **0.90** | **0.90** | **30** |
| **Weighted Average** | **0.92** | **0.90** | **0.90** | **30** |

The baseline's main weakness was the distinction between **Green Flag and Yellow Flag**.

Green Flag had perfect recall (1.00), meaning all actual Green Flag examples were identified, but its precision was lower (0.77), indicating that some examples predicted as Green Flag were actually Yellow Flag.

Yellow Flag had perfect precision (1.00), but its recall was only 0.70, meaning some actual Yellow Flag examples were classified as another category.

Red Flag had perfect precision, recall, and F1-score on the test set.

---

# Fine-Tuned Model Per-Class Metrics

| Class | Precision | Recall | F1-Score | Support |
|---|---:|---:|---:|---:|
| Green Flag | 0.73 | 0.80 | 0.76 | 10 |
| Yellow Flag | 0.71 | 0.50 | 0.59 | 10 |
| Red Flag | 0.58 | 0.70 | 0.64 | 10 |
| **Macro Average** | **0.67** | **0.67** | **0.66** | **30** |
| **Weighted Average** | **0.67** | **0.67** | **0.66** | **30** |

The fine-tuned model performed best on Green Flag examples, with an F1-score of 0.76. Yellow Flag had the lowest recall at 0.50, meaning that half of the actual Yellow Flag examples were classified as another category. Red Flag had a recall of 0.70 but lower precision because several examples predicted as Red Flag were actually Green or Yellow Flag.
---

# Confusion Matrix

The confusion matrix below shows where the fine-tuned model confused the three labels.

<img width="1050" height="750" alt="confusion_matrix" src="https://github.com/user-attachments/assets/54ee3b78-e5cd-449d-b6f1-1a23a755d6d6" />

### Confusion Matrix as a Markdown Table


| Actual \ Predicted | Green Flag | Yellow Flag | Red Flag |
|---|---:|---:|---:|
| **Green Flag** | 8 | 0 | 2 |
| **Yellow Flag** | 2 | 5 | 3|
| **Red Flag** | 1 | 2 | 7 |

---

# Error Analysis

The fine-tuned model incorrectly classified **10 of 30 test examples**.

Its incorrect predictions included errors in multiple directions rather than being limited to a single class boundary.

The incorrect predictions also had relatively low confidence, ranging from approximately **0.34 to 0.36**.

## Error 1: Yellow Flag → Red Flag

**Post:**

> "My partner has recently become more short-tempered and sometimes blows up over small disagreements, but usually apologizes afterward."

**True label:** Yellow Flag  
**Predicted label:** Red Flag  
**Confidence:** 0.34

### Analysis

This example sits near the Yellow/Red boundary. The behavior is concerning because the partner becomes short-tempered and blows up over relatively small disagreements. However, the post also provides mitigating context: the partner usually apologizes afterward.

The model appears to have emphasized the highly negative behavior ("blows up") more heavily than the contextual information about apologizing and the absence of additional evidence of severe or controlling behavior.

This suggests that the model may have difficulty distinguishing **concerning behavior that may require communication** from behavior that meets the project's definition of a **serious Red Flag**.

A potential improvement would be to provide more training examples showing Yellow Flag situations involving concerning behavior that do not rise to the Red Flag level.

---

## Error 2: Green Flag → Red Flag

**Post:**

> "My partner understands that wanting private space does not mean I love them less."

**True label:** Green Flag  
**Predicted label:** Red Flag  
**Confidence:** 0.34

### Analysis

This example describes a supportive and healthy response to personal boundaries. The partner understands that wanting private space does not indicate a lack of love.

The model nevertheless classified the example as Red Flag. This indicates that the fine-tuned model did not consistently recognize positive relationship behavior as Green Flag behavior.

The error is particularly notable because the sentence directly reflects one of the intended characteristics of a Green Flag: respecting personal space and boundaries.

This could indicate that the training data did not contain enough diverse positive examples involving boundaries, independence, and healthy communication.

---

## Error 3: Red Flag → Green Flag

**Post:**

> "My partner uses repeated threats, insults, and control to get their way."

**True label:** Red Flag  
**Predicted label:** Green Flag  
**Confidence:** 0.35

### Analysis

This example contains several behaviors explicitly associated with the project's Red Flag definition: repeated threats, insults, and controlling behavior.

The model nevertheless predicted Green Flag. This is an important error because the behavior is not merely ambiguous or mildly concerning; it directly matches multiple characteristics used to define the Red Flag category.

This suggests that the fine-tuned model did not reliably learn the relationship between severe behavioral indicators and the Red Flag label.

One possible improvement would be to include more training examples containing clearly severe behaviors such as threats, coercive control, manipulation, and repeated boundary violations.

---

# AI-Assisted Error Pattern Analysis

Before completing the final error analysis, I used an AI tool to examine the fine-tuned model's 10 incorrect predictions and look for patterns across the errors.

### AI Prompt

I provided the AI tool with the incorrect predictions, including the true label, predicted label, confidence score, and post text. I asked it to identify recurring patterns in the errors, including which label pairs were most frequently confused, whether the errors occurred near category boundaries, whether post length or limited context appeared to contribute, whether certain words or behaviors were associated with incorrect predictions, and whether the model appeared to overlook contextual information.

### AI-Suggested Patterns

The AI analysis identified several possible patterns:

- Yellow Flag examples were frequently confused with other categories.
- Some Yellow Flag examples were predicted as Red Flag when the model appeared to focus on negative behavior while overlooking mitigating context.
- Some Yellow Flag examples were predicted as Green Flag when the model may have interpreted a concerning situation as relatively normal.
- Some Green Flag examples were incorrectly classified as Red Flag despite describing healthy boundaries or supportive behavior.
- Some Red Flag examples were incorrectly classified as Green or Yellow despite containing severe behaviors.
- Several incorrect predictions had relatively low confidence, ranging from approximately 0.34 to 0.36.
- Context and severity appeared to be more important than individual keywords in explaining several of the errors.

### My Verification

After reviewing all 10 incorrect predictions myself, I found that the strongest verified pattern was difficulty distinguishing the severity of behavior and interpreting contextual information.

Yellow Flag was involved in 5 of the 10 errors. Three Yellow Flag examples were classified as Red Flag, while two were classified as Green Flag. This suggests that the Yellow Flag category was the most difficult for the fine-tuned model to consistently identify.

I also found that 7 of the 10 errors occurred between neighboring categories: Green/Yellow or Yellow/Red. The remaining 3 errors were between Green and Red, which are farther apart in the intended severity scale.

The low confidence scores were also consistent across the incorrect predictions, with incorrect examples generally receiving confidence scores around 0.34–0.36. This suggests that the model was often uncertain when making these mistakes.

I did not find enough evidence to conclude that sarcasm or post length alone caused the errors. Instead, the clearest pattern was that the model sometimes failed to use contextual details to distinguish mildly concerning behavior from severe behavior, while also occasionally failing to recognize clearly healthy or clearly harmful behavior.

---

## Sample Classifications

The following examples show sample predictions made by the fine-tuned model on the test set.

| Post (truncated) | True Label | Predicted Label | Confidence | Correct? |
|---|---|---|---:|---|
| My partner respects my friendships instead of treating them as competition. | Green Flag | Green Flag | 0.35 | Yes |
| My boyfriend and I have different expectations about how much we should spend on vacations... | Yellow Flag | Yellow Flag | 0.34 | Yes |
| When I set a boundary, my partner accepts it without arguing or trying to change my mind. | Green Flag | Green Flag | 0.35 | Yes |
| My partner has recently become more short-tempered and sometimes blows up over small disag... | Yellow Flag | Red Flag | 0.34 | No |
| My girlfriend said she loved me after only a couple of dates, which made me unsure whether... | Yellow Flag | Red Flag | 0.35 | No |

The first example is reasonably classified as a Green Flag because the partner respects the user's friendships rather than viewing them as competition. This behavior is consistent with the project's definition of a Green Flag as healthy, respectful, and supportive relationship behavior.

### Correct Prediction Explanation

**Example:** "My partner respects my friendships instead of treating them as competition."

**True label:** Green Flag  
**Predicted label:** Green Flag  
**Confidence:** 0.35

The model's prediction was reasonable because the post describes a partner who respects the user's friendships rather than viewing them as a threat or attempting to control them. This behavior is consistent with the Green Flag definition because it demonstrates respect, trust, and healthy relationship boundaries.
---

# Reflection: What the Model Captured vs. What I Intended

The intended classification task was to distinguish relationship situations based on both **behavior and context**, rather than simply identifying positive or negative words.

The baseline model performed well overall, achieving 90% accuracy. Its main weakness was the Green/Yellow boundary, suggesting that it sometimes interpreted mildly concerning situations as normal or healthy.

The fine-tuned model performed differently. Although its validation accuracy increased throughout training, its test accuracy was only 66.7%, which was 23.3 percentage points below the zero-shot baseline.

The fine-tuned model's errors suggest that its learned decision boundary did not consistently reflect the intended severity-based distinctions between the three labels. It sometimes classified clearly positive behavior as Red Flag and clearly harmful behavior as Green Flag.

One important example is the Red Flag example involving repeated threats, insults, and control. Despite containing several behaviors explicitly associated with the Red Flag definition, the model predicted Green Flag.

Another pattern was the tendency to classify some concerning but potentially manageable situations as Red Flag, such as the example involving a partner who becomes short-tempered but apologizes afterward.

Overall, the model appears to have captured some relationship-related patterns, but its decision boundary did not consistently match the intended distinction between healthy behavior, concerning behavior, and seriously harmful behavior.

---

# What Could Be Improved

Based on the error analysis, several changes could potentially improve the classifier:

1. **Add more boundary examples.**  
   More examples distinguishing Green vs. Yellow and Yellow vs. Red could help the model learn the intended boundaries.

2. **Increase diversity within each label.**  
   Green Flag examples should include different types of healthy behavior, including communication, independence, boundaries, trust, and support.

3. **Include more clearly severe Red Flag examples.**  
   Examples involving threats, coercive control, manipulation, and repeated harmful behavior could reinforce the Red Flag category.

4. **Include contextual pairs.**  
   Similar situations with different contextual details could help demonstrate why one example is Yellow while another is Red.

5. **Review annotation consistency.**  
   Similar examples should be checked to make sure they were labeled consistently according to the definitions.

---

# Specification Reflection

The project specification helped guide the implementation by requiring explicit label definitions, a balanced dataset, a baseline comparison, fine-tuning, and error analysis. These requirements encouraged me to define the classification boundaries before training the model and to evaluate the fine-tuned model against the same test set used for the baseline.

One way my implementation diverged from the specification was the model used for the zero-shot baseline. The assignment originally specified:

`meta-llama/llama-4-scout-17b-16e-instruct`

However, that model was no longer available through the Groq API when I ran the project. The API returned a `model_not_found` error. I therefore used:

`openai/gpt-oss-120b`

as the available Groq model instead of changing the assignment's classification task or evaluation process.


### Additional Implementation Divergence

Another implementation change occurred during baseline evaluation. The assignment required the model to output exactly one of the three labels, but the model's responses could vary in capitalization. Initially, the response parser compared the model's output directly against the label names, which caused some valid responses such as "yellow flag" to be treated as unparseable because the expected label was stored as "Yellow Flag."

I modified the parser to compare the responses case-insensitively by converting the model output and label names to lowercase before checking for a match. This change did not alter the classification task or the model's predictions; it only made the evaluation code more robust to capitalization differences in the model's output. After this fix, all 30 baseline responses were successfully parsed and evaluated.

---

# AI Usage

AI tools were used throughout the project as an assistant for planning, dataset development, debugging, and analysis. AI assistance did not replace the final labeling or evaluation decisions.

## AI Use 1: Label and Edge-Case Development

I directed an AI tool to help stress-test the Green Flag, Yellow Flag, and Red Flag definitions and identify situations that might be difficult to classify.

The AI produced possible boundary cases involving issues such as jealousy, friendships with ex-partners, personal space, communication frequency, and controlling behavior.

I reviewed these suggestions and used them to refine the label definitions and anticipated edge cases in `planning.md`. I made the final labeling decisions rather than automatically accepting the AI's classifications.

## AI Use 2: Error Pattern Analysis

I directed an AI tool to examine the fine-tuned model's incorrect predictions and identify possible common patterns across the errors.

The AI was asked to consider factors such as label pairs being confused, ambiguous language, post length, context, and whether certain behaviors were associated with particular predictions.

I agreed that the strongest pattern was difficulty distinguishing severity and using contextual information. Yellow Flag was involved in 5 of the 10 errors, making it the most frequently confused class. I also verified that 7 of the 10 errors occurred between neighboring categories. I did not find enough evidence to conclude that sarcasm or post length alone caused the errors, so I did not treat those as confirmed patterns.

## AI Assistance During Annotation

> I used an AI tool to provide preliminary labels for some examples. These labels were treated as suggestions rather than final decisions. I reviewed the examples myself and changed labels when the AI's classification did not match the project definitions.

---

# Planning and Evaluation Criteria

The project used `planning.md` as a working document for decisions made before and during implementation.

`planning.md` contains:

- Community selection and rationale
- Label definitions
- Anticipated edge cases
- Data collection plan
- Dataset balance
- Evaluation metric reasoning
- Definition of "good enough" performance
- AI tool plan
- Annotation consistency considerations

The README summarizes the final implementation and evaluation, while `planning.md` contains more detailed working notes and design reasoning.

---

# Definition of Success

The original project definition of success was:

- At least **80% accuracy**
- At least **0.75 F1-score for each class**
- At least **0.80 macro F1**
- No class recall below **0.70**
- Most errors should occur between neighboring categories rather than completely unrelated categories

The final fine-tuned model met the following targets:

| Success Target | Final Result | Met? |
|---|---:|:---:|
| Accuracy ≥ 80% | **66.7%** | ❌ No |
| Green Flag F1 ≥ 0.75 | **0.76** | ✅ Yes |
| Yellow Flag F1 ≥ 0.75 | **0.59** | ❌ No |
| Red Flag F1 ≥ 0.75 | **0.64** | ❌ No |
| Macro F1 ≥ 0.80 | **0.66** | ❌ No |
| No class recall below 0.70 | **Yellow Flag recall = 0.50** | ❌ No |
| Most errors between neighboring categories | **7 of 10 errors (70%)** | ✅ Yes |

The zero-shot baseline achieved 90% accuracy and a macro F1 of 0.90 on the test set.

The fine-tuned model achieved 66.7% test accuracy, so it did not meet the 80% accuracy target on this test set. It also did not meet the target F1-score for Yellow Flag or Red Flag, and its macro F1 of 0.66 was below the 0.80 target. Yellow Flag recall was 0.50, which was below the minimum target of 0.70. However, the model did meet the Green Flag F1 target, and 70% of its errors occurred between neighboring categories. Overall, the fine-tuned model did not meet the project's definition of success.

---

# Conclusion

This project demonstrated the difference between zero-shot classification and supervised fine-tuning for a three-class text classification task.

The zero-shot Groq baseline achieved **90.0% accuracy**, while the fine-tuned DistilBERT model achieved **66.7% accuracy** on the same 30-example test set.

Although the fine-tuned model's validation accuracy improved throughout training, this improvement did not transfer to the held-out test set. The error analysis showed that the fine-tuned model struggled with several label boundaries and occasionally made severe classification errors.

The results demonstrate why evaluating a model on a held-out test set and examining individual errors is important. Overall accuracy alone would not show the specific ways in which the model's decision boundary differed from the intended label definitions.

---

# Files

- `planning.md` — project planning, label definitions, edge cases, data collection plan, evaluation criteria, and AI tool plan
- `README.md` — final project documentation and evaluation report
- `relationship_flag_classifier_data.xlsx - dataset.csv` — labeled dataset
- `ai201_project3_takemeter_starter_clean.ipynb` — model training and evaluation notebook
- `confusion_matrix.png` — fine-tuned model confusion matrix
