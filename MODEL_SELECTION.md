# Model selection

## Candidates
| Model | Why I tried it |
|---|---|
| logreg_balanced (C=0.5, class_weight=balanced) | Simple, fast, tiny baseline; balanced weights because churners are the minority class |
| random_forest (200 trees, depth 8, balanced) | Non-linear model, may catch interactions |
| gradient_boosting (150 trees, lr 0.05, depth 3) | Usually strong on tabular data |

## Results
| Model | AUC | F1 | F1 weakest slice | p95 ms | Size KB |
|---|---|---|---|---|---|
| logreg_balanced | 0.797 | 0.585 | 0.316 (two-year) | 5.3 | 4 |
| random_forest | 0.755 | 0.532 | 0.429 (one-year/two-year) | 19.6 | 3291 |
| gradient_boosting | 0.759 | 0.396 | 0.250 (one-year) | 5.3 | 206 |

## Decision
I choose logreg_balanced. It has the best AUC and F1 overall, the best F1 on
month-to-month and one-year contracts, the lowest latency (tied) and the smallest size.

## What I accepted losing
Its F1 on two-year contracts (0.316) is lower than random_forest (0.429). I accept this
because that slice is small in the test set, so the number is noisy, and random_forest
is 4x slower and 800x bigger. I will monitor this slice after deployment.

## Promotion rules (for scripts/promote.py)
- Latency budget: 50 ms p95 (CI runners are slower than my laptop; the champion uses 5 ms).
- Minimum F1 gain: 0.01 (smaller differences are noise on a small test set).
- A new model is promoted only if it is within budget AND beats the champion's F1 by the minimum gain.