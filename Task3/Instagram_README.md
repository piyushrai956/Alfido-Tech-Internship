# Instagram Engagement Analysis

Analysis of Instagram posts and engagement data to identify content-type and hashtag patterns, reach signals, and follower-growth trends.

**Notebook:** [`Instagram_Data_Analysis.ipynb`](./Instagram_Data_Analysis.ipynb)

## Dataset

- Source: [Kaggle — bhanupratapbiswas/instgram](https://www.kaggle.com/datasets/bhanupratapbiswas/instgram)
- 7 relational tables: `users` (100), `photos` (257), `comments` (7,488), `likes` (8,782), `follows` (7,623), `tags` (21), `photo_tags` (501)

## Data Quality Note — Read Before Trusting Any "Posting Time" Claim

Every event timestamp in `photos`, `comments`, `likes`, and `follows` is fixed at a **single identical value** across the entire dataset. This was confirmed directly in the notebook (Cell 2):

```
photos.created dat: 1 unique value(s)
comments.created Timestamp: 1 unique value(s)
likes.created time: 1 unique value(s)
follows.created time: 1 unique value(s)
users.created time: 100 unique values (this is the only usable date field)
```

This means a genuine "best time to post" analysis is **not possible** from this data — it's a data-generation artifact, not a gap in the analysis. `users.created time` (account signup) is the only field with real date variation, and is used here as a documented *proxy* for audience growth momentum, not actual posting or engagement time.

Additionally, follower counts are nearly uniform (~76 ± 0.4 per user) in this sample — another synthetic-data artifact — so engagement-rate differences are driven almost entirely by like/comment counts rather than real follower variation.

## Methodology

1. **Engagement metrics** — likes and comments aggregated per photo, joined to each photo's owner's follower count (computed from `follows`, not the unreliable `users.post count` field), giving `engagement_rate = (likes + comments) / followers`.
2. **Content type & hashtags** — engagement compared across `photo type` and filter usage; hashtags resolved via `photo_tags` + `tags` and ranked by both usage frequency and average engagement.
3. **Reach signal** — the `following or not` field on each like records whether the liker already follows the photo's owner, splitting engagement into "from existing followers" vs. "from non-followers."
4. **Growth proxy** — monthly account signups from `users.created time`, the only genuinely time-varying field in the dataset.

## Key Findings

| Metric | Value |
|---|---|
| Photos analyzed | 257 |
| Users analyzed | 100 |
| Engagement rate spread across content types | 0.014 (photo 0.834, video 0.827, carousel 0.820) — within noise |
| Engagement rate spread across filter usage | no filter 0.832 vs. filter 0.828 — within noise |
| Most-used tags | smile (59), beach (42), party (39), fun (38), concert (24) |
| Share of likes from followers | 66.6% |
| Share of likes from non-followers | 33.4% |
| Verified accounts | 8 / 100 |
| Private accounts | 50 / 100 |

**Headline insight:** content-type and filter differences are statistically negligible in this dataset (a spread of ~0.01-0.02 on engagement rate is noise, not signal) — resist the temptation to declare a "best format." The one real, non-noisy finding is that a third of all likes come from people who don't yet follow the account, meaning discovery and hashtag reach matter as much as content quality for the existing audience.

## Visualizations

- Engagement rate by content type and by hashtag (bar charts)
- Most-used tags by frequency vs. highest-engagement tags (bar charts)
- Pie chart — share of likes from followers vs. non-followers
- Monthly account signups (audience growth proxy, line chart)
- Combined dashboard — engagement distribution, top hashtags, content type, and growth trend

## Content Calendar & Recommendations

Because this dataset has no real posting-time variance, the content calendar is built from content-mix and hashtag signals only — **not** a "best hour to post" finding, which this data cannot support. Pair with real Instagram Insights data (which does have genuine timestamps) before finalizing send times.

1. **Invest in discovery, not just retention** — 33.4% of likes already come from non-followers; hashtag reach and Explore-page optimization matter as much as content quality for existing followers.
2. **Anchor hashtag strategy on proven tags** — build content series around `smile`, `beach`, and `party` before testing unproven hashtags.
3. **Don't over-invest in a "best format" theory** — photo, video, and carousel formats are statistically indistinguishable here; mix formats by production cost and message fit instead.
4. **Fix instrumentation before optimizing timing** — this dataset can't answer "when should we post" because event timestamps aren't captured with real variance; real per-post publish and per-engagement timestamps are needed in production analytics before a genuine timing strategy is possible.
5. **Track engagement rate, not raw likes, as the core KPI** — standardizing on engagement rate keeps results comparable across accounts of different sizes once real (non-uniform) follower counts are in play.

## Tech Stack

Python 3.x · Pandas / NumPy · Matplotlib / Seaborn · Jupyter / Colab
