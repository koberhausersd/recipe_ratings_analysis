# Perceived Healthiness and Nutritional Content in Recipes

*Author: Kate Oberhauser*

## Introduction
> “Eat food. Not too much. Mostly plants.” — Michael Pollan

In his 2007 essay "Unhappy Meals", journalist Michael Pollan offered this simple advice on healthy eating as a counter to reductionist nutritionism, the tendency to fixate on individual nutrients rather than the overall quality of the food we consume. Nearly twenty years later, however, nutrition guidance feels increasingly complicated and contradictory, as our tendency to focus on a single nutrient as the key to health persists. Each year seems to bring a new food fad, including the Atkins diet, the Mediterranean diet, and the ketogenic diet. Each celebrates one nutrient while demonizing another. In today’s age of protein, products ranging from cereal and chips to coffee and even Pop-Tarts are marketed with added protein. As different macronutrients rise and fall in popularity, the definition of what counts as “healthy” becomes increasingly blurry amid the constant shifts of dietary trends.

This project investigates **which nutritional characteristics are most associated with recipes labeled as “healthy” on Food.com.** In particular, it examines which features of a recipe contribute to it being perceived as healthy and how that perception relates to its nutrient composition, focusing on whether individual nutrients or combinations of nutrients are more strongly associated with the “healthy” label.

To explore this question, I analyze two datasets from Food.com, `recipes` and `interactions`, containing recipe postings and ratings.

`recipes` contains 83,782 rows and 12 columns:

| Column | Description |
|-------|-------------|
| `name` | Recipe name |
| `id` | Recipe ID |
| `minutes` | Minutes to prepare recipe |
| `contributor_id` | User ID of recipe author |
| `submitted` | Date of recipe submission |
| `tags` | Food.com tags for recipe |
| `nutrition` | Nutrition info: calories, fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbs (PDV) |
| `n_steps` | Number of steps in recipe |
| `steps` | Description of recipe steps |
| `description` | User-provided description |

`interactions` contains 73,1927 rows and 5 columns:

| Column | Description |
|-------|-------------|
| `user_id` | User ID |
| `recipe_id` | Recipe ID |
| `date` | Date of review |
| `rating` | Rating given (1-5) |
| `review` | Review commentary |

In my analysis, I will mainly focus on the `nutrition` and `tags` columns in the `recipes` data. As described in the table above, `nutrition` contains the amounts of each nutrient in each recipe, and `tags` contains labels associated with a recipe such as "healthy", "low-fat", and "low-carb". Looking at how these features interact allows me to uncover patterns as to which nutritional characteristics "healthy" recipes tend to have.

## Data Cleaning and Exploratory Data Analysis
### Data Cleaning
The data was cleaned using the following steps:
1. **Left merge the recipes dataset with the ratings dataset.**
This creates a single DataFrame containing both datasets by matching each recipe in `recipes` with its corresponding ratings in `interactions`.
2. **Replace ratings of 0 with NaN.**
Valid ratings range from 1 to 5, where 1 indicates a very negative rating and 5 indicates a very positive rating. A rating of 0 represents a missing rating rather than an actual score, so these values are replaced with NaN.
3. **Compute the average rating for each recipe.**
The mean rating is calculated for each recipe using the ratings from the merged dataset. Because `NaN` values are ignored in mean calculations, missing ratings do not affect the result.
4. **Merge the average ratings back into the `recipes` dataset.**
The resulting DataFrame contains one row per recipe along with its average rating. This DataFrame is used for the remainder of the analysis.
The resulting dataset contains 13 columns (the original 12 columns from `recipes` plus `avg_rating`).
5. **Convert the values in the nutrition column into separate numeric nutrient columns.**
In the original dataset, the nutrition column is stored as a string that looks like a list containing seven values. Each value corresponds to a different nutrient measurement.
To make these values usable for analysis, the brackets were first removed from the string and the remaining values were split on commas. This produced seven separate columns representing:
- calories
- total_fat
- sugar
- sodium
- protein
- saturated_fat
- carbohydrates

These columns were converted to float values and appended to the DataFrame. The original nutrition column was then dropped.
After this transformation, the dataset increased from 13 columns to 19 columns, since the single nutrition column was replaced with seven separate nutrient columns. The first few rows of the new nutrition columns are shown below.

|   calories |   total_fat |   sugar |   sodium |   protein |   saturated_fat |   carbs |
|-----------:|------------:|--------:|---------:|----------:|----------------:|--------:|
|      138.4 |          10 |      50 |        3 |         3 |              19 |       6 |
|      595.1 |          46 |     211 |       22 |        13 |              51 |      26 |
|      194.8 |          20 |       6 |       32 |        22 |              36 |       3 |
|      878.3 |          63 |     326 |       13 |        20 |             123 |      39 |
|      267   |          30 |      12 |       12 |        29 |              48 |       2 |

6. **Remove rows with invalid nutrition data.**
26 rows contained zero calories but non-zero values for sodium. Inspecting these rows revealed that many corresponded to non-food items (e.g., garbage disposal cleaner and dishwasher detergent), low-calorie items that have no nutritional value such as salt, or recipes where the zero-calorie nutrition information was clearly invalid (e.g., easy microwave popcorn, indian griddle flatbreads). Because these values are inconsistent and cannot represent real food items, these 26 rows were removed from the dataset.
7. **Remove extreme outliers in the nutrition columns.**
Inspection of the distributions of the 7 nutrition variables revealed that the maximum values in each column were substantially larger than the 99th percentile. This indicates that a small number of recipes contain extremely large nutritional values, likely due to data entry errors or recipes that represent multiple servings rather than a single serving. Because these extreme values represent only a small proportion of the dataset and the number of servings per recipe is not provided, recipes with nutrition values above the 99th percentile for each nutrient were removed from the dataset.
8. **Create nutrient density variables.**
Except for calories, all other nutrient columns (total_fat, saturated_fat, sugar, protein, carbohydrates, sodium) are in PDV (percent daily value) units. To make these values comparable across recipes of different calorie levels, nutrient density variables were created by dividing each nutrient column by the calorie value in each recipe. These density features represent the amount of each nutrient per calorie, allowing for more meaningful comparisons between recipes with different calorie levels.
The following columns were appended to the DataFrame:

- `total_fat_density`
- `sugar_density`
- `sodium_density`
- `protein_density`
- `saturated_fat_density`
- `carbs_density`

9. **Add an 'is_healthy' column.**
I added a binary `is_healthy` column indicating whether the recipe’s tag list contains “healthy,” enabling grouped analysis of nutritional differences between recipes tagged as healthy and those that are not.

Results: The cleaned DataFrame has 78,125 rows and 26 columns.

| column                | dtype   |
|:----------------------|:--------|
| name                  | object  |
| id                    | int64   |
| minutes               | int64   |
| contributor_id        | int64   |
| submitted             | object  |
| tags                  | object  |
| n_steps               | int64   |
| steps                 | object  |
| description           | object  |
| ingredients           | object  |
| n_ingredients         | int64   |
| avg_rating            | float64 |
| calories              | float64 |
| total_fat             | float64 |
| sugar                 | float64 |
| sodium                | float64 |
| protein               | float64 |
| saturated_fat         | float64 |
| carbs                 | float64 |
| total_fat_density     | float64 |
| sugar_density         | float64 |
| sodium_density        | float64 |
| protein_density       | float64 |
| saturated_fat_density | float64 |
| carbs_density         | float64 |
| is_healthy            | bool    |

 Head of Final DataFrame:

<div style="overflow-x:auto;">
  <table border="1" class="dataframe" style="white-space: nowrap;">
    <thead>
      <tr style="text-align: right;">
        <th>name</th>
        <th>id</th>
        <th>minutes</th>
        <th>contributor_id</th>
        <th>submitted</th>
        <th>tags</th>
        <th>n_steps</th>
        <th>steps</th>
        <th>description</th>
        <th>ingredients</th>
        <th>n_ingredients</th>
        <th>avg_rating</th>
        <th>calories</th>
        <th>total_fat</th>
        <th>sugar</th>
        <th>sodium</th>
        <th>protein</th>
        <th>saturated_fat</th>
        <th>carbs</th>
        <th>total_fat_density</th>
        <th>sugar_density</th>
        <th>sodium_density</th>
        <th>protein_density</th>
        <th>saturated_fat_density</th>
        <th>carbs_density</th>
        <th>is_healthy</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>1 brownies in the world    best ever</td>
        <td>333281</td>
        <td>40</td>
        <td>985201</td>
        <td>2008-10-27</td>
        <td>['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts', 'lunch', 'snacks', 'cookies-and-brownies', 'chocolate', 'bar-cookies', 'brownies', 'number-of-servings']</td>
        <td>10</td>
        <td>['heat the oven to 350f and arrange the rack in the middle', 'line an 8-by-8-inch glass baking dish with aluminum foil', 'combine chocolate and butter in a medi...</td>
        <td>these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you'll ever make.....sereiously! there's no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they're pure heaven!</td>
        <td>['bittersweet chocolate', 'unsalted butter', 'eggs', 'granulated sugar', 'unsweetened cocoa powder', 'vanilla extract', 'brewed espresso', 'kosher salt', 'all-purpose flour']</td>
        <td>9</td>
        <td>4.0</td>
        <td>138.4</td>
        <td>10.0</td>
        <td>50.0</td>
        <td>3.0</td>
        <td>3.0</td>
        <td>19.0</td>
        <td>6.0</td>
        <td>0.07</td>
        <td>0.36</td>
        <td>0.02</td>
        <td>0.02</td>
        <td>0.14</td>
        <td>4.34e-02</td>
        <td>False</td>
      </tr>
      <tr>
        <td>1 in canada chocolate chip cookies</td>
        <td>453467</td>
        <td>45</td>
        <td>1848091</td>
        <td>2011-04-11</td>
        <td>['60-minutes-or-less', 'time-to-make', 'cuisine', 'preparation', 'north-american', 'for-large-groups', 'canadian', 'british-columbian', 'number-of-servings']</td>
        <td>12</td>
        <td>['pre-heat oven the 350 degrees f', 'in a mixing bowl , sift together the flours and baking powder', 'set aside', 'in another mixing bowl , blend together the s...</td>
        <td>this is the recipe that we use at my school cafeteria for chocolate chip cookies. they must be the best chocolate chip cookies i have ever had! if you don't have margarine or don't like it, then just use butter (softened) instead.</td>
        <td>['white sugar', 'brown sugar', 'salt', 'margarine', 'eggs', 'vanilla', 'water', 'all-purpose flour', 'whole wheat flour', 'baking soda', 'chocolate chips']</td>
        <td>11</td>
        <td>5.0</td>
        <td>595.1</td>
        <td>46.0</td>
        <td>211.0</td>
        <td>22.0</td>
        <td>13.0</td>
        <td>51.0</td>
        <td>26.0</td>
        <td>0.08</td>
        <td>0.35</td>
        <td>0.04</td>
        <td>0.02</td>
        <td>0.09</td>
        <td>4.37e-02</td>
        <td>False</td>
      </tr>
      <tr>
        <td>412 broccoli casserole</td>
        <td>306168</td>
        <td>40</td>
        <td>50969</td>
        <td>2008-05-30</td>
        <td>['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']</td>
        <td>6</td>
        <td>['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray , set aside', 'in a large bowl mix together broccoli , soup , one cup of cheese ...</td>
        <td>since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008</td>
        <td>['frozen broccoli cuts', 'cream of chicken soup', 'sharp cheddar cheese', 'garlic powder', 'ground black pepper', 'salt', 'milk', 'soy sauce', 'french-fried onions']</td>
        <td>9</td>
        <td>5.0</td>
        <td>194.8</td>
        <td>20.0</td>
        <td>6.0</td>
        <td>32.0</td>
        <td>22.0</td>
        <td>36.0</td>
        <td>3.0</td>
        <td>0.10</td>
        <td>0.03</td>
        <td>0.16</td>
        <td>0.11</td>
        <td>0.18</td>
        <td>1.54e-02</td>
        <td>False</td>
      </tr>
      <tr>
        <td>millionaire pound cake</td>
        <td>286009</td>
        <td>120</td>
        <td>461724</td>
        <td>2008-02-12</td>
        <td>['time-to-make', 'course', 'cuisine', 'preparation', 'occasion', 'north-american', 'desserts', 'american', 'southern-united-states', 'dinner-party', 'holiday-event', 'cakes', 'dietary', 'christmas', 'thanksgiving', 'low-sodium', 'low-in-something', 'taste-mood', 'sweet', '4-hours-or-less']</td>
        <td>7</td>
        <td>['freheat the oven to 300 degrees', 'grease a 10-inch tube pan with butter , dust the bottom and sides with flour , and set aside', 'in a large mixing bowl , cr...</td>
        <td>why a millionaire pound cake?  because it's super rich!  this scrumptious cake is the pride of an elderly belle from jackson, mississippi.  the recipe comes from "the glory of southern cooking" by james villas.</td>
        <td>['butter', 'sugar', 'eggs', 'all-purpose flour', 'whole milk', 'pure vanilla extract', 'almond extract']</td>
        <td>7</td>
        <td>5.0</td>
        <td>878.3</td>
        <td>63.0</td>
        <td>326.0</td>
        <td>13.0</td>
        <td>20.0</td>
        <td>123.0</td>
        <td>39.0</td>
        <td>0.07</td>
        <td>0.37</td>
        <td>0.01</td>
        <td>0.02</td>
        <td>0.14</td>
        <td>4.44e-02</td>
        <td>False</td>
      </tr>
      <tr>
        <td>2000 meatloaf</td>
        <td>475785</td>
        <td>90</td>
        <td>2202916</td>
        <td>2012-03-06</td>
        <td>['time-to-make', 'course', 'main-ingredient', 'preparation', 'main-dish', 'potatoes', 'vegetables', '4-hours-or-less', 'meatloaf', 'simply-potatoes2']</td>
        <td>17</td>
        <td>['pan fry bacon , and set aside on a paper towel to absorb excess grease', 'mince yellow onion , red bell pepper , and add to your mixing bowl', 'chop garlic an...</td>
        <td>ready, set, cook! special edition contest entry: a mediterranean flavor inspired meatloaf dish. featuring: simply potatoes - shredded hash browns, egg, bacon, spinach, red bell pepper, and goat cheese.</td>
        <td>['meatloaf mixture', 'unsmoked bacon', 'goat cheese', 'unsalted butter', 'eggs', 'baby spinach', 'yellow onion', 'red bell pepper', 'simply potatoes shredded hash browns', 'fresh garlic', 'kosher salt', 'white pepper', 'olive oil']</td>
        <td>13</td>
        <td>5.0</td>
        <td>267.0</td>
        <td>30.0</td>
        <td>12.0</td>
        <td>12.0</td>
        <td>29.0</td>
        <td>48.0</td>
        <td>2.0</td>
        <td>0.11</td>
        <td>0.04</td>
        <td>0.04</td>
        <td>0.11</td>
        <td>0.18</td>
        <td>7.49e-03</td>
        <td>False</td>
      </tr>
    </tbody>
  </table>
</div>

### Univariate Analysis
Oftentimes, we interpret healthy foods as being low calorie. This is likely due to the nature of whole, unprocessed foods being calorically less dense than fast food and other processed foods. As an overview of our data, here is the calorie distribution within the recipe dataset.

<iframe
  src="assets/calorie_distribution.html"
  width="900"
  height="420"
  frameborder="0"
></iframe>

As the plot shows, the distribution is skewed right, meaning that most of the data is lower calorie.

### Bivariate Analysis

I first examined calories and protein density in a scatter plot:

<iframe
  src="assets/calories_vs_protein_density.html"
  width="900"
  height="420"
  frameborder="0"
></iframe>

There is a clear negative relationship between calories and protein density. Lower-calorie recipes tend to have higher protein density, while higher-calorie recipes generally have lower protein per calorie, suggesting that calorie-dense foods are often less protein-efficient.

I also plotted calories vs saturated fat density

<iframe
  src="assets/calories_vs_saturated_fat.html"
  width="900"
  height="420"
  frameborder="0"
></iframe>

Saturated fat density decreases as calories increase, with high-calorie recipes showing consistently lower saturated fat per calorie. In contrast, low-calorie recipes exhibit a wider range of saturated fat density.

### Interesting Aggregates

To compare recipes labeled as healthy with those that are not, I grouped the cleaned dataset by `is_healthy` and computed the mean of both the nutrient density variables and the original nutrition variables.

#### Average Nutrient Densities by Healthy Label

| is_healthy   |   total_fat_density |   sugar_density |   sodium_density |   protein_density |   saturated_fat_density |   carbs_density |
|:-------------|--------------------:|----------------:|-----------------:|------------------:|------------------------:|----------------:|
| False        |               0.075 |           0.152 |            0.078 |             0.08  |                   0.094 |           0.029 |
| True         |               0.038 |           0.244 |            0.109 |             0.076 |                   0.035 |           0.047 |

This table shows that recipes tagged as healthy tend to have lower total fat density and lower saturated fat density than recipes not tagged as healthy. However, healthy recipes also have higher sugar, sodium, and carbohydrate density on average, suggesting that “healthy” labels on Food.com do not simply correspond to being lower in every nutrient often perceived as unhealthy.

#### Average Nutrition Values by Healthy Label

| is_healthy   |   calories |   total_fat |   sugar |   sodium |   protein |   saturated_fat |   carbs |
|:-------------|-----------:|------------:|--------:|---------:|----------:|----------------:|--------:|
| False        |    348.244 |      28.077 |  43.116 |   21.31  |    29.678 |          35.079 |   9.672 |
| True         |    268.467 |      11.239 |  54.281 |   17.136 |    22.106 |          10.467 |  12.5   |

Looking at the original nutrition variables, healthy recipes have fewer calories, less total fat, and less saturated fat on average. At the same time, they have more sugar and carbohydrates on average, which reinforces that recipes labeled healthy may be lower in fat rather than uniformly lower in all nutrients.

## Assessment of Missingness
### MNAR Analysis
There are three columns in the merged dataset with missing values: `avg_rating`, `description`, and `name`. After cleaning, most missing values remain in `avg_rating` and `description`, so I focus my analysis on these columns.

I believe the description column is likely MNAR (Missing Not At Random). Recipe descriptions are optional and written by contributors when submitting a recipe. It is plausible that contributors are more likely to omit a description if it would be very short, uninformative, or redundant. In this case, the probability that a description is missing depends on the unobserved value of the description itself, which is characteristic of MNAR data.

If additional data were available, such as contributor activity level, number of recipes posted, or engagement metrics, this might help explain the missingness. Conditioning on such variables could make the missingness MAR (Missing At Random) instead of MNAR.

### Missingness Dependency

To assess whether the missingness of `avg_rating` depends on observed variables, I performed permutation tests using `sugar_density` and `saturated_fat_density`.

I created an indicator variable:
- `avg_rating_missing = True` if `avg_rating` is missing
- `avg_rating_missing = False` otherwise

For both tests below, I used the **absolute difference in means** as the test statistic. This is appropriate because I am testing whether the distributions differ at all, not whether one group is specifically larger or smaller than the other.

### Sugar Density and Missingness

<iframe
  src="assets/sugar_density_dist.html"
  width="900"
  height="420"
  frameborder="0"
></iframe>

The distributions of sugar density for recipes with missing and non-missing ratings appear similar, with only a slight shift. The observed difference in mean sugar density is approximately 0.0062.

**Null Hypothesis:**  
The distribution of sugar density is the same for recipes with missing and non-missing average ratings.

**Alternative Hypothesis:**  
The distribution of sugar density differs between the two groups.

**Test Statistic:**  
Absolute difference in mean sugar density.

**Significance Level:**  
0.05

<iframe
  src="assets/sugar_permutation.html"
  width="900"
  height="420"
  frameborder="0"
></iframe>

The permutation test produced a p-value of 0.1384. Since this value is greater than 0.05, I fail to reject the null hypothesis. This suggests that the missingness of `avg_rating` does not depend on sugar density.

### Saturated Fat Density and Missingness

<iframe
  src="assets/satfat_density_dist.html"
  width="900"
  height="420"
  frameborder="0"
></iframe>

The observed difference in mean saturated fat density is approximately 0.0036

**Null Hypothesis:**  
The distribution of saturated fat density is the same for recipes with missing and non-missing average ratings.

**Alternative Hypothesis:**  
The distribution of saturated fat density differs between the two groups.

**Test Statistic:**  
Absolute difference in mean saturated fat density.

**Significance Level:**  
0.05

<iframe
  src="assets/satfat_permutation.html"
  width="900"
  height="420"
  frameborder="0"
></iframe>

The permutation test produced a p-value of 0.0058. Since this value is less than 0.05, I reject the null hypothesis. This provides evidence that the missingness of `avg_rating` depends on saturated fat density.

### Missingness Conclusion:
The results indicate that missingness in `avg_rating` is not Missing Completely At Random (MCAR). While sugar density does not show a significant relationship, saturated fat density does. This suggests the missingness of `avg_rating` is not MCAR and may be MAR, since it appears to depend on at least one observed variable.

## Hypothesis Testing
Circling back to my question of what makes a recipe labeled healthy, I tested whether recipes tagged as healthy tend to have fewer calories than recipes that are not tagged as healthy.

- **Null Hypothesis:** Recipes with the healthy tag have the same average calories as recipes without the healthy tag. Any observed difference is due to random chance.  
- **Alternative Hypothesis:** Recipes with the healthy tag have a lower average calorie count than recipes without the healthy tag.  
- **Test Statistic:** Difference in mean calories (Healthy − Not Healthy)
- **Significance Level:** 0.05

### Results

The observed difference in mean calories was **-79.78**, indicating that healthy recipes have fewer calories on average.

A permutation test was conducted with 5000 repetitions. The resulting p-value was **0.0**.

<iframe
  src="assets/calories_permutation.html" 
  width="100%"
  height="420"
></iframe>

### Conclusion
Since the p-value is less than 0.05, I reject the null hypothesis. This provides evidence that recipes labeled as healthy tend to have lower calorie counts than those that are not labeled as healthy. One possible explanation is that recipes perceived as healthy may include more whole-food ingredients, such as fruits and vegetables, which are generally lower in calories, although this was not directly tested in the analysis.

## Framing a Prediction Problem
My goal is to predict whether a recipe is labeled as **“healthy”** based on its nutritional and structural characteristics.  

This is a **binary classification problem**, where each recipe is classified as either:  
- Healthy (`True`)  
- Not healthy (`False`)  

The response variable is `is_healthy`, which indicates whether a recipe has the “healthy” tag. This aligns with the central theme of the project: understanding how nutritional properties relate to perceived healthiness.  

From earlier analysis, we observed relationships between the presence of the healthy tag and calories, as well as associations between calories and other nutritional features such as saturated fat, sugar, and protein. Therefore, the model will use only recipe-level features available at submission time, such as nutrient values, nutrient densities, number of ingredients, number of steps, and tag-derived indicators. It will not use post-publication variables such as `avg_rating`, since they are only available after users interact with the recipe and would not be known at prediction time.  

When evaluating the model, I will focus on the **F1-score**. This metric is appropriate because it balances precision (the proportion of predicted positive instances that are correctly classified) and recall (the proportion of actually positive instances that are correctly classified), making it more informative than accuracy when classes are imbalanced. In this dataset, there are significantly more recipes without the healthy (63124) tag than with it (15001), so F1-score ensures the model performs well in identifying healthy recipes without over-predicting them.

## Baseline Model

### Model Description  

For the baseline model, I used a **Random Forest classifier** implemented within an sklearn Pipeline.  

The model uses the following features:  
- `calories` (quantitative)  
- `protein_density` (quantitative)  
- `total_fat_density` (quantitative)  

All features are numerical, so no encoding was required.  

### Training and Evaluation  
The data was split into training and testing sets using a 75/25 split. The model was trained on the training set and evaluated using **F1-score**, along with precision and recall.  
- **Training F1-score:** 0.9788  
- **Test F1-score:** 0.4322  
- **Test Precision:** 0.4974  
- **Test Recall:** 0.3822  

### Interpretation  
The baseline model performs very well on the training data but significantly worse on the test data. The large gap between the training F1-score (0.9788) and test F1-score (0.4322) indicates that the model is **overfitting** to the training data and does not generalize well to unseen data.  

While the features used (calories, protein density, and fat density) provide some predictive signal, they are not sufficient on their own to accurately classify whether a recipe is healthy. Additionally, the relatively low recall suggests that the model is missing many recipes that are actually labeled as healthy.  

Overall, this baseline model is not strong, but it provides a useful reference point for improvement in the final model.  

## Final Model
To improve upon the baseline model, I introduced additional features that better capture how recipes are labeled as healthy.  

In addition to the baseline features (`calories`, `protein_density`, `total_fat_density`), I added:  

- `sugar_density` (quantitative): Sugar content is an important nutritional factor that may influence whether a recipe is perceived as healthy.  
- `low_fat_tag` (nominal, boolean): Indicates whether the recipe explicitly includes a low-fat label.  
- `low_carb_tag` (nominal, boolean): Indicates whether the recipe includes a low-carb label.  
- `diet_tag` (nominal, boolean): Captures broader dietary-related labels such as low-calorie, low-sodium, or low-cholesterol.  

These engineered features are useful because they incorporate both nutritional composition and labels from the `tags` column, which are directly related to how recipes are categorized as healthy. In particular, I expect the three additional tag features to improve the model because I suspect nutritional metrics such as `low_fat` or `low_carb` are highly correlated with the perceived healthiness of a recipe.

### Model and Hyperparameter Tuning  
I used a **Random Forest classifier** within a Pipeline and performed hyperparameter tuning using **GridSearchCV** with 5-fold cross-validation.  
A Random Forest model is well-suited for this task because it can capture complex relationships between nutritional features and tag-based indicators while reducing overfitting compared to a single decision tree by averaging predictions across many trees.
GridSearchCV was used to evaluate combinations of hyperparameters and select the model that maximizes cross-validated performance, helping ensure the final model generalizes well to unseen data.

The following hyperparameters were tuned:  
- `n_estimators`: number of trees in the forest  
- `max_depth`: controls model complexity and helps prevent overfitting  
- `min_samples_split`: prevents overly specific splits and improves generalization  

The best-performing hyperparameters were:  
- `n_estimators = 200`  
- `max_depth = None`  
- `min_samples_split = 20`  

### Model Performance  
- **Training F1-score:** 0.8864  
- **Test F1-score:** 0.8020  
- **Test Precision:** 0.8555  
- **Test Recall:** 0.7548

### Interpretation  

The final model shows a substantial improvement over the baseline model, with the test F1-score increasing by 0.3698 (from 0.4322 to 0.8020), indicating significantly better performance on unseen data.  

Additionally, the gap between training and test performance is much smaller than in the baseline model, suggesting that the final model generalizes well and is less prone to overfitting.  

The improvement is likely due to the inclusion of additional nutritional features and engineered tag-based features, which provide more informative signals about whether a recipe is labeled as healthy. In particular, features derived from tags (e.g., `low_fat_tag`, `diet_tag`) directly capture how recipes are categorized, making them highly predictive. This suggests that perceived healthiness on Food.com may be driven by the presence or absence of specific nutrients or dietary labels, rather than by an overall balance of nutritional content.

### Conclusion  

The final model is a strong improvement over the baseline model, achieving both higher predictive performance and better generalization. By combining feature engineering with hyperparameter tuning, the model more effectively captures the relationship between recipe characteristics and the “healthy” label.

## Fairness Analysis
Given the importance of calorie-related features, it is also important to evaluate whether the model performs differently across recipes with varying calorie levels. To evaluate fairness, I compared model performance across two groups:  

- **Low-calorie recipes:** recipes with calories below or equal to the median  
- **High-calorie recipes:** recipes with calories above the median  

### Evaluation Metric  

I used **recall parity** as the evaluation metric. Recall measures the proportion of truly healthy recipes that are correctly identified by the model. This allows us to assess whether the model is better at identifying healthy recipes in one group versus another.  

### Hypotheses  

- **Null Hypothesis:** The model is fair. Recall is the same for low-calorie and high-calorie recipes, and any observed difference is due to chance.  
- **Alternative Hypothesis:** The model is more unfair. It's recall for low-calorie recipes is higher than for high-calorie recipes.

### Test Statistic  
Difference in recall: Recall (low-calorie group) − Recall (high-calorie group)  

Observed difference: **0.065**  

### Results  

A permutation test was conducted by randomly shuffling the calorie group labels. The resulting p-value was **0.0**, indicating that the observed difference is highly unlikely to occur by chance.  

<iframe 
  src="assets/fairness_permutation.html"
  width="100%" 
  height="420"
></iframe>

---

### Conclusion  

Since the p-value is less than 0.05, we reject the null hypothesis. There is strong evidence that the model achieves higher recall for low-calorie recipes than for high-calorie recipes.  

This suggests that the model is more effective at identifying healthy recipes within the low-calorie group. One possible explanation is that lower-calorie recipes more closely align with patterns the model associates with healthiness, making them easier to classify correctly. 