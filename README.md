# Risk Scoring Model

External enrichment failures are handled gracefully and do not interrupt the analysis process. Missing external data sources do not invalidate the scan and are excluded from risk computation where applicable.

## Package Risk

Each package is evaluated using four normalized risk components, each scaled from 0 to 100:

| Component       | Weight |
| --------------- | -----: |
| Vulnerability   |    45% |
| ML Anomaly      |    20% |
| Maintainer Risk |    20% |
| Typosquatting   |    15% |

## Hard Override

To capture high-confidence attack indicators, a hard override mechanism is applied. If any individual component exceeds a threshold of 70, the package risk is immediately set to 100.

The evaluation follows this priority order:

1. Typosquatting
2. ML Anomaly
3. Maintainer Risk
4. Vulnerability

A component score equal to 70 does not trigger the override.

## Adaptive Risk Computation

If no override condition is triggered, the package risk is computed using an adaptive weighting strategy that considers only active signals.

A signal is considered active only when its value is greater than or equal to 20. This threshold prevents low-intensity noise or weak signals from being amplified during normalization.

Let `S` be the set of components where:


The package risk is calculated as:

```text
Risk = Σ(value_i × normalized_weight_i)
```

where:

```text
normalized_weight_i =
weight_i / Σ(weights of active components)
```

This approach ensures that:

* missing or zero-value signals do not suppress the score;

If no component satisfies the activation threshold, the package risk is defined as zero.

## Risk Levels

|  Score | Level      |
| -----: | ---------- |
|   0–19 | Safe       |
|  20–69 | Suspicious |
| 70–100 | High Risk  |

## Scan-Level Risk Aggregation

The overall project risk is computed using a weighted aggregation strategy that emphasizes the highest-risk dependencies while also accounting for overall package quality and risk distribution.

```text
Overall Risk =
  mean(top 10% highest package scores) × 0.45
  + mean(all package scores) × 0.35
  + exposure score × 0.20
```

The exposure score captures the proportion of risky dependencies within the project:

```text
Exposure Score =
  (number of high-risk packages / total dependencies) × 100
  + (number of suspicious packages / total dependencies) × 40
```

## Health Score

The overall system health score is defined as:

```text
Health Score = 100 - Overall Risk
```

This formulation ensures that higher risk directly reduces the overall health of the dependency ecosystem while prioritizing critical and high-impact risk indicators.
