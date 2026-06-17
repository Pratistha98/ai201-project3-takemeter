# TakeMeter — FIFA World Cup Discourse Classifier

A fine-tuned text classifier that evaluates discourse quality in FIFA World Cup fan communities. Given a tweet, the model classifies it as `analysis`, `hot_take`, or `reaction`.

---

## Community Choice

I chose the FIFA World Cup 2022 Twitter community (sourced from a public Kaggle dataset). This community is a strong fit for a classification task because posts range from pure emotional reactions to detailed tactical analysis, making the discourse varied and interesting to classify. World Cup discussions are intense, opinionated, and cover a wide spectrum of post types — the distinction between a hot take and real analysis genuinely matters to fans in this space.

---

## Label Taxonomy

| Label | Definition |
|---|---|
| `analysis` | A structured point backed by stats, tactics, or historical comparison. Evidence is specific and supports the claim. |
| `hot_take` | A bold opinion stated confidently with no real supporting evidence or argument. |
| `reaction` | An immediate emotional response to a match or moment, with little to no argument. |

**Examples per label:**

**analysis:**
- "Argentina's high press in the second half forced 3 turnovers that led directly to goals."
- "Morocco conceded only 1 open play goal in 6 games. Their defensive block was historically good."

**hot_take:**
- "Mbappe is overrated, he disappeared when it mattered most."
- "France would have won if Deschamps wasn't so defensive minded."

**reaction:**
- "I CANT BELIEVE WE LOST THAT MATCH OMG"
- "That penalty miss broke my heart. I can't."

---

## Data Collection

**Source:** FIFA World Cup 2022 Tweets — public Kaggle dataset (22,524 tweets)

**Labeling process:**
1. Downloaded the public Kaggle dataset
2. Filtered tweets: removed links, kept length between 20–300 characters, removed duplicates
3. Sampled 250 tweets
4. Used Groq (llama-3.3-70b-versatile) to pre-label each tweet using my label definitions
5. Reviewed the label distribution and spot-checked examples for accuracy
6. Analysis examples were AI-generated using Groq because Twitter's character limit makes real analytical posts rare
7. Added 4 additional hot_take examples to reach 200 total

**Label distribution:**

| Label | Count |
|---|---|
| `analysis` | 70 |
| `reaction` | 70 |
| `hot_take` | 60 |
| **Total** | **200** |

**Three difficult-to-label examples:**

1. *"Mbappe only scored 8 goals and France still lost — he's overrated."*
   → Decided: `hot_take`. The stat is cherry-picked to decorate an opinion, not build a real argument. Removing the opinion leaves nothing that proves the claim independently.

2. *"France relied too heavily on Mbappe. No depth in attack when he was marked."*
   → Decided: `hot_take`. Makes a point but provides no evidence — no stats, no tactical specifics. It asserts rather than argues.

3. *"Morocco proved African football has arrived. 4 clean sheets in 5 games is historic."*
   → Decided: `analysis`. Includes a specific verifiable stat that directly supports the claim.

---

## Fine-Tuning Approach

- **Base model:** distilbert-base-uncased (HuggingFace, open source)
- **Training setup:** Google Colab free T4 GPU
- **Epochs:** 3
- **Learning rate:** 2e-5
- **Batch size:** 16
- **Split:** 70% train / 15% validation / 15% test

**Key hyperparameter decision:** Used 3 epochs to avoid overfitting on a small 200-example dataset. More epochs would cause the model to memorize training examples rather than learn general patterns.

---

## Baseline

**Model:** Groq llama-3.3-70b-versatile (zero-shot, no fine-tuning)

**Prompt used:**
```
You are classifying FIFA World Cup tweets from football fans.
Assign each tweet to exactly one of the following categories.

hot_take: A bold, confident opinion stated without supporting evidence or stats.
Example: "Messi is the GOAT, no more debate needed."

analysis: A structured point backed by stats, tactics, historical comparison, or detailed reasoning.
Example: "Argentina's high press won them 70% of midfield duels in the final."

reaction: An immediate emotional response to a match or moment.
Example: "GOAAAAAL I CANT BELIEVE THIS IS HAPPENING"

Respond with ONLY the label name. Do not explain your reasoning.
Valid labels: hot_take, analysis, reaction
```

**How results were collected:** Each test example was sent to the Groq API individually. Responses were parsed and matched against the label map. All 30 test examples were parseable.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq) | 90% |
| Fine-tuned DistilBERT | 80% |

### Per-Class Metrics — Baseline (Groq)

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.92 | 1.00 | 0.96 | 11 |
| hot_take | 0.82 | 1.00 | 0.90 | 9 |
| reaction | 1.00 | 0.70 | 0.82 | 10 |

### Confusion Matrix — Fine-Tuned Model

| | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: analysis** | 11 | 0 | 0 |
| **True: hot_take** | 0 | 0 | 9 |
| **True: reaction** | 0 | 0 | 10 |

### Three Wrong Predictions Analyzed

**1.** Tweet: *"France would have won if Deschamps wasn't so defensive minded"*
- True label: `hot_take` | Predicted: `reaction`
- Why it failed: On Twitter, hot_takes are short and casual — they look identical to reactions. The model never learned the hot_take boundary because both share the same short, opinionated tone.

**2.** Tweet: *"Ronaldo was invisible this entire tournament, Portugal deserved better"*
- True label: `hot_take` | Predicted: `reaction`
- Why it failed: The post expresses frustration, which resembles a reaction. Without explicit stats or argument structure, the model defaulted to reaction every time.

**3.** Tweet: *"Messi only wins because he has a better team around him"*
- True label: `hot_take` | Predicted: `reaction`
- Why it failed: This is a systematic failure — the model predicted zero hot_takes across the entire test set. It learned two classes well (analysis and reaction) but never learned hot_take as a distinct category.

### Sample Classifications

| Tweet | Predicted | Confidence | Correct? |
|---|---|---|---|
| "Argentina's midfield averaged 140 passes per game, highest in the tournament" | analysis | 94% | ✅ |
| "GOAAAAAL MBAPPE ARE YOU KIDDING ME THIS IS INSANE" | reaction | 97% | ✅ |
| "Messi is finally the GOAT no more debates" | reaction | 81% | ❌ (true: hot_take) |
| "Morocco's 4-4-2 low block allowed only 1.2 xG per game across 6 matches" | analysis | 91% | ✅ |
| "England will never win anything with this squad" | reaction | 78% | ❌ (true: hot_take) |

The analysis prediction is reasonable — the tweet contains a specific stat and a comparative claim, which are clear markers of the analysis label as defined.

---

## Reflection: What the Model Learned vs. What I Intended

I intended the model to distinguish three types of discourse: structured argument (analysis), unsupported opinion (hot_take), and emotional reaction (reaction). What it actually learned was a simpler two-way distinction: **formal sentences with numbers = analysis, everything else = reaction.**

The model never learned `hot_take` as a separate category. This happened for two reasons. First, Twitter is a poor source for hot_takes — they are short and emotional, making them look identical to reactions at the surface level. Second, the analysis examples were AI-generated and consistently formal, making that class easy to learn while the hot_take/reaction boundary remained blurry. The model learned surface features (length, formality, presence of numbers) rather than the intent behind the post.

---

## Spec Reflection

**One way the spec helped:** The spec's emphasis on defining labels precisely before collecting data forced me to write clear definitions early. This made the Groq pre-labeling prompt much more consistent and reduced ambiguous labels.

**One way implementation diverged:** The spec assumes you can collect 200 real posts from your community. In practice, Twitter's API restrictions made scraping unreliable, and the natural distribution of tweets heavily favored reactions over analysis. I used AI-generated examples to balance the dataset — a deviation that likely hurt the model's ability to generalize to real analysis tweets.

---

## AI Usage

1. **Pre-labeling:** Used Groq (llama-3.3-70b-versatile) to pre-label 250 scraped tweets using my label definitions from planning.md. I reviewed the distribution and spot-checked examples. I did not review every single label individually — this is a limitation and likely introduced some label noise.

2. **Analysis data generation:** Prompted Groq to generate 70 realistic analysis-style World Cup tweets with stats and tactical observations to balance the dataset. In hindsight, this created a mismatch — real analysis tweets don't look as clean and formal as AI-generated ones, which may have made the analysis class artificially easy to learn.

3. **Failure pattern analysis:** After reviewing the confusion matrix, I identified the systematic pattern that the model predicted zero hot_takes — classifying all of them as reactions. This confirmed the hot_take/reaction boundary was never learned due to surface similarity in tweet format.
