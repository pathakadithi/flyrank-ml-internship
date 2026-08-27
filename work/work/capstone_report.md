# Capstone Report — Content Opportunity Scoring for Search-Driven Content Review

- **Author:** Pathak Adithi
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/pathakadithi/flyrank-ml-internship
- **Date:** 2026-08-27

---

## 0. Abstract

This research asks whether observable search-performance signals can be used to prioritize pages for content review when review capacity is limited. The analysis uses the anonymized FlyRank warehouse release and five historical features covering impressions, clicks, CTR, and search position. A Logistic Regression model was trained to approximate the Week-4 baseline-defined opportunity policy and evaluated using both an 80/20 random stratified split and a client-grouped validation check. The model achieved an F1 score of 0.6141 under the random split and 0.6095 under client-grouped validation, with recall close to 1.0 in both checks. These results provide directional evidence that a learned scoring approach can approximate a rule-based content-opportunity policy, but they do not prove future traffic recovery, causal refresh impact, or Google's ranking behavior.

---

## 1. Problem Framing

### Research Question

Can observable search-performance signals be used to prioritize pages for content review when review capacity is limited?

### Decision Supported

The project supports a content-review prioritization decision.

The intended workflow is to use historical search-performance signals to identify pages that may deserve review first when a content team has limited review capacity.

### Unit of Analysis

The primary unit of analysis is a content item / page represented through aggregated historical search-performance signals.

### Output

The workflow produces a baseline-defined opportunity score and a ranked review queue.

The ranked queue includes:

- Rank
- Client pseudonymous identifier
- Content pseudonymous identifier
- Query pseudonymous identifier
- Baseline opportunity score
- Reason code
- Recommended action

Pseudonymous identifiers are used for grouping or internal analysis and are not treated as public-facing identifying information.

### Human Action

A human reviewer uses the ranked output to decide which pages should be inspected first.

The output is intended to prioritize review effort. It is not intended to automatically refresh, rewrite, merge, prune, or otherwise modify content.

### Cost of a Wrong Call

A false positive can consume limited editorial review capacity on a page that does not match the baseline-defined opportunity condition.

A false negative can cause a page that might have been useful to review to be missed or reviewed later.

Because the model has very high recall but lower precision, the resulting queue should be treated as a candidate list for human review.

### Why Data / ML Helps

A rule-based baseline provides a transparent starting point for identifying opportunities.

Machine learning can combine multiple historical signals into a repeatable scoring approach that can approximate the baseline policy across a large number of pages.

The purpose is therefore not to replace editorial judgment, but to make prioritization more systematic and repeatable.

---

## 2. Data Safety

### Data Release

This capstone uses the anonymized FlyRank warehouse release:

`flyrank_pseudonymized_warehouse_release_v20260703`

The analysis uses the following warehouse tables:

- `fact_content_daily_performance`
- `fact_content_query_90d`
- `dim_content`
- `dim_clients`

The daily performance table contains 78,835,655 rows.

The query table contains 2,414,248 rows.

The release contains 519,606 content items.

The available daily performance window used for this analysis is:

**2025-01-27 through 2026-06-30**

### Data Used

The model uses historical search-performance signals derived from the warehouse data.

The final model feature list is:

- `impressions_90d`
- `clicks_90d`
- `ctr`
- `avg_position_90d`
- `avg_position_last30`

### Deliberately Excluded Information

The analysis does not publish or use client-identifying information such as:

- Client names
- Domains
- Public URLs
- Private search queries
- Credentials
- Raw warehouse exports

Pseudonymous IDs are not treated as predictive features.

They may be retained for grouping or internal analysis where necessary, but they are not used by the Logistic Regression model as input features.

### Leakage Risks Considered

Potential leakage risks were considered during the validation and research-claim audit.

In particular:

- Label-derived fields such as `trend_direction` and `trend_pct` were not used as model features.
- Pseudonymous IDs were not used as predictive features.
- Future refresh outcomes were not used as features.
- Client-identifying information was not used as model input.
- Private queries and credentials were not published.
- The model features are restricted to historical search-performance signals.

The target is derived from the Week-4 baseline, so the model is explicitly interpreted as learning the baseline-defined policy rather than predicting an independently observed future outcome.

### Public Safety

The public-facing paper and repository outputs intentionally avoid publishing client names, domains, private queries, credentials, or raw data exports.

---

## 3. Baseline

### Week-4 Baseline

The Week-4 work established a transparent rule-based content-opportunity baseline.

The baseline combines observable historical search-performance signals, including:

- Search visibility
- CTR relative to position
- Recent CTR change
- Search position
- Historical performance

The baseline produces one `baseline_score`.

The baseline action is defined as:

```text
baseline_score > 0  → refresh
baseline_score <= 0 → no_action
