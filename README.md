## Introduction
Food plays an essential role in our daily lives—not only as nourishment but also as a source of creativity, culture, and comfort. As more people become conscious of their dietary habits and strive to balance convenience with health, the time it takes to prepare meals has become an important consideration. In today's fast-paced world, many home cooks prefer recipes that are quick to make, but we wondered whether this convenience comes at a nutritional cost. Specifically, we aim to investigate whether there is a relationship between cooking time and the amount of calories in a recipe. Are faster meals generally more calorie-dense due to reliance on processed ingredients, or do longer recipes result in richer, higher-calorie dishes? To explore this, we are using two datasets consisting of recipes and reviews from food.com posted since 2008. This dataset was originally compiled for the recommender system research paper Generating Personalized Recipes from Historical User Preferences by Majumder et al., and provides valuable insights into recipe attributes such as cooking time, ingredients, and nutritional information.

The first dataset, recipe, contains 83782 rows, indicating 83782 unique recipes, with 10 columns recording the following information:





The second dataset, interactions, contains 731927 rows and each row contains a review from the user on a specific recipe. The columns it includes are:





**Given the datasets, we are investigating whether there is a relationship between cooking time and the amount of calories in a recipe.** To facilitate this analysis, we first extracted and separated the values in the ‘nutrition’ column into individual components such as ‘calories (#)’, ‘total fat (PDV)’, and others. We also made use of the ‘minutes’ column, which records the total time required to prepare and cook each recipe. These two columns—‘calories (#)’ and ‘minutes’—are central to our analysis. To better explore potential patterns, we performed data cleaning to handle missing or extreme values and conducted summary statistics to examine general trends across different cooking durations.

By examining how cooking time correlates with caloric content, we hope to gain insights into whether quicker meals tend to be lighter or more calorie-dense, or whether longer recipes are associated with richer and potentially higher-calorie dishes. This investigation could be useful for health-conscious users aiming to balance nutrition with convenience and may also provide helpful feedback for recipe creators on Food.com as they design meals that align with users’ time and dietary preferences. Future research may further explore how cooking complexity, ingredient types, or even cuisine styles influence this relationship.


## Data Cleaning and EDA
We begin by merging the raw recipe metadata (RAW_recipes.csv) with user interactions data (interactions.csv) using id and recipe_id as keys. This integration enables us to analyze both recipe attributes and user behaviors simultaneously. To ensure cleaner data, we replace 0.0 ratings with NaN, and compute each recipe's average_rating by grouping on the recipe name.
The original nutrition column in the dataset contains nutritional information for each recipe, stored as a string representation of a list. To make this column usable, we flattened the list into separate columns, one for each nutritional component using lambda function. 

We created two new columns, prop_carbohydrates and prop_fat, to represent the proportion of calories in a recipe that come specifically from carbohydrates and fat, respectively. These derived features allow us to better understand the nutritional density and macronutrient breakdown of each recipe, which may be predictive of user preferences or ratings.

`prop_carbohydrates`

This column estimates what fraction of total calories in the recipe comes from carbohydrates. The carbohydrates (PDV) column gives the % Daily Value for carbs. Then we convert this PDV into grams of carbohydrates by multiplying by 150 grams (100% daily value for carbs is assumed to be 150g).Since 1 gram of carbohydrate = 4 kcal, we multiply the result by 4 to get the total calories from carbohydrates. Finally, we then divide this value by the total calories in the recipe to compute the proportion

`prop_fat`

This column calculates the proportion of total calories in a recipe that comes the total fat (PDV) column, converting PDV to grams based on the assumption that 100% fat PDV = 58 grams. Since 1 gram of fat = 9 kcal, we multiply the result by 9 to get calories from fat. Finally, we divide by the total calories of the recipe.

### Univariate Analysis


