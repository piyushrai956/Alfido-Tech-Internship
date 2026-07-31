# Zomato Restaurant Data Analysis

Analysis of restaurant and review data to extract insights on ratings, cuisines, location preferences, and the factors that actually drive rating.

**Notebook:** [`Zomato_Analysis.ipynb`](./Zomato_Analysis.ipynb)

## Dataset

- Source: [Kaggle — bhanupratapbiswas/zomato](https://www.kaggle.com/datasets/bhanupratapbiswas/zomato)
- Raw size: 56,252 rows x 13 columns
- Key fields: `name`, `rate`, `votes`, `location`, `cuisines`, `approx_cost(for two people)`, `listed_in(type)`, `online_order`, `book_table`

## Data Quality Note

The source file has broken CSV quote-escaping on a subset of rows — review-like text leaks into other columns, shifting values across the row. This was detected via validity checks on fields with a known small value set (`online_order`, `book_table` in {Yes, No}, `votes` numeric) and those rows were dropped rather than guessed at.

- **4,535 rows (8.1%) dropped** as corrupted
- **51,717 clean rows** used for all analysis

The dataset also has no dedicated free-text review column — the only review-like content survives as the corrupted fragments removed above — so the wordcloud analysis uses `cuisines` and `dish_liked` as the closest available text signal, not actual reviews.

## Methodology

1. **Cleaning** — corruption filtering (above), rating extraction from `"4.1/5"` / `"NEW"` / `"-"` formats, cost-for-two comma-stripping to numeric, text field normalization.
2. **Deduplication** — restaurants can appear multiple times (once per `listed_in(type)`); location/hotspot analysis uses a restaurant-level dedup on `name` + `address` so one popular restaurant isn't overcounted.
3. **Exploration** — cuisine exploded from its multi-value comma-separated field for per-cuisine stats; relationships examined between rating and cuisine, location, cost, votes, online ordering, table booking, and listing type.
4. **Modeling** — a Random Forest Regressor and Linear Regression baseline predict `rating` from votes, cost, location, cuisine, restaurant type, and listing type, to identify which factors actually drive rating rather than just correlate with it.

## Key Findings

| Metric | Value |
|---|---|
| Clean records analyzed | 51,717 / 56,252 (91.9%) |
| Top-rated cuisine (min. 50 listings) | Modern Indian — 4.31 / 5 (145 listings) |
| Lowest-rated common cuisine | Hyderabadi — 3.52 / 5 |
| Most listed cuisines by volume | North Indian, Chinese, South Indian |
| Biggest restaurant hotspot | Whitefield — 631 restaurants |
| Rating vs. cost correlation | r = 0.39 (weak) |
| Rating vs. votes correlation | r = 0.43 (moderate) |
| Random Forest model fit | R² = 0.68 (vs. 0.33 for Linear Regression) |
| Top predictive feature | `votes` — 62% of feature importance |

**Headline insight:** engagement (`votes`) is a far stronger predictor of rating than price or cuisine. The single biggest lever for improving perceived restaurant quality on the platform is driving more customer engagement, not menu curation alone.

## Visualizations

- Top 15 cuisines by average rating (bar chart)
- Location hotspots — restaurant count and average rating by neighborhood (bar charts)
- Correlation heatmap — rating, votes, cost
- Rating distribution by price bucket (boxplot)
- Rating by online-order / table-booking / listing type (boxplots)
- Cuisine x Location average-rating heatmap
- Wordclouds — popular cuisines and most-liked dishes
- Feature importance — top 20 drivers of predicted rating (Random Forest)

## Recommendations

1. **Localized 'hotspot' partnership packages** — prioritize merchant acquisition and sponsored placement in the top restaurant-dense neighborhoods (Whitefield, BTM, HSR, Marathahalli, Electronic City).
2. **Cuisine-based content & curated collections** — build recurring editorial content around top-rated, high-demand cuisines like Modern Indian.
3. **Reward online-ordering & table-booking adoption** — both features correlate with higher ratings; incentivize non-adopting restaurants to enable them.
4. **Price-tier discovery filters, not just price sorting** — since cost only weakly predicts rating, help users find high-rating, low-cost "hidden gem" restaurants.
5. **Fix structured review capture at the data layer** — 8.1% of this raw dataset was unusable due to quoting corruption; a platform should store review text and sub-ratings as clean structured fields from day one.

## Tech Stack

Python 3.x · Pandas / NumPy · scikit-learn · Matplotlib / Seaborn · WordCloud · Jupyter / Colab
