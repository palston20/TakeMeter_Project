# TakeMeter- A Relationship Advice Flag Classifier Planning

## 1. Community

### What community did you choose and why? What makes this discourse varied enough to be interesting?

I chose **r/relationship_advice** because the community is centered around people describing real relationship situations and asking other users how they should interpret or respond to them. This makes it a good fit for a text-classification task because the posts contain a wide range of interpersonal situations, including communication problems, boundaries, jealousy, trust, friendships, dating, family dynamics, and everyday relationship behavior.

The discourse is varied enough to be interesting because two posts can describe superficially similar situations but have very different levels of concern depending on the context. For example, a partner being protective could be a normal expression of care in one situation, a yellow flag if it becomes somewhat controlling, or a red flag if it develops into serious manipulation or isolation. The model therefore needs to learn patterns in the situation rather than simply look for individual keywords.

The classification task will focus on the behavior described in the post, rather than automatically accepting the original Reddit author's interpretation of the situation.

---

## 2. Labels

I will use **three labels**: Green Flag, Yellow Flag, and Red Flag.

### 🟢 Green Flag

**Green Flag:** The situation does not present a significant concern, and the behavior described is generally normal, healthy, respectful, or supportive.

**Example 1:** A partner listens when the other person brings up a concern, communicates openly, and tries to understand their perspective instead of becoming defensive.

**Example 2:** A couple has separate friends and hobbies, respects each other's independence, and communicates clearly about their expectations.

### 🟡 Yellow Flag

**Yellow Flag:** The situation is concerning or unusual, but it is not necessarily a serious problem and may be resolved through communication, additional context, or a reasonable change in behavior.

**Example 1:** A partner is uncomfortable with the other person's friendship with an ex, but they have not tried to control the friendship and are willing to discuss reasonable boundaries.

**Example 2:** Two partners have very different standards for cleanliness and need to discuss how those differences would work if they lived together.

### 🔴 Red Flag

**Red Flag:** The situation involves seriously concerning behavior, such as a major boundary violation, dishonesty, disrespect, manipulation, coercive control, repeated harmful behavior, or behavior that warrants addressing the situation directly.

**Example 1:** A partner secretly checks the other person's private messages after being explicitly told that doing so is not okay.

**Example 2:** A partner repeatedly cheats, lies about the cheating, and refuses to respect agreements about fidelity.

---

## 3. Hard Edge Cases

### What type of post will be genuinely ambiguous between two labels? How will it be handled? ###

The hardest posts will be situations that could reasonably fall between Yellow Flag and Red Flag.

For example, jealousy can exist across both categories. A partner saying, "I feel insecure about your friendship with your ex; can we talk about boundaries?" could be a Yellow Flag because the concern can be discussed respectfully. However, if that jealousy becomes demands to end friendships, monitoring, threats, or isolation, it becomes a Red Flag.

Another difficult boundary is between Green Flags and Yellow Flags. A behavior may be unusual without actually being harmful. For example, partners may have different communication styles, cleanliness standards, or expectations about how much time they spend together. These differences should not automatically be labeled as red flags.

### Annotation Rule for Ambiguous Posts

When I encounter an ambiguous post during annotation, I will:

1. Identify the specific behavior that is being classified.
2. Look for context that changes the severity of the behavior.
3. Determine whether the behavior represents a significant boundary violation, manipulation, dishonesty, disrespect, or harmful pattern.
4. If there is a reasonable explanation and the issue could primarily be resolved through communication, I will generally use **Yellow Flag** rather than Red Flag.
5. If the post does not contain enough information to determine whether the behavior is seriously concerning, I will use **Yellow Flag** rather than assuming the worst.
6. I will record difficult cases and revisit them after reviewing the full dataset to make sure similar situations are labeled consistently.

The goal is to classify the level of concern supported by the information in the post, not to make assumptions about information that is not provided.

---

## 4. Data Collection Plan

### Source

Examples will be collected from **r/relationship_advice**. I will use publicly available Reddit relationship-advice posts as the source material and record the source URL for each example.

The model input will contain the relevant relationship situation. I will remove or avoid information that directly gives away the intended label, such as a title that explicitly says "red flag" or "green flag," when possible.

I will also clean the examples so that unnecessary Reddit formatting, usernames, and other information that does not contribute to the classification task are removed.

### Dataset Size

The final dataset will contain **200 labeled examples**.

My target distribution is:

| Label | Target |
|---|---:|
| Green Flag | 67 |
| Yellow Flag | 67 |
| Red Flag | 66 |
| **Total** | **200** |

This keeps the classes approximately balanced, which is important because a classifier should not be able to achieve high performance simply by predicting the most common label.

### If a Label Is Underrepresented

If one label is underrepresented after collecting 200 candidate examples, I will **not simply duplicate examples** to reach the target.

Instead, I will:

1. Search for additional posts from the same community that fit the underrepresented category.
2. Review the examples using the written label definitions.
3. Continue collecting until the class distribution is approximately balanced.
4. If the community genuinely contains fewer examples that meet a label definition, I will document the imbalance rather than forcing questionable examples into the category.

Before finalizing the dataset, I will also review examples within each category for consistency.

---

## 5. Evaluation Metrics

I will use accuracy, precision, recall, F1-score, and a confusion matrix**.

### Accuracy

Accuracy measures the percentage of predictions that are correct overall.

It is useful as a general measure of model performance, but accuracy alone is not enough because it does not show whether the model performs equally well across Green, Yellow, and Red Flag posts.

### Precision

Precision measures how often the model is correct when it predicts a particular label.

This is important because, for example, I want to know how often posts predicted as **Red Flag** are actually Red Flag examples according to my annotation rules.

### Recall

Recall measures how many of the examples belonging to a particular label the model successfully identifies.

Recall is especially important for **Red Flag** examples because a model that misses many genuinely concerning situations would not be very useful as a relationship-advice classification tool.

### F1-Score

F1-score combines precision and recall into one metric.

I will examine F1-score for each individual class, rather than only looking at one overall score. This will show whether the model performs consistently across the three labels.

### Confusion Matrix

The confusion matrix will show which labels the model confuses with each other.

This is particularly important for this project because I expect the biggest challenge to be distinguishing **Green vs. Yellow** and **Yellow vs. Red**. A confusion matrix can show whether the model is making reasonable neighboring-label mistakes or making more serious mistakes, such as repeatedly classifying Red Flags as Green Flags.

### Primary Evaluation Approach

I will report:

- Overall accuracy
- Precision for Green, Yellow, and Red
- Recall for Green, Yellow, and Red
- F1-score for Green, Yellow, and Red
- Macro-average F1
- Confusion matrix

Because the dataset is designed to be approximately balanced, macro F1 will be particularly useful for checking whether the model performs well across all three labels rather than only performing well on one class.

---

## 6. Definition of Success

The classifier will be considered successful if it demonstrates strong performance across all three classes rather than simply achieving a high overall accuracy.

My target criteria are:

- **At least 80% overall accuracy**
- **At least 0.75 F1-score for each individual label**
- **At least 0.80 macro F1-score**
- No class should have recall below **0.70**
- The confusion matrix should show that most errors occur between neighboring categories, especially Green/Yellow or Yellow/Red, rather than large numbers of Red examples being classified as Green.

These criteria are specific enough to determine objectively whether the model met the project's definition of success.

### What Would Be Good Enough for a Real Community Tool?

For a real community-facing tool, I would require stronger safeguards than I would for this class project. A classifier should not be treated as an authority on whether someone's relationship is healthy or unhealthy. Relationship situations are subjective and often contain incomplete information.

A useful community tool would therefore need:

- Strong performance across all three labels
- Particularly reliable recall for Red Flag situations
- Clear communication that the prediction is a classification, not professional advice
- Human review or the ability for users to challenge the classification
- Continued evaluation on new examples rather than assuming the initial 200 examples represent every relationship situation

For this project, the model will be considered "good enough" for deployment in a hypothetical community tool only if it meets the quantitative criteria above **and** its failure analysis does not reveal a major systematic problem, such as consistently missing serious boundary violations.

---

## 7. AI Tool Plan

AI tools will be used as **supporting tools for dataset development and evaluation**, not as a replacement for my own annotation decisions.

### A. Label Stress-Testing

Before completing the 200-example dataset, I will give an AI tool my three label definitions and my current hard-edge-case rules.

I will ask the AI to generate 5–10 hypothetical posts that sit directly on the boundary between two labels, especially:

- Green Flag vs. Yellow Flag
- Yellow Flag vs. Red Flag

I will independently try to classify each generated example.

If I find that I cannot consistently decide which label an example belongs to, that will indicate that my definitions are too vague. I will revise the definitions and edge-case rules **before completing the final annotation of all 200 examples**.

The purpose of this step is to make the labeling rubric more consistent before it is used on the actual dataset.

### B. Annotation Assistance

I may use an LLM to pre-label a batch of examples as an annotation aid.

If I do this, the AI-generated label will not automatically become the final label. I will review each pre-labeled example and make the final annotation decision using my written rubric.

For disclosure and reproducibility, I will track which examples were AI-pre-labeled by adding an annotation-status field or maintaining a separate record containing:

- Example ID
- AI-generated label
- Final human label
- Whether the human label was changed
- AI tool used

The final dataset will therefore distinguish between AI assistance and my final annotation decisions.

If I decide not to use an LLM for pre-labeling, I will manually annotate the dataset and document that decision instead.

### C. Failure Analysis

After training and evaluating the classifier, I will collect examples where the model's prediction differs from the true label.

I will give the list of incorrect predictions to an AI tool and ask it to identify possible patterns in the errors.

I will specifically look for patterns such as:

- The model confusing Green and Yellow because of mild concerns.
- The model confusing Yellow and Red because of strong emotional language.
- The model relying too heavily on words such as "cheating," "jealous," "love," or "toxic."
- The model missing the importance of context.
- The model performing worse on one label than the others.
- The model incorrectly treating the poster's opinion as the classification instead of the underlying behavior.
- The model missing serious boundary violations when they are described indirectly.

I will not automatically accept the AI's explanation of the errors. I will review the misclassified examples myself and compare the proposed pattern against the actual text, labels, and confusion matrix.

If a pattern appears to be real, I will document it in the final evaluation and explain how it affected the model's performance.

---

## 8. Annotation Consistency and Dataset Quality

Before training the final model, I will review the dataset for:

- Duplicate or nearly duplicate situations
- Missing labels
- Incorrect labels
- Examples where the label is explicitly revealed in the text
- Examples that contain too little information to classify
- Examples that are inconsistent with the written rubric
- Overrepresentation of one type of relationship problem
- Situations where the same behavior was labeled differently without a meaningful contextual difference

The goal is for the dataset to represent the three categories consistently enough that the model can learn the intended distinctions.

---

## 9. Final Goal

The goal of this project is to determine whether a text classifier can learn to distinguish **Green Flag, Yellow Flag, and Red Flag relationship situations from the context of a user's description**.

The main challenge is not simply recognizing words associated with relationships. It is determining whether the model can learn the difference between **normal behavior, concerning but potentially resolvable behavior, and seriously concerning behavior** while handling the ambiguity that naturally occurs in relationship-advice discussions.
