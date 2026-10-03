# Cyberbullying Pattern Detector

Detecting harassment as a **trajectory across a conversation**, not as a single toxic sentence.

> Online harassment is often gradual. Each message may look mild on its own, but the sequence tells a different story. This project models conversation evolution to catch escalation early, with confidence scores and human review built in.

**Status:** 🚧 In progress  |  **Demo:** _link to Hugging Face Space / video_  |  **Write-up:** _link to blog post_

---

## Table of Contents

1. [Motivation](#motivation)
2. [Example](#example)
3. [Pattern Labels](#pattern-labels)
4. [Approach](#approach)
5. [Data](#data)
6. [Results](#results)
7. [Safeguards and Ethics](#safeguards-and-ethics)
8. [Project Structure](#project-structure)
9. [Getting Started](#getting-started)
10. [Limitations](#limitations)
11. [Roadmap](#roadmap)
12. [References and Acknowledgements](#references-and-acknowledgements)

---

## Motivation

Most toxicity classifiers (e.g., Perspective API, Detoxify) score messages one at a time. That misses patterns like:

- gradual escalation over several messages
- repeated targeting of one person
- social exclusion and pile-ons

This project asks: **can modeling the sequence catch harassment earlier and more accurately than per-message scoring?**

## Example

```
Message 1: "You're annoying."
Message 2: "Nobody likes you."
Message 3: "Why are you even here?"
Message 4: "Everyone should ignore her."
```

**System output:**

```
Pattern detected: Escalating hostile interaction
Confidence: 0.87  →  Route: human review

Indicators:
  • Repeated targeting
  • Personal attacks
  • Social exclusion
  • Increasing hostility
```

## Pattern Labels

Multi-label classification; a thread can have more than one.

| Label | Definition |
|---|---|
| `escalating_hostility` | Hostility intensity increases over the thread |
| `repeated_targeting` | The same person is the target of attacks across multiple messages |
| `social_exclusion` | Messages aim to isolate, ignore, or expel the target |
| `pile_on` | Multiple authors direct hostility at one target |

_Full annotation guidelines: [`docs/label_spec.md`](docs/label_spec.md)_

## Approach

Four methods are compared on the same test set:

| # | Method | Description |
|---|---|---|
| 1 | **Per-message baseline** | Pretrained toxicity model; flag a thread if any message exceeds a threshold |
| 2 | **Temporal features + GBM** | Toxicity slope, max consecutive jump, second-person pronoun count, repeated-target count, length trend, fed to logistic regression / gradient boosting |
| 3 | **Prompted LLM** | Whole thread plus label definitions, structured JSON output (labels, indicators, confidence) |
| 4 | **Sequence model** | Fine-tuned transformer encodes each message; GRU / small Transformer encoder predicts thread-level labels |

Additional ablation: concatenating the full thread into a single transformer input.

**Training details:** class weighting or focal loss for imbalance, 3 random seeds (mean ± std reported), temperature scaling for calibration.

## Data

| Source | Use | Notes |
|---|---|---|
| Conversations Gone Awry (ConvoKit) | Thread-level derailment | Wikipedia talk pages / Reddit CMV |
| Jigsaw Toxic Comment / Civil Comments | Per-message toxicity scorer | Single-message labels |
| Public cyberbullying datasets | Supplementary | Quality-checked before use |
| Synthetic threads (LLM-generated) | Pattern coverage and hard negatives | Hand-reviewed sample; **never** used as the only test data |
| Hand-labeled gold set | Final evaluation | ___ threads, labeled by me |

**Unified format:**

```json
{
  "thread_id": "t_0001",
  "messages": [
    {"author": "a1", "target": "a2", "text": "...", "timestamp": "..."}
  ],
  "labels": ["escalating_hostility", "social_exclusion"]
}
```

**Splits are made by thread, not by message,** to prevent leakage. The gold test set contains real (non-synthetic) data only.

## Results

_Fill in after experiments._

| Method | Macro-F1 | PR-AUC | Precision | Recall | Mean detection point (msg #) |
|---|---|---|---|---|---|
| Per-message baseline | | | | | |
| Temporal features + GBM | | | | | |
| Prompted LLM | | | | | |
| Sequence model (GRU) | | | | | |

**Early detection:** average message index at which each model first flags a thread that is later labeled harassment.

**Calibration:** reliability diagram and ECE before and after temperature scaling in [`notebooks/calibration.ipynb`](notebooks/calibration.ipynb).

**Review queue tiers:**

| Tier | Confidence | % of threads | Precision |
|---|---|---|---|
| Auto-flag | ≥ __ | | |
| Human review | __ – __ | | |
| Ignore | < __ | | |

### Error Analysis

Manual review of 50+ false positives and false negatives, categorized:

| Category | Count | Example |
|---|---|---|
| Friendly banter | | |
| Sarcasm | | |
| Quoting someone else's insult | | |
| Reclaimed language | | |
| Missed implicit exclusion | | |

### Subgroup Check

False-positive rates for threads containing dialect features (e.g., AAE) or identity terms, compared to the overall rate: _results here_.

## Safeguards and Ethics

- **Decision support, not automated punishment.** Outputs are meant for a human reviewer.
- **Confidence-aware routing.** Low-confidence cases are never auto-flagged.
- **False-positive feedback.** The demo includes a "mark as false positive" control to show how reviewer feedback could be collected.
- **Bias evaluation.** Dialect and identity-term subgroup analysis is reported above.
- **Privacy.** Only public or research datasets are used. No private messages are collected, stored, or required.
- **Misuse risk.** Tools like this could be repurposed for surveillance or censorship. Any real deployment would need consent, transparency, data-retention limits, and an appeals process.

## Project Structure

```
.
├── data/
│   ├── raw/
│   ├── processed/
│   └── gold/              # hand-labeled test set
├── src/
│   ├── data/              # loading, unification, splitting
│   ├── features/          # temporal feature extraction
│   ├── models/            # baseline, GBM, LLM prompt, GRU
│   └── eval/              # metrics, calibration, early detection
├── notebooks/
├── app/                   # Streamlit / Gradio demo
├── docs/
│   └── label_spec.md
├── requirements.txt
└── README.md
```

## Getting Started

```bash
# Clone and install
git clone https://github.com/<your-username>/cyberbullying-pattern-detector.git
cd cyberbullying-pattern-detector
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Prepare data
python -m src.data.prepare

# Train / evaluate
python -m src.models.train --model gbm
python -m src.models.train --model gru --seeds 3
python -m src.eval.run_all

# Launch demo
streamlit run app/app.py
```

## Limitations

- Conversation-level labels are scarce, and part of the training data is synthetic, which may not reflect real harassment.
- Performance on sarcasm, in-group banter, and non-English text is limited.
- The gold set is small, so confidence intervals are wide.
- Labels reflect one annotator's judgment; no inter-annotator agreement yet.
- Platform- and culture-specific norms are not modeled.

## Roadmap

- [ ] Second annotator and inter-annotator agreement
- [ ] Multilingual support
- [ ] Author-relationship features (friends vs. strangers)
- [ ] Reviewer feedback loop for active learning
- [ ] Hosted demo

## References and Acknowledgements

- Zhang et al., *Conversations Gone Awry: Detecting Early Signs of Conversational Failure* (ACL 2018)
- ConvoKit: https://convokit.cornell.edu
- Jigsaw Toxic Comment Classification Challenge
- Detoxify: https://github.com/unitaryai/detoxify
- _Add any additional papers, datasets, and tools you use_

## License

MIT (or your choice). Check the licenses of all datasets before redistributing any data.
