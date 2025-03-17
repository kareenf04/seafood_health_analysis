# Investigation on the Relationship Between the Presence of Seafood and the Health Score of Recipes
**Authors**: Kareen Farhat, Vani Sahjwani
## Project Overview
This is a data science project focused on investigating the relationship between the use of seafood and health scores in recipes. The datasets used to explore this topic were derived from [food.com](https://www.food.com/?ref=nav). This project is for DSC80 at UCSD.


## Introduction

Food plays a fundamental role in our daily lives, influencing both our nutrition and overall well-being. Among the various food groups, seafood is often considered a healthy option due to its high protein content and essential nutrients. However, its actual impact on the health scores of recipes remains unclear. This project seeks to explore the relationship between the presence of seafood in recipes and their assigned health scores.  

To conduct this investigation, we analyzed datasets from [food.com](https://www.food.com/?ref=nav), containing recipes and user-submitted ratings. The datasets include information on ingredients, preparation time, nutritional values, and user interactions. The primary question guiding this study is:  

**Does the presence of seafood in a recipe correlate with a higher health score?**  

Understanding this relationship is important for both consumers seeking healthier meal options and recipe developers aiming to cater to health-conscious individuals.  

### Data Overview  

The dataset consists of two main files:  

- **RAW_recipes.csv** (83,782 rows) contains details about each recipe, including:  
  - `name`: Name of the recipe  
  - `id`: Unique identifier for each recipe  
  - `minutes`: Preparation time  
  - `tags`: List of tags associated with the recipe  
  - `nutrition`: A list containing nutritional information:  
    - Calories  
    - Total fat (PDV)  
    - Sugar (PDV)  
    - Sodium (PDV)  
    - Protein (PDV)  
    - Saturated fat (PDV)  
    - Carbohydrates (PDV)  
    N.B. Here PDV stands for 'percentage daily value' based on a 2,000 calorie intake.
  - `description`: A brief user-provided summary of the recipe  

- **RAW_interactions.csv** (731,927 rows) includes user ratings and reviews:  
  - `user_id`: ID of the user who rated the recipe  
  - `recipe_id`: ID of the rated recipe  
  - `rating`: The score given by the user (ranging from 1 to 5)  

## Data Cleaning and Exploratory Data Analysis  

To prepare the data for analysis, we followed these steps:  

1. **Merged the Recipes and Interactions Datasets**  
   - We performed a left merge on `id` from the recipes dataset and `recipe_id` from the interactions dataset to associate ratings with recipes.  

2. **Handled Missing Values**  
   - Ratings of 0 were replaced with `NaN`, as they likely represent missing data rather than actual ratings.  

3. **Computed Average Ratings**  
   - For each recipe, we calculated the average user rating to gain insight into overall reception. This was added as a column named `rating`.

4. **Identified Seafood Recipes**  
    - A recipe was classified as containing seafood if either:  
        - Its `tags` column included seafood-related terms (e.g., `'seafood'`, `'crab'`, `'fish'`).  
        - Its `ingredients` string contained seafood-related items (e.g., `'salmon'`, `'shrimp'`, `'tuna'`).  
    - We then added an `is_sea` column of boolean values which classified each recipe as either containing or not containing seafood.

5. **Extracted and Processed Nutrition Information**  
   - The `nutrition` column was split into individual columns for the relevent values (calories, sodium, protein, saturated fat, and total fat) and these columns were all then standardized. These columns were named `calories`, `sodium`, `protein`, `satfat`, and `total_fat` respectively.
   - We then standardized all of the nutrition columns except for the `calories` column. These stanndardized values were used to create a score for 'health' which we defined as $`1-\frac{sodium \times saturated fat}{protein \times total fat}`$

The first few rows of this cleaned DataFrame are shown below, in which we have selected the most relevant columns to our analysis.

| id     | name                                  | minutes | tags                                              | n_ingredients | rating | calories | sodium | protein | satfat | total_fat | health_score |
|--:-----|--:------------------------------------|--:------|--:------------------------------------------------|--:------------|--:-----|--:-------|--:-----|--:------|--:-----|--:--------|--:-----------|
| 275022 | impossible macaroni and cheese pie    | 50      | ['60-minutes-or-less', 'time-to-make', 'course... | 7             | 3.0    | 386.1    | -0.02  | 0.11    | 0.11   | 0.02      | 2.08         |
| 275022 | impossible rhubarb pie                | 55      | ['60-minutes-or-less', 'time-to-make', 'course... | 8             | 3.0    | 377.1    | -0.34  | -0.31   | -0.07  | -0.24     | 0.69         |
| 275022 | impossible seafood pie                | 45      | ['60-minutes-or-less', 'time-to-make', 'course... | 9             | 3.0    | 326.6    | 0.06   | 0.05    | 0.05   | -0.04     | 2.38         |
| 275022 | paula deen s caramel apple cheesecake | 45      | ['60-minutes-or-less', 'time-to-make', 'course... | 9             | 5.0    | 577.7    | -0.17  | -0.30   | 0.14   | 0.33      | 0.77         |
| 275022 | midori poached pears                  | 25      | ['lactose', '30-minutes-or-less', 'time-to-mak... | 9             | 5.0    | 386.9    | -0.72  | -0.49   | -0.23  | -0.53     | 0.35         |

With this cleaned dataset, we are ready to analyze how the presence of seafood correlates with the health scoresand ratings of recipes.

### Univariate Analysis
For this analysis, we examined the average health score per number of ingredients in the recipes. As the plot below shows that the health score tends to increase as the number of ingredients increases. However, recipes with more than 20 ingredients do not follow this trend, and this could suggest that excessively complex recipes may include ingredients that negatively impact health scores, such as components that are high in saturated fats or sodium. Alternatively, the deviation could be due to fewer data points for recipes with more than 20 ingredients on food.com, leading to increased variability in health scores. Further analysis could explore whether specific ingredient types contribute to this pattern.
<iframe
  src="assets/univariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis
For this analysis, we examined the average health score at each average rating for both recipes that did and did not include seafood. The graph below shows that  all recipes that were given an average rating of at least 1 had a higher health score when they contained seafood. 
<iframe
  src="assets/bivariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Some Aggregates
For this section, we investigated the relationship between the preparation time (in minutes) and the health score of the recipes. First, we created a smaller dataframe, 'less_time' to store the recipes with prep times that are at most 3 hours. We then create 4 columns called `mean`, `median`, `max`, and `min` in which we calculate and store the relevant data for each of the recipes. Lastly we grouped this dataframe by the columns `is_sea` and `minutes`.

|   | is_sea | minutes | mean  | median | max   | min   |
|---|--------|---------|-------|--------|-------|-------|
| 0 | False  | 0       | 78.09 | 78.09  | 78.09 | 78.09 |
| 1 | False  | 1       | 41.70 | 37.37  | 99.00 | 0.94  |
| 2 | False  | 2       | 41.79 | 37.37  | 99.10 | 0.94  |
| 3 | False  | 3       | 40.97 | 37.37  | 99.40 | 0.88  |
| 4 | False  | 4       | 41.12 | 37.37  | 98.42 | 0.91  |

Interestingly, the graph shows that as the prep time increases the health score of a recipe fluctuates more and more regardless of wether a recipe contains seafood or not. Also, we see that in recipes with seafood the median is consistently greater than the mean suggesting that a few low-scoring seafood recipes may be pulling the mean down while the majority of seafood recipes tend to have relatively high health scores. This right-skewed distribution indicates that while most seafood recipes are considered healthy, there are some exceptions with significantly lower health scores, possibly due to preparation methods that take longer or additional ingredients.
<iframe
  src="assets/agg_sea.html"
  width="800"
  height="600"
  frameborder="0"
></iframe><iframe
  src="assets/agg_sea.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


## Assessment of Missingness  

Two columns, **`description`** and **`review`**, in the initial merged dataset contain missing values. To better understand the nature of these missing entries, we conducted an analysis to determine whether the missingness follows any systematic patterns.

### NMAR Analysis  

We suspect that the missingness of the **`review`** column is **Not Missing At Random (NMAR)**. Users may be less inclined to leave a review if they feel indifferent about a recipe, as providing feedback requires extra effort. In contrast, recipes that evoke a strong positive or negative reaction are more likely to receive reviews. This pattern suggests that the absence of a review may itself carry information about the recipe's impact on the user.

### Missingness Dependency  

We further examined whether the missingness in **`description`** is associated with other variables in our dataset, specifically with:
- **`is_sea`**: a boolean indicator of whether the recipe contains seafood.
- **`health_score`**: the computed health score of a recipe.

#### Health Score and Missing Descriptions

- **Null Hypothesis:** The missingness of **`description`** does not depend on the **health score** of a recipe.
- **Alternative Hypothesis:** The missingness of **`description`** does depend on the **health score** of a recipe.
_Significance Level: 0.05_

A permutation test was performed by comparing the mean health scores of recipes with missing descriptions to those with non-missing descriptions.

<iframe
  src="assets/missing1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

- **Observed difference:** 0.6965 
- **p-value:** 0.515

Since the p-value is greater than 0.05, we fail to reject the null hypothesis. This suggests that the health score of a recipe does not significantly influence whether its description is missing.

#### Seafood and Missing Descriptions

- **Null Hypothesis:** The missingness of **`description`** does not depend on whether a recipe contains seafood.
- **Alternative Hypothesis:** The missingness of **`description`** depends on whether a recipe contains seafood.

Another permutation test was conducted by shuffling the missingness of **`description`** and comparing the proportion of missing values between seafood and non-seafood recipes.

<iframe
  src="assets/missing2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

- **Observed difference:** 0.0002 
- **p-value:** 0.6612

Again, since the p-value is greater than 0.05, we fail to reject the null hypothesis. This suggests that the presence of seafood in a recipe does not significantly influence whether its description is missing.

Our analysis shows that the missing of **descriptions** appears to be random with respect to seafood presence and health score.


## Hypothesis Testing  

As mentioned in the introduction, we are investigating whether seafood recipes are healthier than non-seafood recipes and whether they receive higher ratings. We conducted two permutation tests to analyze these claims.

### Health Score Comparison
To examine whether seafood recipes are healthier than non-seafood recipes, we ran a permutation test with the following hypotheses:

- **Null Hypothesis:** Seafood recipes are as healthy as non-seafood recipes. Any observed difference in health scores is due to random chance.
- **Alternative Hypothesis:** Seafood recipes are healthier than non-seafood recipes.
**Test Statistic:** The difference in median health scores between seafood and non-seafood recipes.
_Significance Level: 0.05_

The reason we chose to run a permutation test is because we do not have any information on the population, and we want to check if the two distributions look like they come from the same population.

We proposed that seafood recipes differ in health score because of the study we are concerned with, and we would like to verify this using the recipes in our dataset, so we used the average health score.

For the test statistic, we chose the difference in mean of the health scores instead of the absolute difference in mean. This is because we have a directional hypothesis — that seafood recipes are scored higher than other recipes. By looking at the difference in mean between the two groups, we can see what type of recipes typically have a higher health score, which answers our question.

To run the test, we first split the data points into two groups: seafood recipes and non-seafood recipes. _The **observed statistic** was **4.9703**._

Then, we shuffled the is_seafood labels 1000 times and recalculated the difference in mean health scores for each permutation. _We obtained a **p-value** of **0.000**._

<iframe
  src="assets/hyp1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Conclusion:** Since the p-value is less than 0.05, we reject the null hypothesis. This suggests that seafood recipes tend to be healthier than non-seafood recipes. While we confirm the claimed health benefits of seafood, we can also infer that recipes containing seafood maintain these benefits, even though they might include other ingredients with varying health factors.

### Rating Comparison
To investigate whether seafood recipes receive higher ratings than non-seafood recipes, we ran another permutation test with the following hypotheses:

- **Null Hypothesis:** Seafood recipes are rated as highly as non-seafood recipes. Any observed difference in ratings is due to random chance.
- **Alternative Hypothesis:** Seafood recipes are rated higher than non-seafood recipes.
**Test Statistic:** The difference in mean ratings between seafood and non-seafood recipes.
_Significance Level: 0.05_

We proposed that people rate seafood recipes differently because people might be concerned with the health benefits related to the recipe. We would like to capture all opinions from users, so we used rating instead of an average rating that factors in taste.

For the test statistic, we chose the difference in mean of the ratings of the two groups instead of the absolute difference in mean. This is because we have a directional hypothesis—that people rate seafood recipes higher than other recipes. By looking at the difference in mean between the two groups, we can see what type of recipes typically have a higher rating, which answers our question.

We chose a permutation test because we do not assume normality in ratings and want to determine if the observed differences are due to random variation. Additionally, we dropped NaN ratings because they represented recipes that had no recorded ratings. Treating NaN as 0 would have skewed the analysis, introducing artificial outliers and misrepresenting the actual distribution of ratings.

After shuffling the is_seafood labels 1000 times and computing the mean rating difference, we obtained a p-value of 0.171.

<iframe
  src="assets/hyp2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Conclusion:** Since the p-value is greater than 0.05, we fail to reject the null hypothesis. This suggests that seafood recipes are not rated significantly higher than non-seafood recipes, meaning any observed difference could be due to chance. Ratings depend on multiple factors, and while some users may prioritize taste, others may prioritize health benefits. As a result, ratings can vary significantly and may not necessarily reflect whether a recipe contains seafood.