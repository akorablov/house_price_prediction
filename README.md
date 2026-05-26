## Overview

This project walks through that end-to-end process using a dataset of 4140 residential property sales. I trained and compared three regression models: Linear Regression, Decision Tree, and Random Forest, to estimate sale prices from structural features like living area, number of rooms, condition, and waterfront status. The goal wasn't just to get a prediction out, but to understand how well these features actually explain price, where each model struggles and what that reveals about real estate pricing in practice.

## The Questions

1. Can house prices be predicted using a limited set of basic structural features?
2. How does a simple linear model hold up against more complex tree-based approaches?
3. What do the residuals and diagnostics tell us about where predictions break down?

## Tools Used

- **Python** - pandas for data wrangling, matplotlib for visualisation, NumPy for numerical work, SciPy for statistical diagnostics, and scikit-learn for all the modeling
- **Jupyter Notebooks** - for keeping the analysis and code in one readable place
- **Visual Studio Code** - for writing and testing scripts
- **Git & GitHub** - for version control and sharing

## Data Cleaning

Running a few basic checks turned up two problems I wouldn't have caught otherwise.

**Zero-price records:** 49 records (1.2% of the dataset) had a sale price of $0, which are almost certainly data entry errors. These were removed before modeling to avoid distorting results.

**Missing condition values:** The `condition` column contained 545 missing entries, which I filled using the median. I chose the median over the mean because it's less sensitive to outliers, which felt like the right call for a skewed variable like property condition.

```python
df = df[df['price'] > 0]
df['condition'] = df['condition'].fillna(df['condition'].median())
```

View the full notebook here: [house_price_prediction_improved.ipynb](house_price_prediction_improved.ipynb)

## The Analysis

The feature set includes everything structural the dataset offers: `sqft_living`, `bedrooms`, `bathrooms`, `floors`, `condition`, `yr_built`, `yr_renovated`, `view` and `waterfront`. The last one, a simple yes/no flag for whether a property is on the waterfront, turns out to matter quite a lot, as we'll see.

```python
FEATURES = ['sqft_living', 'bedrooms', 'bathrooms', 'floors',
            'condition', 'yr_built', 'yr_renovated', 'view', 'waterfront']
X = df[FEATURES]
y = df['price']
```

Data was split 80/20 into training and test sets. Three models were trained, but importantly, the Decision Tree and Random Forest weren't just run on default settings. Without constraints, tree models will memorise the training data and fall apart on anything new. So both were regularised with `max_depth` and `min_samples_leaf` limits to keep them honest. All three models were also evaluated with 5-fold cross-validation, not just a single test split.

```python
models = {
    'Linear Regression': LinearRegression(),
    'Decision Tree': DecisionTreeRegressor(max_depth=6, min_samples_leaf=10, random_state=42),
    'Random Forest': RandomForestRegressor(n_estimators=200, max_depth=12, min_samples_leaf=4, random_state=42),
}
```

## Results

| Model | Test R² | CV R² (mean) | MAE | RMSE |
|---|---|---|---|---|
| Linear Regression | 0.55 | 0.47 | $160,508 | $267,982 |
| Decision Tree | 0.25 | 0.28 | $183,162 | $345,011 |
| Random Forest | 0.31 | 0.35 | $178,301 | $330,757 |

Linear Regression came out on top, which might seem surprising, but it makes sense here. With a relatively small and mostly linear feature set, the added complexity of tree-based models doesn't buy much. They can't find structure that isn't there.

The model explains about 55% of the variation in sale prices, with an average prediction error around $160,000. That's reasonable for a structural-features-only model, but it's also a clear signal that price is being driven by things we're not measuring - location above all else.

The cross-validation R² (0.47) is worth noting too. It's lower than the test score (0.55), which means the test split was slightly kind. The CV number is the more honest estimate of how this would perform on genuinely new data.

![actual_vs_prdicted_house_prices.png](images/actual_vs_prdicted_house_prices.png)

Looking at the residuals, there's a clear fan-shaped pattern, errors stay small for cheaper homes but blow out for expensive ones. This is heteroscedasticity and it shows up in the Q-Q plot too, where the tails deviate from the normal line. In plain terms: the model is fairly reliable in the middle of the market, but loses confidence at the high end where premium factors like views, neighbourhood prestige, and lot quality start dominating the price.

## Insights

**On predicting price from structure alone:** it works, up to a point. Fifty-five percent of price variance is explainable from physical features, which is genuinely useful as a baseline. But a $160,000 average error on a median price around $460,000 is a honest reminder that a house is more than its square footage.

**On model complexity:** more complexity didn't help here. Linear Regression beat both tree models because the signal in this feature set is largely linear. This isn't a general result. Give a Random Forest rich location data and it would likely pull ahead, but it's a good illustration of why model choice should follow the data, not the other way around.

**On the residuals:** the heteroscedasticity pattern is the most useful diagnostic finding. It points directly at what's missing: the model doesn't know anything about where the house is, and for expensive properties that matters enormously. The diagnostics don't just show that the model has limits, they show *where* and *why*.

## What I Learned

The biggest lesson was how much data quality and feature selection matter before any modeling begins. Cleaning the zero-price records and adding `waterfront` (one binary column) pushed R² from 0.39 to 0.55. No amount of model tuning would have achieved that.

Fair comparison was another thing that required care. Tree models with no constraints will overfit badly and look terrible against a linear model on test data, not because they're worse, but because the comparison isn't set up properly. Getting that right meant applying regularisation and cross-validation consistently across all three.

And finally, the residual diagnostics turned out to be more informative than the headline metrics. R² tells you how well the model fits. Residual plots tell you *how* it's failing, and that's usually the more actionable insight.

## Challenges

The trickiest part wasn't the modeling, it was the data. Zero-price records don't show up in a missing value check. You only spot them if you think to ask whether the values make sense in the real world, not just whether they're present. That kind of domain-aware inspection is easy to skip and expensive to miss.

The model comparison also required some thought. An unconstrained Decision Tree can score R² = -1.97 on test data, which looks like a damning result until you realise it just memorised the training set. Understanding why that happens, and how to fix it, was more valuable than the final numbers.

## Conclusion

This project is a case for keeping things simple and being rigorous about the basics. A well-prepared dataset, a complete feature set, and a fair evaluation framework produced a model that's interpretable, consistent, and honest about its limitations.

For anyone looking to take it further: adding location features (zip code, latitude/longitude) would likely be the single biggest improvement. After that, interaction terms, a log transformation on price to tame the heteroscedasticity and a proper grid search for hyperparameters are all natural next steps.
