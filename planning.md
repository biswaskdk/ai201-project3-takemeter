# TakeMeter Planning

## Community

I chose Hacker News as my community. Hacker News is a public discussion site where people comment on technology, programming, AI, startups, science, business, law, and society. It is a good fit for this project because the comments are text-heavy and vary in style. Some comments explain technical or news-related topics, some share personal experience, some make arguments, and some are short low-substance reactions.

## Label Taxonomy

### personal_experience

A comment belongs to `personal_experience` when the writer mainly shares something they personally did, saw, used, built, worked on, or lived through.

Example 1:
“I set up an API cost meter on my Claude Code status line and it frequently runs into the hundreds, sometimes thousands of dollars.”

Example 2:
“I even successfully ran an older version of Minecraft on a laptop released in 2003. It was playable with minimum settings and a performance mod.”

### technews_explanation

A comment belongs to `technews_explanation` when it mainly explains a tech-related topic, news item, tool, system, process, term, or factual detail. This can include technical questions or useful corrections. It should not mainly be a personal story or a strong opinion.

Example 1:
“ds4 is optimized for systems with unified memory. It works best on Apple Silicon with 96GB+ of RAM.”

Example 2:
“Thinking shouldn’t be too hard to deal with. Let the model generate freely until it hits a closing think token, then do constrained decoding.”

### opinion_argument

A comment belongs to `opinion_argument` when it makes a claim, judgment, criticism, or stance and supports it with reasoning or context. This label can include tech or non-tech topics if the main purpose is to argue a point.

Example 1:
“AI code can be useful, but it should not be trusted without review because small mistakes can create serious bugs.”

Example 2:
“Companies cannot build software for free just because AI writes some code. They still pay for time, tools, equipment, and review.”

### low_substance_reaction

A comment belongs to `low_substance_reaction` when it is mostly a short reaction, joke, insult, sarcasm, simple dismissal, or vague statement with little useful explanation. This label is for comments that are not clear tech explanations, not personal experiences, and not developed arguments.

Example 1:
“He’s one of the chief hallucinators, show some respect.”

Example 2:
“The founders of Palantir didn’t finish reading the books.”

## Hard Edge Cases

### 1. Personal experience vs. opinion_argument

Some comments include both a personal story and a broader claim. For these, I will look at the main purpose of the comment.

Example:
“I used AI coding tools at work for six months. They saved time, but I still think teams are overhyping them because review takes longer than people admit.”

Possible labels:

* `personal_experience`
* `opinion_argument`

Decision:
I will label this as `opinion_argument` if the main point is the broader claim that AI tools are overhyped. If the comment mainly describes what the person did or experienced, I will label it `personal_experience`.

### 2. Tech explanation vs. opinion_argument

Some comments explain a technical topic but also use that explanation to support a judgment. This was common in the Hacker News data.

Example:
“Open models may be cheaper, but they still depend on expensive hardware, quantization, and routing tradeoffs. I don’t think they will catch frontier models soon.”

Possible labels:

* `technews_explanation`
* `opinion_argument`

Decision:
I will label this as `opinion_argument` if the technical details support a clear stance. I will label it `technews_explanation` only when the main purpose is to explain how something works.

### 3. Personal experience vs. technews_explanation

Some comments explain a technical topic through the writer’s own experience.

Example:
“At my last job, we used Tree-sitter because it gave us syntax trees instead of plain text matching, which made code search more reliable.”

Possible labels:

* `personal_experience`
* `technews_explanation`

Decision:
I will label this as `technews_explanation` if the comment mainly explains the tool or technical idea. I will label it `personal_experience` if the personal story is the main point.

### 4. Low-substance reaction vs. opinion_argument

Some comments make a strong claim but do not explain it much.

Example:
“So a slumlord.”

Possible labels:

* `low_substance_reaction`
* `opinion_argument`

Decision:
I will label this as `low_substance_reaction` because it is mostly a short judgment without reasoning. If the comment gives a reason or context for the claim, I will label it `opinion_argument`.

### 5. Short factual correction vs. low_substance_reaction

Some comments are short but still useful.

Example:
“The word you’re looking for is ‘sterilizing.’”

Possible labels:

* `technews_explanation`
* `low_substance_reaction`

Decision:
I will label short comments as `technews_explanation` only if they give a useful correction, definition, or factual clarification. Very short comments with little meaning, like “Correct” or “That would be interesting,” will be removed from the dataset.

### 6. Non-tech discussion in Hacker News

Hacker News includes many comments about housing, politics, law, health, and society. These are still part of the community, but they may not fit `technews_explanation`.

Example:
“Public records should be public. Charging for them is just a tax on knowing the law.”

Possible labels:

* `opinion_argument`
* `low_substance_reaction`

Decision:
I will label this as `opinion_argument` because it makes a clear claim and gives a reason. I will not label non-tech comments as `technews_explanation` unless they explain a technical or news-related system.

## Data Collection Plan

I will collect at least 200 public comments from Hacker News. I will use comments from different threads so the dataset is not based on only one topic. I will remove very short comments that do not have enough meaning to label clearly.

The final dataset will be saved as one CSV file with these columns:

```csv
text,label,notes
```

I will aim for each label to have at least 20% of the dataset. For a 200-row dataset, that means each label should have at least 40 examples. My final dataset has 220 rows:

| Label                  | Count | Percent |
| ---------------------- | ----: | ------: |
| opinion_argument       |    64 |   29.1% |
| technews_explanation   |    63 |   28.6% |
| low_substance_reaction |    49 |   22.3% |
| personal_experience    |    44 |   20.0% |

## Evaluation Metrics

I will use overall accuracy to measure how often the model predicts the correct label. I will also use per-class F1 scores because some labels may be harder than others. For example, `technews_explanation` and `opinion_argument` may be confused when a comment includes both factual detail and a strong claim.

I will also use a confusion matrix to see which labels the model mixes up most often. This will help me understand whether the model learned the label boundaries or only learned surface patterns.

## Definition of Success

A successful model should perform better than the zero-shot Groq baseline on the same test set. I would consider the model useful if it reaches about 70% accuracy or higher and has reasonable F1 scores across all labels.

The model should not only perform well on one common label. Since all labels have at least 20% representation, the model should learn each class instead of mostly predicting the majority class.

For a real community tool, the classifier would not need to be perfect, but it should be good enough to show useful patterns in Hacker News discussion. If it often confuses `technews_explanation` with `opinion_argument`, I would not consider it ready without more data or clearer labels.

## AI Tool Plan

### Label Stress-Testing

I will use an AI tool to test my label definitions before finalizing the dataset. I will ask it to generate comments that sit between two labels, such as `technews_explanation` vs. `opinion_argument` or `personal_experience` vs. `opinion_argument`. If the examples are hard to classify, I will update my decision rules.

### Annotation Assistance

I may use an AI tool to suggest labels for some comments, but I will review every label myself. I will not accept labels without reading the comment. If I use AI pre-labeling, I will mention this in the README.

### Failure Analysis

After training, I will give the model’s wrong predictions to an AI tool and ask it to find patterns. I will check those patterns myself by rereading the examples. I will only include patterns in my final report if I agree that they are supported by the actual mistakes.
