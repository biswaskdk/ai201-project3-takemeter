# TakeMeter: Hacker News Comment Classifier

## Project Overview

TakeMeter is a fine-tuned text classifier that labels Hacker News comments by discourse type. The goal is to classify comments into four labels: `personal_experience`, `technews_explanation`, `opinion_argument`, and `low_substance_reaction`.

This project uses a labeled Hacker News dataset, a zero-shot Groq baseline, and a fine-tuned `distilbert-base-uncased` model. The final result shows that the Groq baseline performed better than the fine-tuned model, but the fine-tuned model still revealed useful failure patterns.

---

## Community

I chose Hacker News because it has active public discussion about technology, programming, AI, startups, science, business, law, and society. Hacker News comments are text-heavy and vary a lot in quality and style.

Some comments explain technical topics. Some share personal experience. Some make arguments. Some are short jokes, reactions, or dismissals. This makes Hacker News a good fit for a discourse classification task.

---

## Label Taxonomy

### `personal_experience`

A comment belongs to `personal_experience` when the writer mainly shares something they personally did, saw, used, built, worked on, or lived through.

Example 1:

> I used AI coding tools at work for six months. They saved time, but I still reviewed every line before merging.

Example 2:

> I even successfully ran an older version of Minecraft on a laptop released in 2003. It was playable with minimum settings and a performance mod.

---

### `technews_explanation`

A comment belongs to `technews_explanation` when it mainly explains a tech-related topic, news item, tool, system, process, term, or factual detail. This includes technical questions and useful corrections. It should not mainly be a personal story or strong opinion.

Example 1:

> ds4 is optimized for systems with unified memory. It works best on Apple Silicon with 96GB+ of RAM.

Example 2:

> Thinking should not be too hard to deal with. Let the model generate freely until it hits a closing think token, then do constrained decoding.

---

### `opinion_argument`

A comment belongs to `opinion_argument` when it makes a claim, judgment, criticism, or stance and supports it with reasoning or context.

Example 1:

> AI code can be useful, but it should not be trusted without review because small mistakes can create serious bugs.

Example 2:

> Companies cannot build software for free just because AI writes some code. They still pay for time, tools, equipment, and review.

---

### `low_substance_reaction`

A comment belongs to `low_substance_reaction` when it is mostly a short reaction, joke, insult, sarcasm, simple dismissal, or vague statement with little useful explanation.

Example 1:

> He’s one of the chief hallucinators, show some respect.

Example 2:

> The founders of Palantir didn’t finish reading the books.

---

## Dataset

The dataset contains public Hacker News comments. I collected comments from multiple Hacker News threads so the dataset would not depend on one topic.

The final dataset has 200 labeled examples in one CSV file:

```text
takemeter_hn_comments_cleaned_final.csv
```

The CSV columns are:

```text
text,label,notes
```

The dataset was not pre-split. The notebook handled the train, validation, and test split automatically using a 70% / 15% / 15% split.

### Label Distribution

| Label                    | Count | Percent |
| ------------------------ | ----: | ------: |
| `opinion_argument`       |    60 |     30% |
| `technews_explanation`   |    54 |     27% |
| `low_substance_reaction` |    46 |     23% |
| `personal_experience`    |    40 |     20% |

No label is above 70% of the dataset. Each label has at least 20% representation.

---

## Labeling Process

I labeled each comment using the definitions from `planning.md`. I removed very short comments that did not have enough meaning to classify clearly, such as one-word replies or vague short reactions.

Some labels were difficult because Hacker News comments often mix explanation, opinion, and personal experience in the same post. When that happened, I labeled the comment based on its main purpose.

---

## Difficult Labeling Examples

### Example 1: Personal experience vs. opinion argument

Comment:

> I used AI coding tools at work for six months. They saved time, but I still think teams are overhyping them because review takes longer than people admit.

Possible labels:

* `personal_experience`
* `opinion_argument`

Decision:

I labeled this type of comment as `opinion_argument` when the main point was the broader claim about AI tools being overhyped. If the comment mainly described the writer’s own experience, I used `personal_experience`.

---

### Example 2: Tech explanation vs. opinion argument

Comment:

> Open models may be cheaper, but they still depend on expensive hardware, quantization, and routing tradeoffs. I do not think they will catch frontier models soon.

Possible labels:

* `technews_explanation`
* `opinion_argument`

Decision:

I labeled this as `opinion_argument` because the technical details support a clear stance. If the main purpose had only been to explain quantization or routing, I would have used `technews_explanation`.

---

### Example 3: Short correction vs. low-substance reaction

Comment:

> The word you’re looking for is “sterilizing.”

Possible labels:

* `technews_explanation`
* `low_substance_reaction`

Decision:

I labeled short comments as `technews_explanation` only when they gave a useful correction, definition, or factual clarification. Very short comments with little meaning were removed from the dataset.

---

## Fine-Tuning Approach

The fine-tuned model used:

```text
distilbert-base-uncased
```

I trained it in Google Colab using the starter notebook. The notebook split the dataset into train, validation, and test sets, tokenized the comments, trained the model, and evaluated it on the locked test set.

### Training Setup

| Setting                     | Value                     |
| --------------------------- | ------------------------- |
| Base model                  | `distilbert-base-uncased` |
| Dataset size                | 200 examples              |
| Train/validation/test split | 70% / 15% / 15%           |
| Epochs                      | 3                         |
| Learning rate               | 2e-5                      |
| Batch size                  | 16                        |
| Test set size               | 30                        |

### Hyperparameter Decision

I kept the default learning rate of `2e-5` and batch size of `16` because the dataset was small and the notebook was designed for this project size. I also tested other epoch counts, but the best result came from the 3-epoch run, so I used that result for the final evaluation.

---

## Baseline

The baseline used Groq’s `llama-3.3-70b-versatile` model in a zero-shot setup. It classified the same test examples as the fine-tuned model.

The prompt gave the model the four label definitions and asked it to output only one label name.

### Baseline Prompt

```text
You are classifying Hacker News comments.
Assign each comment to exactly one of the following labels.

personal_experience:
The writer mainly shares something they personally did, saw, used, built, worked on, or lived through.

technews_explanation:
The comment mainly explains a tech-related topic, news item, tool, system, process, term, or factual detail. This includes technical questions and useful corrections. It should not mainly be a personal story or strong opinion.

opinion_argument:
The comment makes a claim, judgment, criticism, or stance and supports it with reasoning or context.

low_substance_reaction:
The comment is mostly a short reaction, joke, insult, sarcasm, simple dismissal, or vague statement with little useful explanation.

Respond with ONLY the label name.
Do not explain your reasoning.

Valid labels:
personal_experience
technews_explanation
opinion_argument
low_substance_reaction
```

---

## Evaluation Results

### Overall Accuracy

| Model                   | Accuracy |
| ----------------------- | -------: |
| Groq zero-shot baseline |    0.667 |
| Fine-tuned DistilBERT   |    0.433 |

The Groq baseline performed better than the fine-tuned model. The fine-tuned model was 0.233 points lower than the baseline.

---

## Baseline Per-Class Metrics

| Label                    | Precision | Recall |   F1 | Support |
| ------------------------ | --------: | -----: | ---: | ------: |
| `personal_experience`    |      0.67 |   0.67 | 0.67 |       6 |
| `technews_explanation`   |      0.67 |   0.25 | 0.36 |       8 |
| `opinion_argument`       |      0.57 |   0.89 | 0.70 |       9 |
| `low_substance_reaction` |      0.86 |   0.86 | 0.86 |       7 |

The baseline struggled most with `technews_explanation`. It performed best on `low_substance_reaction`.

---

## Fine-Tuned Model Per-Class Metrics

| Label                    | Precision | Recall |   F1 | Support |
| ------------------------ | --------: | -----: | ---: | ------: |
| `personal_experience`    |      0.00 |   0.00 | 0.00 |       6 |
| `technews_explanation`   |      0.56 |   0.62 | 0.59 |       8 |
| `opinion_argument`       |      0.40 |   0.89 | 0.55 |       9 |
| `low_substance_reaction` |      0.00 |   0.00 | 0.00 |       7 |

The fine-tuned model did best on `technews_explanation` and `opinion_argument`. It failed to correctly predict `personal_experience` and `low_substance_reaction`.

---

## Fine-Tuned Confusion Matrix

Rows are true labels. Columns are predicted labels.

| True Label               | Predicted `personal_experience` | Predicted `technews_explanation` | Predicted `opinion_argument` | Predicted `low_substance_reaction` |
| ------------------------ | ------------------------------: | -------------------------------: | ---------------------------: | ---------------------------------: |
| `personal_experience`    |                               0 |                                3 |                            3 |                                  0 |
| `technews_explanation`   |                               0 |                                5 |                            2 |                                  1 |
| `opinion_argument`       |                               0 |                                1 |                            8 |                                  0 |
| `low_substance_reaction` |                               0 |                                0 |                            7 |                                  0 |

The main error pattern was that the fine-tuned model over-predicted `opinion_argument`. Every `low_substance_reaction` example in the test set was predicted as `opinion_argument`.

---

## Wrong Prediction Analysis

### Wrong Example 1

Comment:

> Early SUVs were actually extremely unsafe, high wheel base made them more prone to tipping over, no crumple zones and similar safety measures because we'll just make it big instead, and so on.

True label:

```text
opinion_argument
```

Predicted label:

```text
personal_experience
```

Analysis:

This comment explains a claim about SUV safety. It includes factual details, but the purpose is to argue that early SUVs were unsafe. The model likely focused on the concrete descriptive wording and treated it like an experience-based comment.

---

### Wrong Example 2

Comment:

> I’ll admit, I don’t have perfect pitch. I do however have 2 friends with perfect pitch and a teacher with it.

True label:

```text
personal_experience
```

Predicted label:

```text
technews_explanation
```

Analysis:

This kind of comment sits between personal experience and explanation. The writer shares personal context, but the topic also explains pitch and music ability. The model likely focused on the explanatory topic instead of the writer’s personal framing.

---

### Wrong Example 3

Comment:

> He’s one of the chief hallucinators, show some respect.

True label:

```text
low_substance_reaction
```

Predicted label:

```text
opinion_argument
```

Analysis:

This is a sarcastic reaction with little explanation. The model likely saw it as a judgment and mapped it to `opinion_argument`. This shows that the model struggled to separate short sarcastic reactions from actual arguments.

---

## Sample Classifications

| Comment                                                     | True Label            | Predicted Label       | Confidence | Notes                                                                                  |
| ----------------------------------------------------------- | --------------------- | --------------------- | ---------: | -------------------------------------------------------------------------------------- |
| I’ll admit, I don’t have perfect pitch...                   | `personal_experience` | `personal_experience` |      0.267 | Correct. The comment shares the writer’s own experience with pitch and music.          |
| Working on a project connected through multiple repos...    | `personal_experience` | `personal_experience` |      0.263 | Correct. The writer describes their own project workflow.                              |
| I'm confused; did you reply on the wrong thread...          | `personal_experience` | `personal_experience` |      0.270 | Correct by model output, but confidence is low.                                        |
| Interesting, I was wondering where catching these errors... | `personal_experience` | `personal_experience` |      0.273 | Correct by model output, but confidence is low.                                        |
| Early SUVs were actually extremely unsafe...                | `opinion_argument`    | `personal_experience` |      0.281 | Incorrect. The model confused an explanatory safety argument with personal experience. |

The confidence scores were low, which shows that the fine-tuned model was not very certain in its predictions.

---

## Reflection: What the Model Learned vs. What I Intended

I intended the model to learn the difference between personal experience, tech/news explanation, opinion arguments, and low-substance reactions.

The model partially learned `technews_explanation` and `opinion_argument`, but it did not learn all labels well. It over-predicted `opinion_argument` and failed on `low_substance_reaction`. This suggests that DistilBERT had trouble with subtle discourse boundaries, especially sarcasm and short reactions.

The model may have learned surface cues instead of the full label meaning. For example, if a comment made a judgment, the model often treated it as `opinion_argument`, even when it was actually a short sarcastic reaction.

The Groq baseline performed better because it is a much larger model with stronger general language understanding. The fine-tuned DistilBERT model was trained on only 200 examples, which was not enough for this subjective task.

---

## What I Would Improve

If I continued this project, I would:

1. Collect more examples, especially for `personal_experience` and `low_substance_reaction`.
2. Add more sarcastic and short-reaction examples.
3. Make the boundary between `opinion_argument` and `low_substance_reaction` clearer.
4. Try a larger model than DistilBERT.
5. Test class-weighted loss to help weaker labels.
6. Use a larger test set for more stable metrics.

---

## Spec Reflection

The planning document helped guide the project because it forced me to define the labels before collecting the full dataset. This made the annotation process more consistent.

The implementation diverged from the original plan because the label `technical_explanation` was changed to `technews_explanation`. This better matched the Hacker News data because many comments explained both technology and tech-related news, not only technical systems.

Another change was that I removed very short comments from the dataset. They were hard to label consistently and would have added noise.

---

## AI Usage

I used AI assistance in several parts of this project.

### Label Stress-Testing

I asked AI to generate and review edge cases between labels, such as `personal_experience` vs. `opinion_argument` and `technews_explanation` vs. `opinion_argument`. I used this to sharpen the decision rules in `planning.md`.

### Annotation Assistance

I used AI to help organize and label batches of Hacker News comments into CSV format. I reviewed the labels myself and corrected cases where the suggested label did not match my rules. I also removed auto-labeling notes from the final CSV.

### Failure Analysis

I used AI to help interpret the confusion matrix and model errors. I then checked the error patterns myself. The main pattern I found was that the fine-tuned model over-predicted `opinion_argument` and failed to detect `low_substance_reaction`.

---

## Files in This Repository

```text
planning.md
README.md
takemeter_hn_comments_cleaned_final.csv
evaluation_results.json
confusion_matrix.png
```

---

## Demo Video

Demo video link:

```text
PASTE_DEMO_VIDEO_LINK_HERE
```
