## Introduction
Food plays an essential role in our daily lives—not only as nourishment but also as a source of creativity, culture, and comfort. As more people become conscious of their dietary habits and strive to balance convenience with health, the time it takes to prepare meals has become an important consideration. In today's fast-paced world, many home cooks prefer recipes that are quick to make, but we wondered whether this convenience comes at a nutritional cost. Specifically, we aim to investigate whether there is a relationship between cooking time and the amount of calories in a recipe. Are faster meals generally more calorie-dense due to reliance on processed ingredients, or do longer recipes result in richer, higher-calorie dishes? To explore this, we are using two datasets consisting of recipes and reviews from food.com posted since 2008. This dataset was originally compiled for the recommender system research paper Generating Personalized Recipes from Historical User Preferences by Majumder et al., and provides valuable insights into recipe attributes such as cooking time, ingredients, and nutritional information.

The first dataset, recipe, contains 83782 rows, indicating 83782 unique recipes, with 10 columns recording the following information:

| Column           | Description                                                                                                                              |
|------------------|------------------------------------------------------------------------------------------------------------------------------------------|
| `'name'`         | Recipe name                                                                                                                              |
| `'id'`           | Recipe ID                                                                                                                                |
| `'minutes'`      | Minutes to prepare recipe                                                                                                                |
| `'contributor_id'` | User ID who submitted this recipe                                                                                                       |
| `'submitted'`    | Date recipe was submitted                                                                                                                |
| `'tags'`         | Food.com tags for recipe                                                                                                                 |
| `'nutrition'`    | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'`      | Number of steps in recipe                                                                                                                |
| `'steps'`        | Text for recipe steps, in order                                                                                                          |
| `'description'`  | User-provided description                                                                                                                |
| `'ingredients'`  | Text for recipe ingredients                                                                                                              |
| `'n_ingredients'`| Number of ingredients in recipe                                                                                                          |




The second dataset, interactions, contains 731927 rows and each row contains a review from the user on a specific recipe. The columns it includes are:

| Column      | Description             |
|-------------|-------------------------|
| `'user_id'` | User ID                 |
| `'recipe_id'` | Recipe ID              |
| `'date'`    | Date of interaction     |
| `'rating'`  | Rating given            |
| `'review'`  | Review text             |




**Given the datasets, we are investigating whether there is a relationship between cooking time and the amount of calories in a recipe.** To facilitate this analysis, we first extracted and separated the values in the `nutrition` column into individual components such as `calories (#)`, `total fat (PDV)`, and others. We also made use of the `minutes` column, which records the total time required to prepare and cook each recipe. These two columns—`calories (#)` and `minutes`—are central to our analysis. To better explore potential patterns, we performed data cleaning to handle missing or extreme values and conducted summary statistics to examine general trends across different cooking durations.

By examining how cooking time correlates with caloric content, we hope to gain insights into whether quicker meals tend to be lighter or more calorie-dense, or whether longer recipes are associated with richer and potentially higher-calorie dishes. This investigation could be useful for health-conscious users aiming to balance nutrition with convenience and may also provide helpful feedback for recipe creators on Food.com as they design meals that align with users’ time and dietary preferences. Future research may further explore how cooking complexity, ingredient types, or even cuisine styles influence this relationship.


## Data Cleaning and EDA
We begin by merging the raw recipe metadata (RAW_recipes.csv) with user interactions data (interactions.csv) using id and recipe_id as keys. This integration enables us to analyze both recipe attributes and user behaviors simultaneously. To ensure cleaner data, we replace 0.0 ratings with NaN, and compute each recipe's average_rating by grouping on the recipe name.

The original nutrition column in the dataset contains nutritional information for each recipe, stored as a string representation of a list. To make this column usable, we flattened the list into separate columns, one for each nutritional component using lambda function. 

We created two new columns, `prop_carbohydrates` and `prop_fat`, to represent the proportion of calories in a recipe that come specifically from carbohydrates and fat, respectively. These derived features allow us to better understand the nutritional density and macronutrient breakdown of each recipe, which may be predictive of amount of calories.

`prop_carbohydrates`

This column estimates what fraction of total calories in the recipe comes from carbohydrates. The carbohydrates (PDV) column gives the % Daily Value for carbs. Then we convert this PDV into grams of carbohydrates by multiplying by 150 grams (100% daily value for carbs is assumed to be 150g).Since 1 gram of carbohydrate = 4 kcal, we multiply the result by 4 to get the total calories from carbohydrates. Finally, we then divide this value by the total calories in the recipe to compute the proportion.

`prop_fat`

This column calculates the proportion of total calories in a recipe that comes the total fat (PDV) column, converting PDV to grams based on the assumption that 100% fat PDV = 58 grams. Since 1 gram of fat = 9 kcal, we multiply the result by 9 to get calories from fat. Finally, we divide by the total calories of the recipe.


### Univariate Analysis
To explore the distribution of numerical features, we focused on two key variables: `calories` and `minutes` (cooking time). Due to skewness, we applied log transformation.

#### Log-Transformed Calories
To understand the skewness of numeric features such as calories and cooking time, we log-transform these variables to make their distributions more symmetric:

Using np.log, we created a new column log_calories and plot its histogram with 50 bins. The resulting distribution reveals a right-skewed calorie distribution that becomes nearly normal after transformation, concentrating most values between log(4) and log(7). Zero or negative calorie values are replaced with 0 prior to log transformation.

<iframe
  src="assets/log_calories_hist.html"
  width="700"
  height="600"
  frameborder="0"
></iframe>

After log transformation, the distribution became more symmetric and bell-shaped, peaking between log values 5 and 6 (roughly corresponding to 150–400 real calories). This transformation helps ensure that models later on do not overemphasize extreme calorie values and allows better visual comparisons.


#### Log-Transformed Cooking Time
Similarly, we create log_minutes from the minutes column. The histogram shows an extreme right-skew in raw cooking time, with the transformed distribution improving interpretability and reducing the influence of long-tail outliers.

<iframe
  src="assets/log_mins_hist.html"
  width="700"
  height="600"
  frameborder="0"
></iframe>

The transformed histogram shows that most recipes now fall between 2 and 5 log-minutes (which translates to approximately 7–150 minutes in real time). The transformation improves interpretability and ensures subsequent comparisons between groups are not dominated by outliers.



### Bivariate Analysis
To compare cooking behavior across different calorie levels, we split recipes into two groups based on the median calorie value:

<iframe
  src="assets/cooking_time_hist.html"
  width="700"
  height="600"
  frameborder="0"
></iframe>

While both groups peak around log-minutes ≈ 4 (about 55 minutes), high-calorie recipes show a slightly wider spread and a second bump at higher durations. This suggests that high-calorie recipes may involve longer or more complex preparation on average, but the overlap between the groups is significant.

To further compare raw cooking times, we used a box plot grouped by calorie group.

<iframe
  src="assets/boxplot.html"
  width="700"
  height="600"
  frameborder="0"
></iframe>

The box plot highlights that high-calorie recipes generally have higher medians and more extreme high-end outliers. The median cooking time for high-calorie recipes is noticeably larger, and the interquartile range (IQR) is also slightly wider, suggesting more variability in their preparation times.

This bivariate analysis supports the hypothesis that high-calorie dishes are more time-intensive, possibly due to longer ingredient prep, marination, or cooking processes.



### Interesting Aggregates
To understand whether cooking time differs systematically between high- and low-calorie recipes, we calculated summary statistics for each group. However, before doing so, we first removed outliers from the minutes column to ensure that our comparisons were not skewed by extreme values. We first remove Outliers since cooking times in the dataset span an extremely wide range, with some recipes taking thousands or even millions of minutes. To filter out these implausible or extreme durations, we applied the Interquartile Range (IQR) method. 

On average, high-calorie recipes take significantly longer to cook than low-calorie recipes. The mean for high-calorie dishes is around 42 minutes, compared to 31 minutes for low-calorie ones. The medians (40 vs. 25) reinforce this trend, indicating that the difference is not just due to a few high-duration values. However, both groups exhibit considerable standard deviation, reflecting high variability in cooking times. This suggests that while the trend exists, further statistical testing would be needed to determine whether the difference is statistically significant. This aggregate analysis helps us generate hypotheses: for example, that calorie-rich dishes may involve more steps, longer ingredient preparation, or extended baking/cooking processes.

| Calorie Group | Max  | Min | Median | Mean     | Count  | Std Dev   |
|---------------|------|-----|--------|----------|--------|-----------|
| High Calories | 120  | 0   | 40.0   | 42.44    | 101860 | 25.37     |
| Low Calories  | 120  | 1   | 25.0   | 31.43    | 108278 | 22.98     |




## Assessment of Missingness
Three columns, `date`, `rating`, and `review`, in the merged dataset have a significant amount of missing values, so we decided to assess the missingness on the dataframe.

### NMAR Analysis
We believe that the missingness of the `rating` column is Not Missing At Random (NMAR) because the likelihood of a missing rating may depend on the rating itself—an unobserved value. For example, users who felt indifferent or dissatisfied with a recipe might be less motivated to leave a rating, as they may not feel strongly enough to take the extra steps required. In contrast, users with strong opinions—whether highly positive or negative—are more likely to leave ratings due to the emotional motivation to share their experiences. This behavior creates a pattern where missing ratings are systematically related to users’ underlying, unreported impressions of the recipe.

To better assess whether the missingness is NMAR or potentially Missing At Random (MAR), follow-up surveys could be conducted to ask users why they chose not to rate a recipe. If their reasons are unrelated to the actual rating they would have given (e.g., time constraints or forgetting), the missingness could be treated as MAR instead.

### Missing Dependency
We moved on to examine the missingness of the `rating` column in the merged DataFrame by testing whether its missingness is dependent on other variables. Specifically, we are investigating whether the likelihood of a missing rating depends on the cooking time, represented by the `minutes` column.

**Cooking Time and Rating Missingness**

**Null Hypothesis (H₀):** The missingness of ratings does not depend on the cooking time of the recipe.

**Alternative Hypothesis (H₁):** The missingness of ratings does depend on the cooking time of the recipe.

**Test Statistic:** The absolute difference in the mean cooking time (‘minutes’) between recipes with missing ratings and those with non-missing ratings.

**Significance Level (α):** 0.05



We ran a permutation test by shuffling the missingness of minutes for 1000 times to collect 1000 simulating mean differences in the two distributions as described in the test statistic.


<iframe
  src="assets/permutation_min2.html"
  width="700"
  height="600"
  frameborder="0"
></iframe>


The observed statistic of 51.45 is indicated by the red vertical line on the graph. Since the p-value that we found (0.128) is >0.05 which is the significance level that we set, we **fail to reject the null hypothesis**. The missingness of ‘rating’ does not depend on the cooking time of the recipe.


**Calories and Rating**

**Null Hypothesis (H₀):** The missingness of ratings does not depend on the amount of calories of the recipe.

**Alternative Hypothesis (H₁):** The missingness of ratings does depend on the amount of calories of the recipe.

**Test Statistic:** The absolute difference in mean calories between recipes with missing ratings and those with non-missing ratings.

**Significance Level (α):** 0.05


We ran another permutation test by shuffling the missingness of calories for 1000 times to collect 1000 simulating mean differences in the two distributions as described in the test statistic.


<iframe
  src="assets/permutation_rating2.html"
  width="700"
  height="600"
  frameborder="0"
></iframe>


The observed statistic of 69.01 is indicated by the red vertical line on the graph. Since the p-value that we found (0.0) is < 0.05 which is the significance level that we set, we **reject the null hypothesis**. The missingness of rating does depend on the amount of calories of the recipe.



## Hypothesis Testing
As mentioned earlier, we were curious whether high-calorie recipes tend to require more cooking time than low-calorie recipes. Since calorie content often reflects the complexity or richness of a dish, we hypothesized that higher-calorie recipes might involve more ingredients, longer preparation steps, or extended cooking methods.

To evaluate this, we conducted a permutation test using the following setup:


**Null Hypothesis:** There is no difference in cooking times between high-calorie and low-calorie recipes.

**Alternative Hypothesis:** High-calorie recipes take more time to cook than low-calorie recipes.

**Test Statistic:** The difference in mean cooking time (high-calorie minus low-calorie).

**Significance Level:** 0.05

We chose a permutation test because we do not assume normality or equal variance between the two groups, and we want to determine whether the observed difference in cooking time could arise by chance if calorie level had no effect on cooking time. This method is robust for comparing real-world data where underlying distributions may be unknown or skewed.
To perform the test, we split the data into two groups using the median calorie value to define "high-calorie" and "low-calorie" recipes. And then we computed the observed difference in mean cooking time between the two groups. The result was approximately +57.5 minutes, indicating that on average, high-calorie recipes take longer to prepare. We then shuffled the group labels (high/low calorie) 1000 times, each time recomputing the difference in means to generate a null distribution under the assumption of no association. Lastly, we compared the observed difference to this null distribution to compute a one-tailed p-value.

<iframe
  src="assets/hypothesis_low_high2.html"
  width="700"
  height="600"
  frameborder="0"
></iframe>


<iframe
  src="assets/hypothesis_empirical.html"
  width="700"
  height="600"
  frameborder="0"
></iframe>


Since the p-value is far below the 0.05 threshold, we reject the null hypothesis and conclude that high-calorie recipes do take significantly longer to cook than low-calorie recipes. This supports our hypothesis that calorie-dense dishes tend to involve more complex or time-consuming preparation steps.


## Framing a Prediction Problem
In this project, we aim to predict the number of calories in a recipe based on its cooking time, which we frame as a regression problem. Since calories is a continuous numerical variable, we treat this as a standard supervised learning task where the model learns to output a numeric value rather than classifying into categories.

We chose calories as the response variable because it is a fundamental nutritional attribute that users may want to estimate even before seeing full ingredient details. Accurately predicting the calorie content based on preparation characteristics like cooking time may help users or platforms make quicker judgments about recipe suitability, especially for dietary or health-related applications.

The predictor we are using is `minutes`, which represents the total cooking time for the recipe. From our exploratory analysis, we observed a positive correlation between cooking time and calories—longer recipes tend to have higher calorie counts on average. This relationship is likely due to the fact that more complex or time-intensive recipes often involve richer ingredients or larger portions, contributing to increased caloric density.

To evaluate our model’s performance, we will use **Root Mean Squared Error (RMSE)** and **R² score**:

**RMSE** measures the average magnitude of prediction errors, giving higher weight to larger errors.

**R²** score indicates how much of the variation in calorie content can be explained by cooking time.

Since we are only using information available prior to cooking (i.e., time estimate), this model may also serve as a simple calorie estimator in situations where ingredient data is not yet available.

This setup gives us a straightforward, interpretable regression problem based on a single predictor. In future iterations, we could expand the model by incorporating additional features like nutrient proportions, ingredient counts, or recipe type.



## Baseline Model
For our baseline model, we implemented a linear regression pipeline to predict the number of calories in a recipe based on two features: season and minutes. The minutes column represents the cooking time of the recipe and is a numerical variable, while season is a categorical feature derived from the month in which the recipe was submitted. We calculated the season by taking the month modulo 12 and dividing it into four seasonal groups.

To properly handle the categorical feature, we used one-hot encoding on the season column through a ColumnTransformer, which ensures that the linear regression model interprets each season as a separate binary feature. The numerical feature, minutes, was passed through without transformation.

We split the dataset into training and testing sets with an 80/20 ratio and trained our pipeline using the training data. The model's performance on the test set resulted in an R² score of 0.0004 and a root mean squared error (RMSE) of approximately 588.74. These metrics indicate that the baseline model does not capture much of the variance in calorie values, suggesting that season and minutes alone are not strong predictors of calorie content. Future models may benefit from incorporating more relevant nutritional or ingredient-level features.


## Final Model
For the final model, we built a linear regression pipeline to **predict the total calories** of a recipe based on a selected set of features that are available before a recipe is prepared or rated. The features we chose are:

`season`

`minutes`

`prop_fat`

`n_ingredients`

`n_steps`

We evaluated the model using R² score and Root Mean Squared Error (RMSE) to assess how well the model explains the variation in calories and how far its predictions deviate from the true values.

`season`

This column represents the seasonal label derived from the submission date of the recipe. The rationale for including this feature is that certain types of dishes are more common or richer in particular seasons (e.g., hearty, high-calorie recipes in winter). This categorical feature was one-hot encoded using `OneHotEncoder` in our preprocessing pipeline to ensure proper handling by the model.

`minutes`

This column measures the total cooking time in minutes. We included it because our earlier EDA and hypothesis testing showed that high-calorie recipes tend to take longer to prepare. Since cooking time varies widely across recipes, we applied standardization using `StandardScaler` to make this feature more numerically stable and prevent extreme values from dominating the model.

`prop_fat`

This column indicates the proportion of calories in the recipe that come from fat, calculated using the nutritional PDV information. We chose this feature because fat content is a major contributor to total caloric value, and recipes with higher fat density are expected to have higher calorie counts. This column was left as-is since it’s already in proportion form and centered around meaningful units.

`n_ingredients`

This column counts the number of ingredients in a recipe. We calculated the correlation and found a slight positive relationship with calorie content (correlation ≈ 0.13). The intuition is that recipes with more ingredients might have more complex or larger portions, which often lead to higher calorie totals. Although it was initially considered for transformation, we ultimately excluded it from the final preprocessor to simplify the pipeline.

`n_steps`

This column captures the number of procedural steps in the recipe instructions. Similar to `n_ingredients`, we found a positive correlation (~0.15) with calories, suggesting that more involved recipes may lead to richer, higher-calorie outcomes. Since the distribution of `n_steps` is skewed (as shown in the histogram), we applied standardization to scale this feature properly for modeling.

We used **LinearRegression** from sklearn wrapped in a pipeline with preprocessing steps applied through ColumnTransformer. The data was split into a training and testing set using an 80/20 ratio with a fixed random_state for reproducibility.
The performance of our final model was:

R² Score: 0.0456

RMSE: 575.25

By including additional features such as `prop_fat`, `n_ingredients`, and `n_steps`, we were able to increase the explanatory power of the model and reduce prediction error. Although the overall R² remains modest, the enhanced model captures more meaningful variation in recipe calorie content than the baseline.



## Fairness Analysis
For our fairness analysis, we investigated whether our linear regression model predicts calories with equal accuracy across recipes of different quality levels, as measured by their average user rating. We split the dataset into two groups: high-rated recipes and low-rated recipes, using the median average rating as the threshold. We chose the median instead of the mean because ratings are often skewed, and the median provides a more robust central measure.
To assess fairness, we evaluated the model's Root Mean Squared Error (RMSE) separately for each group. RMSE captures how far off the model's calorie predictions are from the actual values — a lower RMSE indicates better predictive accuracy. If the model were fair, we would expect similar RMSE values for both high-rated and low-rated recipes.

**Null Hypothesis:** Our model is fair — the RMSE for high-rated and low-rated recipes is roughly the same, and any difference is due to random variation.

**Alternative Hypothesis:** Our model is unfair — the RMSE differs significantly between high-rated and low-rated recipes.

**Test Statistic:** Difference in RMSE (high-rated – low-rated)

**Significance Level:** 0.05


<iframe
  src="assets/fairness2.html"
  width="700"
  height="600"
  frameborder="0"
></iframe>


We conducted a permutation test to assess the significance of the observed RMSE difference. The observed difference in RMSE between high-rated and low-rated recipes was 111.12, with the high-rated group having substantially worse predictive performance. After shuffling the group labels 1000 times and recalculating the RMSE differences under the null distribution, we found a p-value of 0.0000. This value is below our significance threshold of 0.05, leading us to reject the null hypothesis.

This result suggests that our model is unfair, performing significantly worse on recipes that are more highly rated. In the context of user trust and recipe usability, this discrepancy could mislead users by giving less accurate calorie estimates for the recipes they are most likely to enjoy.

