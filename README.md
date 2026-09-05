# What actually decides an apartment's price in Czechia?

**I scraped around 20,000 real apartment listings from Sreality.cz to see how much of an apartment's price you can explain without ever seeing a place. Answer: about 82%, using size, location and condition. The other 18% is everything a spreadsheet can't capture.**

> If you want to go a little deeper, open the ▸ details sections for the methodology, caveats and code. They are there if you are curious-not because you need to read them.

---
![Map of apartment prices across Czechia](images_and_data/cz_map.png)
*Every dot is a real listing. Brighter means a higher price. Prague and Brno stand out immediately, while the rest of the country is noticeably quieter.*

## The numbers that surprised me most

- **Prague apartments cost 3.7x more** than the cheapest region (Ústecký kraj), 9.9M CZK vs. 2.70M CZK, median.
- **3,000,000 CZK buys you 18 m² in Prague or 68 m² in Most.** Same money, 3.9x the space.
- **Brick beats panel by 41%.** A "cihlová" (brick) apartment costs ~104,074 CZK/m², a "panelová" (Communist-era prefab) one costs ~73,611 CZK/m².
- **"Coming soon" beats "already built".** Off-plan apartments still at the blueprint stage (`Projekt`) sell for more per m² than finished new construction (`Novostavba`), 146,405 vs. 132,337 CZK/m². More on this below. It's the single most counter-intuitive thing in the whole dataset.
- **Distance from Prague isn't a straight line down.** Price drops steadily for the first ~75 km, then ticks back up past 75 km, because that's far enough to start running into Brno, Ostrava and other regional cities with their own price gravity.
- Even within Prague, location swings price 1.8x: Staré Město (225,000 CZK/m²) vs. Černý Most (127,143 CZK/m²).
- A tuned Random Forest explains **~82% of price variance** using only structural + location data, no photos. Typical miss: ~1.2M CZK on a mid-market apartment. Good enough to be interesting, nowhere near appraisal-grade.

---

## Why I built this

I wanted to know what moves apartment prices in Czechia. So I gathered the listings and let them answer three questions:

1. What really drives apartment prices: size, location, condition, or something else entirely?
2. Can a model predict price accurately enough to be useful, using only the kind of information you'd see in a listing?
3. Where does the model break down and what does that tell us about what a spreadsheet can never capture?

---

## Property prices vary widely across regions

![Median price by region](images_and_data/region_median_price.png)

Prague's median listing price (**9.86M CZK**) isn't just the highest in the country, it's **3.7x the cheapest region**, Ústecký kraj (**2.70M CZK**). That's not a gentle gradient, it's a cliff and it happens almost entirely at the Prague border.

### What the same budget actually buys

To make that concrete: here's how far **3,000,000 CZK** stretches, city by city.

![What 3 million CZK buys](images_and_data/3m_buy.png)

In Prague, that budget hypothetically buys an **18 m²** space, barely bigger than a hotel room. In Most, the same money buys **68 m²**, a proper two-bedroom flat.

### Even within Prague, location makes a difference

![Prague neighborhoods, most vs least expensive](images_and_data/prague_neighborhoods.png)

Staré Město (Old Town) runs **225,000 CZK/m²**. Černý Most, on the opposite edge of the city, runs **127,143 CZK/m²**, just over half.

**A correction, worth explaining rather than quietly editing in.** Anuita (the outstanding cooperative-ownership balance on družstevní units) is added to price_czk. An annuity paid down over years isn't quite the same as cash today, but it's more honest than ignoring the obligation entirely. The effect was small but real: Dolní Měcholupy was Prague's cheapest neighborhood. With the true cost included, it's no longer the cheapest at all.

---

## How much does Prague proximity cost?

![How much does Prague proximity cost](images_and_data/prague_proximity.png)

The obvious story is here: apartments 0-5 km from Prague's center go for **181,703 CZK/m²**, apartments 100+ km away go for **81,125 CZK/m²**, a 55% drop. Distance to the capital is the second-strongest predictor in the entire model, right behind size.

But look closer and the curve isn't a straight decline, it **bottoms out around the 50-75 km mark, then rises again** past 75 km. At some point, being far from Prague no longer means being in the middle of nowhere. It can mean being closer to Brno, Ostrava, or another regional center, each with its own pull on local prices. A single "distance to Prague" number can not see that, it only knows about one city.

<details>
<summary><strong>Why this matters for the model, not just the story</strong></summary>

<br>

This is the clearest evidence in the whole project that `dist_to_prague_km` is a genuinely useful but incomplete location signal. A model using distance-to-nearest-major-city would likely pick up more of this pattern. It is on the "what's next" list below.

</details>

---

## Does "Better condition" always mean pricier?

![How much does apartment condition matter](images_and_data/apartment_condition.png)

This is the chart that overturned my own assumption going in. I expected a clean staircase: worse condition = cheapest, `Novostavba` (new construction) = priciest. That's not what the data shows.

**The single most expensive category per m² isn't finished new construction, it's `Projekt`: apartments still being sold off-plan.** At 146,405 CZK/m², off-plan listings out-price apartments currently under construction (`Ve výstavbě`, 140,000 CZK/m²) and completed new builds (`Novostavba`, 132,337 CZK/m²). Meanwhile `Před rekonstrukcí` (needs renovation) is the cheapest surviving category at 67,432 CZK/m², but it's barely behind `Dobrý` (good condition, 68,030 CZK/m²) and both sit well below several "worse-sounding" construction-stage categories.

---

## The brick tax

![Panelák vs Cihla price gap](images_and_data/panelak_cihla.png)

Controlling for nothing else but construction type, brick ("cihlová") buildings command a **41% premium** over prefab panel buildings ("panelová"), 104,074 vs. 73,611 CZK/m². Panel buildings still make up a huge share of Czech housing stock and the market clearly still prices in the difference decades after most of them were built.

**And the single most expensive listing in the whole dataset?** An 87,000,000 CZK, 212 m² apartment in Prague 1's Josefov quarter, roughly 1,720 average Czech monthly salaries, for one flat.

---

## The cleaning files: when the numbers don't make sense

Before any modeling, the raw listings had to survive some genuinely funny quality checks:

- A "68 m² apartment" in **Jirkov** (population ~19,000) was listed on **floor 368**. For context: the AZ Tower in Brno, Czechia's tallest building, has 30 floors total. This one would out-tower it more than 12 times over. Dropped.
- An 82 m² apartment in **Poděbrady** was listed for **98,000 CZK** total, about 1,195 CZK per square meter, roughly the going rate for a decent bicycle, not an apartment. A 77 m² flat in **Tábor** at 100,000 CZK (~1,299 CZK/m²) isn't far behind. Filtered out.
- A listing in **Mladá Boleslav** claimed **5,989 m²** of living space for 9.7M CZK, either a palace at bargain-basement pricing or (far more likely) a typo that turned an apartment into a small stadium. Filtered out.

None of these are edge cases worth modeling, they are data-entry mistakes. But finding them required actually looking at the data with some skepticism, not just checking for `NaN`.

---

## Can a model actually predict the price?

Short answer: yes, mostly. And the "mostly" is the interesting part.

I compared seven models: two non-ML baselines, Linear Regression (raw and log-transformed), a Decision Tree, a Random Forest and Gradient Boosting, each one fairly tuned and evaluated the same way.

![Model comparison](images_and_data/model_comparison.png)
*(Single train/test split, shown for quick visual comparison, the table below reports the more rigorous number.)*

| Model | R² (mean of 15 spatial splits) | Avg. Error (MAPE) |
|---|---|---|
| Naive baseline (region median) | 0.279 | 35.6% |
| Naive baseline (price/m² x region area) | 0.701 | 29.3% |
| Linear Regression | 0.700 | 28.3% |
| Linear Regression (log-price) | 0.745 | 21.4% |
| Decision Tree | 0.766 | 22.4% |
| **Gradient Boosting** | **0.818** | **19.3%** |
| **Random Forest** | **0.823** | **18.8%** |

*(Every model was re-run across 15 independent geographic train/test splits and the mean is reported here, see the methodology note below)*

In plain terms: the best model explains **about 82% of the variation in apartment asking prices**, using size, distance to Prague, room count, condition and a handful of similar structural features. No photos, no great natural light.

MAPE is an average error, not a bound, on a mid-market apartment (~6-7M CZK), the typical miss is around 1.2M CZK and it's worse on expensive Prague listings, better on cheap regional ones. That's also being measured against asking price, not confirmed sale price, since that's what a listing scrape gives you. So: genuinely useful for showing what drives price and for spotting a listing that looks structurally over- or under-priced relative to comparable ones.

### So what actually matters?

![Feature importance](images_and_data/feature_importance.png)

Two features dominate: apartment size and distance to Prague. Exactly how much depends on which model you ask, Gradient Boosting concentrates 79.5% of its decision-making in these two alone. Random Forest spreads more weight elsewhere (57.9% combined), giving more credit to room count, the Prague-region flag and metro access. Either way, everything else is a distant second act.

That's a slightly unglamorous conclusion. It's also exactly what anyone who is apartment-hunted in Czechia already suspected. The model isn't inventing a story, it's confirming one with numbers.

<details>
<summary><strong>For the curious: how this was actually validated</strong></summary>

<br>

Real-estate data has a trap that's easy to fall into: listings cluster geographically (same building, same new-development), so a random train/test split lets a model recognize a location instead of generalizing to it. This project avoids that specifically:

- **Spatially grouped train/test split**, done at the level of a ~2 km grid cell (`GroupShuffleSplit`), so an entire cell is always fully train or fully test, never split between the two.
- **Grouped cross-validation**, every hyperparameter search uses `GroupKFold` on the same grid, so tuning can't benefit from the same leakage the split is designed to prevent.
- **Near-duplicate removal happens before the split, not after.** Listings sharing identical lat/lon/area/price are removed from the full dataset up front, so they can't land on both sides of a split and inflate the score.
- **VIF multicollinearity check**, confirmed `latitude`/`longitude` could safely be replaced with a single `distance_to_prague` feature without destabilizing the linear model.
- **Repeated spatial evaluation (the headline metric)**, every model was re-evaluated across **15 independent 80/20 geographic splits**, reporting mean ± standard deviation.
- **Two escalating baselines**, a region-average baseline is trivial to beat, a region price-per-m² x area baseline is not. The real test is beating the second one.

Full details, code and diagnostics (residuals, coefficient stability, VIF tables) are in the [notebook](house_price_cz.ipynb).

</details>

---

## Where the model struggles

- **Errors grow with price.** The model is proportionally consistent, but on multi-million-CZK Prague listings, an absolute miss of 1M+ CZK is common.
- **~20% of price variation is simply invisible to this data.** No photos, no interior quality, no exact micro-location, no historical price changes.
- **Rare categories destabilize the linear model.** A handful of unusual heating types can swing a linear coefficient by millions of CZK, a small-sample artifact, not a real effect. The tree-based models are far more robust to this.

---

## How far could this realistically go?

Location is the biggest lever left unused. `latitude`/`longitude` are in the dataset already, they're dropped in favor of `distance_to_prague_km` for interpretability. Two ways to spend that signal more aggressively:

- **A comps-style feature**, median price/m² of the *k* nearest listings, instead of one distance-to-Prague number.
- **Distance to nearest major city, not just Prague**, directly addresses the dip-then-rise pattern found above, a Brno-proximity signal would likely explain a real chunk of the residual once you're 75+ km from Prague.
- **Leakage-safe target encoding for `district`/`city`** instead of dropping them for high cardinality.

Beyond that, in rough order of effort-to-payoff:

- **Log-target for the tree models** could tame the errors grow with price pattern directly.
- **Richer features**: nearby schools/parks/noise, building age, waterfront or view, days on market / listing age, floor-within-building context.
- **Photo-derived features**, a model scoring "renovated vs. dated" from listing photos. Most likely to move the needle a lot, also the most work by a wide margin.
- **Permutation importance / SHAP** instead of impurity-based feature importance, for more defensible claims about what matters most.

**Realistic target with all of the above: high-80s to low-90s R².** The hard ceiling isn't modeling technique, it's the data itself: this project predicts asking price, not confirmed sale price. Two identical apartments listed by two different sellers can carry genuinely different asking prices. Getting meaningfully past that ceiling would mean switching to sold-price data with true comparables (tax records, transaction history), which is a different, much larger data problem than anything a listing scrape can solve.

---

## Tools used

- **Python** - pandas & NumPy for cleaning and feature engineering, GeoPandas for the map, Matplotlib for every chart above
- **scikit-learn** - pipelines, `ColumnTransformer`, grouped cross-validation and all seven models
- **statsmodels** - VIF multicollinearity diagnostics
- **Jupyter Notebook** - the full analysis, end to end, in one reproducible place

## Project structure

```
├── images_and_data/
│   ├── *.png                     # chart images used in this README
│   ├── sreality_master.csv       # raw scraped listings
│   └── cz_prices_cleaned.csv     # cleaned dataset used for modeling
├── house_price_cz.ipynb          # full analysis, cleaning > modeling > diagnostics
└── README.md
```

Open the notebook and run it top to bottom every result above, including the repeated spatial evaluation, regenerates from scratch with fixed random seeds.

---

*Data: scraped from [Sreality.cz](https://www.sreality.cz), 2026. Personal identifiers (phone numbers, contact details) were removed before any analysis. This project is for portfolio/educational purposes, not financial or investment advice.*