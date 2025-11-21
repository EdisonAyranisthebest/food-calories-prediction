# Calorie Prediction Using Recipe Features

**Author:** Edison Ayran

**Course:** DSC 80 at UC San Diego

---

## Introduction

### Dataset Overview
This analysis examines the **Recipes and Ratings** dataset from Food.com, containing recipes and user reviews posted since 2008. The dataset provides information about recipe characteristics, nutritional content, and user engagement through ratings.

### Research Question
**What is the relationship between a recipe's nutritional profile and its average user rating?**

More specifically: Do "healthier" recipes such as those higher in protein or lower in calories, receive systematically different ratings than less healthy recipes?

This question allows us to investigate whether user preferences on Food.com align more closely with nutritional value or indulgence.

### Why This Matters
Understanding what makes recipes popular can help:
* **Recipe creators** optimize their content for user engagement
* **Home cooks** find highly-rated recipes that match their dietary preferences
* **Researchers** understand trends in food preferences and online communities
* **Food platforms** design better recommendation systems

### Dataset Composition
The dataset contains **83,782 recipes** and **731,927 user interactions** (ratings and reviews). After cleaning and processing, 81,173 recipes (97%) have at least one rating.

**Key columns relevant to our analysis:**

**Recipe Characteristics:**
- `name`: Recipe name
- `id`: Unique recipe identifier
- `minutes`: Preparation time
- `n_steps`: Number of steps in the recipe
- `submitted`: Date the recipe was posted

**Nutritional Information:**
- `calories`: Calorie content
- `protein_pdv`: Protein as percentage of daily value
- `sugar_pdv`: Sugar as percentage of daily value
- `total_fat_pdv`: Total fat as percentage of daily value
- `carbohydrates_pdv`: Carbohydrates as percentage of daily value

**User Engagement:**
- `avg_rating`: Average rating (1-5 stars) calculated from all user reviews

Together, these variables allow us to explore how nutritional content, recipe complexity, and preparation time relate to user satisfaction.

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning Steps

**1. Merging Datasets**
I left-merged the recipes and interactions datasets on recipe ID, keeping all recipes even if they had no ratings. This resulted in 234,429 rows (multiple interactions per recipe).

**2. Handling Missing Ratings**
Ratings of 0 were replaced with `NaN` because the rating scale is 1-5 stars. A rating of 0 indicates that no rating was given (missing data) rather than "0 stars."

Before replacement: 15,035 ratings of 0  
After replacement: 15,036 NaN ratings

**3. Calculating Average Ratings**
I computed the mean rating for each recipe across all its reviews:
- 81,173 recipes have at least one rating
- 2,609 recipes have no ratings (3.1% of total)

**4. Parsing Nutritional Data**
The `nutrition` column was stored as a string representation of a list. I parsed it into separate numerical columns for calories, protein, sugar, sodium, total fat, saturated fat, and carbohydrates (all as percentage of daily value except calories).

**Cleaned Dataset Structure:**
- **83,782 recipes** total
- **21 columns** after cleaning (up from 12 originally)
- **Missing values:** Only in `avg_rating` (2,609), `description` (70), and `name` (1)

**Head of Cleaned Data:**

| name                                   | minutes | n_steps | calories | protein_pdv | avg_rating |
|----------------------------------------|---------|---------|----------|-------------|------------|
| 1 brownies in the world best ever      | 40      | 10      | 138.4    | 3.0         | 4.0        |
| 1 in canada chocolate chip cookies     | 45      | 12      | 595.1    | 13.0        | 5.0        |
| 412 broccoli casserole                 | 40      | 6       | 194.8    | 22.0        | 5.0        |
| millionaire pound cake                 | 120     | 7       | 878.3    | 20.0        | 5.0        |
| 2000 meatloaf                          | 90      | 17      | 267.0    | 29.0        | 5.0        |

### Univariate Analysis

**Distribution of Average Ratings:**

<iframe
  src="assets/rating-distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of ratings is heavily left-skewed, with most recipes receiving ratings between 4.5 and 5 stars. This suggests that users on Food.com tend to rate recipes generously, or that poorly-rated recipes may not survive long enough to accumulate many ratings.

**Distribution of Calories:**

<iframe
  src="assets/calorie-distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Recipe calories follow a right-skewed distribution with most recipes under 500 calories. The median is 305 calories, but there's a long tail of high-calorie recipes extending past 1,000 calories.

### Bivariate Analysis

**Protein Content vs. Average Rating:**

<iframe
  src="assets/protein-vs-rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The scatter plot shows a very weak relationship between protein content and average rating. Most recipes cluster around 4.5-5 stars regardless of protein level, suggesting that users don't systematically prefer high-protein recipes.

**Average Rating by Cooking Time:**

<iframe
  src="assets/time-vs-rating-box.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Recipes across all time categories (quick, medium, long) have similar median ratings around 5.0 stars. This suggests cooking time doesn't strongly influence user satisfaction.

### Interesting Aggregates

**Average Recipe Characteristics by Rating Tier:**

| rating_tier        | calories | protein_pdv | sugar_pdv | minutes | n_steps |
|--------------------|----------|-------------|-----------|---------|---------|
| Low (1-3)          | 423.65   | 32.11       | 90.81     | 112.90  | 10.68   |
| Medium (3-4)       | 428.32   | 33.35       | 90.60     | 109.48  | 10.35   |
| High (4-4.5)       | 428.04   | 33.11       | 89.91     | 114.91  | 10.07   |
| Very High (4.5-5)  | 430.97   | 33.16       | 89.51     | 115.69  | 10.08   |

Interestingly, nutritional content (calories, protein, sugar) is remarkably similar across all rating tiers. This suggests that ratings are not strongly driven by nutritional factors, but rather by taste, presentation, or other qualities not captured in our data.

---

## Assessment of Missingness

### NMAR Analysis

I believe the `avg_rating` column is likely **NMAR (Not Missing At Random)**.

**Reasoning:** The missingness of `avg_rating` likely depends on the unobserved rating value itself. Recipes with no ratings may be:
1. **Unpopular or low-quality recipes** that users didn't bother to rate
2. **New recipes** that haven't been discovered yet
3. **Recipes that users tried and disliked** so much they didn't leave a rating

The probability of missingness depends on what the rating would have been if observed—this is the definition of NMAR.

**Additional data to make it MAR:**
To make this MAR instead of NMAR, we could collect:
- **View counts:** Number of times the recipe was viewed
- **Recipe engagement metrics:** Whether users saved, printed, or shared the recipe
- **Time since posting:** How long the recipe has been on the platform

With this data, we could explain the missingness through observable variables rather than the missing rating itself.

### Missingness Dependency

I tested whether the missingness of `avg_rating` depends on other columns using permutation tests.

**Test 1: Does missingness depend on `minutes` (preparation time)?**

<iframe
  src="assets/missingness-minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Result:** p-value < 0.001 (significant)

The missingness of `avg_rating` **DOES depend** on preparation time. Recipes without ratings tend to have longer preparation times (mean of 228.7 minutes) compared to recipes with ratings (mean of 111.4 minutes). This 12.8-minute difference is statistically significant.

**Test 2: Does missingness depend on `n_steps` (recipe complexity)?**

<iframe
  src="assets/missingness-steps.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Result:** p-value < 0.001 (significant)

The missingness of `avg_rating` **DOES depend** on the number of steps. Recipes without ratings have more steps on average (11.6 vs. 10.1), suggesting that more complex recipes are less likely to be tried and rated.

**Test 3: Does missingness depend on `calories`?**

**Result:** p-value < 0.001 (significant)

The missingness also depends on calorie content, with unrated recipes having slightly higher calories on average.

### Conclusion

The missingness of `avg_rating` is **MAR (Missing At Random)** conditional on observable features like preparation time, recipe complexity, and calorie content. This suggests that longer, more complex, higher-calorie recipes are systematically less likely to receive ratings, possibly because users find them intimidating or time-consuming.

---

## Hypothesis Testing

### Research Question
Do high-protein recipes receive higher average ratings than low-protein recipes?

### Hypotheses

**Null Hypothesis (H₀):** High-protein recipes (protein ≥ 50% DV) and low-protein recipes (protein < 50% DV) have the same average rating. Any observed difference in mean average ratings is due to random chance.

**Alternative Hypothesis (H₁):** High-protein recipes have a higher average rating than low-protein recipes.

### Test Setup
- **Test Statistic:** Difference in mean ratings (high-protein minus low-protein)
- **Significance Level:** α = 0.05
- **Method:** One-sided permutation test (10,000 permutations)

### Results

**Group Sizes:**
- Low-protein recipes (< 50% DV): 61,043
- High-protein recipes (≥ 50% DV): 20,130

**Observed Statistics:**
- Mean rating (low-protein): 4.6313
- Mean rating (high-protein): 4.6074
- **Observed difference:** -0.0239 (high-protein recipes actually have *lower* ratings)

**P-value:** 1.0000

<iframe
  src="assets/hypothesis-test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Conclusion

We **fail to reject the null hypothesis**. At the 5% significance level, there is no statistical evidence that high-protein recipes have higher average ratings than low-protein recipes.

In fact, the observed difference was slightly negative (high-protein recipes had marginally *lower* ratings), though this difference is negligible (0.024 stars) and not statistically significant.

**Interpretation:** Protein content does not appear to influence recipe ratings on Food.com. Both groups have nearly identical average ratings (≈4.6/5), suggesting that users rate recipes based on factors other than nutritional content—likely taste, ease of preparation, or presentation.

**Limitations:**
- The 50% DV threshold is somewhat arbitrary
- Ratings may be affected by confounding variables (cooking time, complexity, cuisine type)
- The rating distribution is highly skewed (many 5-star ratings), making differences harder to detect

---

## Framing a Prediction Problem

### Prediction Problem
**Predict the calorie content of recipes** (regression problem)

### Response Variable
**Target:** `calories` (continuous numerical variable)

### Justification

**Why predict calories?**
- Helps users quickly estimate nutritional content without manual calculation
- Enables automatic categorization of recipes by calorie level
- Assists with dietary decision-making

**Time of prediction:** At recipe submission, before detailed nutritional analysis is performed.

**Features available at prediction time:**
- Recipe metadata: name, tags, ingredients list, steps, preparation time
- Recipe structure: number of ingredients, number of steps
- **NOT available:** User ratings (come after posting), other nutritional values (require same analysis as calories)

### Evaluation Metric: RMSE

**Chosen metric:** Root Mean Squared Error (RMSE)

**Why RMSE?**
1. **Interpretable:** RMSE is in calories, making it easy to understand (RMSE of 100 = off by ~100 calories)
2. **Penalizes large errors:** Squares errors before averaging, so being off by 500 calories is penalized much more than being off by 50
3. **Standard for regression:** Widely used, making results comparable to other work

**Why not alternatives?**
- **MAE:** Doesn't penalize extreme errors enough (a 500-cal error is as problematic as five 100-cal errors)
- **R²:** Harder to interpret practically
- **MAPE:** Problematic for low-calorie recipes (division by small numbers)

---

## Baseline Model

### Model Description

**Algorithm:** Linear Regression with StandardScaler preprocessing

**Features:** 3 quantitative features
1. `minutes` (continuous) - Preparation time
2. `n_steps` (discrete, treated as quantitative) - Number of steps
3. `n_ingredients` (discrete, treated as quantitative) - Number of ingredients

**No categorical features** in the baseline model (keeping it simple).

**Pipeline:**
```python
Pipeline([
    ('scaler', StandardScaler()),
    ('regressor', LinearRegression())
])
```

### Performance

**Training RMSE:** 639.20 calories  
**Test RMSE:** 583.40 calories

**Baseline comparison:**
- Naive baseline (always predicting mean): ~640 calories RMSE
- Our model: 583.40 calories RMSE
- **Improvement:** ~9% better than naive

### Model Assessment

**Is this model "good"?**

**Not really.** While the baseline model is better than simply predicting the mean, an RMSE of 583 calories is quite high:
- For a typical 300-calorie recipe, this represents a ~194% error
- Predictions could easily be off by 1,000+ calories
- The model is unreliable for practical dietary decisions

**Why is performance limited?**
1. **Weak features:** Our basic numerical features (minutes, steps, ingredients) have very weak correlations with calories (all < 0.13)
2. **Missing key information:** Knowing a recipe has 10 ingredients doesn't tell us *what* those ingredients are (butter vs. lettuce matters enormously!)
3. **Linear assumptions:** Assumes simple additive relationships

**Path forward:** We need to engineer features that capture *what type* of food the recipe is, not just how it's structured.

---

## Final Model

### Iteration Process

We built two iterations beyond the baseline:

**Version 1 (Ridge Regression):**
- Added 9 engineered features from tags and ingredients
- Used Ridge regression with L2 regularization
- Test RMSE: **577.43 calories** (1.0% improvement over baseline)

**Version 2 (Random Forest):**
- Enhanced feature engineering (16 total features)
- Switched to Random Forest to capture non-linearity
- Test RMSE: **587.70 calories** (WORSE than baseline!)
- Showed severe overfitting (training RMSE: 417.96, gap of ~170 calories)

**Final Model Choice:** **Ridge Regression (V1)** - better generalization, modest but real improvement

### Features Added (Beyond Baseline)

**From recipe tags (6 binary features):**
1. `is_dessert` - Dessert/sweet indicators
2. `is_healthy` - Healthy/low-fat indicators
3. `is_main_dish` - Main course indicators
4. `is_appetizer` - Appetizer/snack indicators

**From ingredients (2 count features):**
5. `high_calorie_ingredients` - Count of butter, oil, cream, cheese, sugar
6. `low_calorie_ingredients` - Count of vegetables, herbs, spices

**Transformations (3 features):**
7. `log_minutes` - Log transformation of preparation time
8. `ingredients_squared` - Polynomial feature (n_ingredients²)
9. `minutes_x_ingredients` - Interaction term

**Total:** 12 features (3 original + 9 engineered)

### Why These Features?

These features capture the **data generating process** for calories:

1. **Recipe category matters most:** Desserts have different calorie profiles than salads, regardless of ingredient count
2. **Ingredient types, not counts:** Five vegetables ≠ five dairy products in terms of calories
3. **Non-linear relationships:** Cooking time may have diminishing returns (log helps)
4. **Interactions:** Complex recipes (high time × ingredients) tend to be richer

### Hyperparameter Tuning

**Method:** 5-fold GridSearchCV  
**Parameter tuned:** `alpha` (regularization strength)  
**Tested values:** [0.001, 0.01, 0.1, 1.0, 10.0, 100.0]  
**Best alpha:** 1.0  
**Best CV RMSE:** ~575 calories

### Final Performance

**Training RMSE:** 632.50 calories  
**Test RMSE:** 577.43 calories  
**Improvement over baseline:** 5.96 calories (1.0%)

<iframe
  src="assets/model-comparison.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Model Assessment

**Why was improvement modest?**

Despite thoughtful feature engineering, the improvement was only 1% because:

1. **Fundamental data limitation:** We don't have ingredient **quantities**
   - A recipe with "1 tsp butter" vs "2 sticks butter" both get `high_calorie_ingredients = 1`
   - This is the crucial missing information

2. **Binary features too coarse:** Tag-based features are yes/no indicators
   - Both a fruit salad and a triple-chocolate cake are tagged "dessert"
   - Doesn't capture the calorie variance within categories

3. **Linear model:** Ridge regression assumes additive effects
   - May not capture complex interactions between ingredients and cooking methods

**Lessons learned:**
- **More complex ≠ better:** Random Forest's complexity hurt when features are weak (V2 overfitted)
- **Feature quality matters most:** Sophisticated models can't compensate for missing crucial information
- **Regularization helps:** Ridge's L2 penalty prevented overfitting better than Random Forest

**What would actually help?**
- Ingredient quantities ("2 cups butter" vs "1 tsp butter")
- Ingredient-level nutritional data
- Recipe portion sizes
- Preparation methods (fried vs. grilled)

---

## Fairness Analysis

### Research Question
Does our model predict calories equally well for **dessert recipes** vs. **non-dessert recipes**?

### Why This Matters
- Desserts and non-desserts have very different calorie profiles
- Systematic prediction errors could mislead users making dietary decisions
- Users need accurate predictions regardless of recipe type

### Groups
- **Group X (Desserts):** 3,418 recipes (20.4% of test set)
- **Group Y (Non-Desserts):** 13,339 recipes (79.6% of test set)

### Hypotheses

**Null Hypothesis (H₀):** The model is fair. RMSE for desserts and non-desserts are roughly the same; any observed difference is due to random chance.

**Alternative Hypothesis (H₁):** The model is unfair. RMSE for desserts and non-desserts are significantly different.

### Test Setup
- **Evaluation Metric:** RMSE
- **Test Statistic:** |RMSE(desserts) - RMSE(non-desserts)|
- **Significance Level:** α = 0.05
- **Method:** Permutation test (1,000 permutations)

### Results

**Observed Performance:**
- RMSE for Desserts: **928.38 calories**
- RMSE for Non-Desserts: **444.99 calories**
- **Observed Difference: 483.39 calories**

**P-value:** < 0.001 (highly significant)

<iframe
  src="assets/fairness-test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Distribution of Errors by Group:**

<iframe
  src="assets/fairness-boxplot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Conclusion

**The model is UNFAIR.**

At the 5% significance level, we have strong evidence that the model performs significantly worse for dessert recipes. The model's error for desserts is **more than double** that of non-desserts (108% higher).

**Additional Evidence:**
- **Mean Absolute Error:** Desserts 379.59 cal vs. Non-desserts 234.36 cal (145 cal difference)
- **Median Absolute Error:** Desserts 244.98 cal vs. Non-desserts 169.99 cal (75 cal difference)
- **Large errors (>200 cal):** 59.9% of desserts vs. 41.7% of non-desserts

### Why Is the Model Unfair?

**1. Higher calorie variance in desserts:**
- Dessert range: 50 cal (fruit salad) to 1,500+ cal (cheesecake)
- Non-desserts: More consistent (300-800 cal typically)

**2. Ingredient quantity matters more for desserts:**
- Desserts use concentrated sugars and fats
- Small changes in amounts = massive calorie swings
- Our model only captures ingredient *presence*, not *amounts*

**3. Feature design limitations:**
- Binary features (has_sugar, has_fat) don't distinguish "1 tsp" from "3 cups"
- The `is_dessert` tag doesn't differentiate low-cal from high-cal desserts

### Implications

**For users:**
- Dessert calorie predictions are **unreliable** (RMSE ~930 calories)
- Could severely underestimate dessert intake
- Non-dessert predictions are more trustworthy (RMSE ~445 calories)

**For future work:**
- Collect ingredient quantities
- Create dessert-specific features
- Train separate models for desserts vs. non-desserts
- Warn users about prediction uncertainty for desserts

---

## Conclusion

This analysis revealed that **nutritional content has little relationship with recipe ratings** on Food.com. Users rate recipes based on taste and satisfaction rather than health metrics. Our predictive model for calories achieved modest performance (577 RMSE), limited by the absence of ingredient quantities. Most significantly, the model exhibited **substantial unfairness**, performing twice as poorly on desserts—a critical limitation for any practical deployment.

**Key Takeaways:**
1. Protein content doesn't affect ratings (hypothesis test: p = 1.0)
2. Simple features poorly predict calories (need ingredient amounts)
3. Feature engineering helps, but can't overcome data limitations
4. Model fairness analysis is crucial—revealed severe dessert prediction bias

**GitHub Repository:** (https://github.com/EdisonAyranisthebest/food-calories-prediction)
