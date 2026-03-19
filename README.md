# Recipes Analysis – What Actually Constitutes “Healthy”?

*Author: Kate Oberhauser*

## Introduction
> “Eat food. Not too much. Mostly plants.” — Michael Pollan

In his 2007 essay "Unhappy Meals", journalist Michael Pollan offered this simple advice on healthy eating as a counter to reductionist nutritionism, the tendency to fixate on individual nutrients rather than the overall quality of the food we consume. Nearly twenty years later, however, nutrition guidance feels increasingly complicated and contradictory, as our tendency to focus on a single nutrient as the key to health persists. Each year seems to bring a new food fad, including the Atkins diet, the Mediterranean diet, and the ketogenic diet. Each celebrates one nutrient while demonizing another. In today’s age of protein, products ranging from cereal and chips to coffee and even Pop-Tarts are marketed with added protein. As different macronutrients rise and fall in popularity, the definition of what counts as “healthy” becomes increasingly blurry amid the constant shifts of dietary trends.

With this in mind, this project investigates **what actually makes a recipe healthy and how a recipe’s healthiness relates to the nutrients it contains**. Specifically, I examine whether certain nutrients are more strongly associated with recipes labeled as healthy, or whether combinations of nutrients better explain these patterns. 

To explore this question, I analyze two datasets from Food.com, `recipes` and `interactions`, containing recipe postings and ratings.

`recipes` contains 83782 rows and 12 columns:

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

`interactions` contains 731927 rows and 5 columns:

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
The resulting dataset contains 13 columns (the original 12 columns from `recipes` plus `avg_rating`). Because there are quite a few columns, a preview of the first 5 rows of a few relevant columns in the resulting dataset are shown below.

| name                                 |   minutes | tags                                      | ingredients                                                                      | nutrition                                     |   avg_rating |
|:-------------------------------------|----------:|:------------------------------------------|:---------------------------------------------------------------------------------|:----------------------------------------------|-------------:|
| 1 brownies in the world    best ever |        40 | 60-minutes-or-less, time-to-make, course  | bittersweet chocolate, unsalted butter, eggs, granulated sugar                   | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0]      |            4 |
| 1 in canada chocolate chip cookies   |        45 | 60-minutes-or-less, time-to-make, cuisine | white sugar, brown sugar, salt, margarine                                        | [595.1, 46.0, 211.0, 22.0, 13.0, 51.0, 26.0]  |            5 |
| 412 broccoli casserole               |        40 | 60-minutes-or-less, time-to-make, course  | frozen broccoli cuts, cream of chicken soup, sharp cheddar cheese, garlic powder | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0]     |            5 |
| millionaire pound cake               |       120 | time-to-make, course, cuisine             | butter, sugar, eggs, all-purpose flour                                           | [878.3, 63.0, 326.0, 13.0, 20.0, 123.0, 39.0] |            5 |
| 2000 meatloaf                        |        90 | time-to-make, course, main-ingredient     | meatloaf mixture, unsmoked bacon, goat cheese, unsalted butter                   | [267.0, 30.0, 12.0, 12.0, 29.0, 48.0, 2.0]    |            5 |

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

Results: The cleaned DataFrame has 78,125 rows and 25 columns.

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
  <table style="white-space: nowrap;">
    <table border="1" class="dataframe">
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
  </table>
</div>

### Univariate Analysis
Oftentimes, we interpret healthy foods as being low calorie. This is likely due to the nature of whole, unprocessed foods being calorically less dense than fast food and other processed foods. As an overview of our data, here is the calorie distribution within the recipe dataset.

<iframe
  src="assets/calorie_distribution.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

As the plot shows, there distribution is skewed right, meaning that most of the data is lower calorie. Most of the recipes in our dataset are less than 500 calories, as the majority of the data lies to the left of the 500 mark on the x-axis.

### Bivariate Analysis

I first examined calories and protein density in a scatter plot:

<iframe
  src="assets/calories_vs_protein_density.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>

There is a clear negative relationship between calories and protein density. Lower-calorie recipes tend to have higher protein density, while higher-calorie recipes generally have lower protein per calorie, suggesting that calorie-dense foods are often less protein-efficient.

I also plotted calories vs saturated fat density

<iframe
  src="assets/calories_vs_saturated_fat.html"
  width="900"
  height="600"
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
There are 3 columns in the merged dataset with missing values: `avg_rating`, `description`, and `name`. 

I believe the description column may be MNAR. A recipe description is written by the contributor when uploading a recipe, and I suspect some contributors may omit the description if they feel it would be very short or not particularly informative. In this case, the probability that the description is missing depends on the the description itself.

Additional variables such as data on the contributor's activity level and number of recipe postings could make the missingness of the `description` column MAR, or dependent on data other columns. For example, more active, experienced recipe posters may be more likely to include a description in order to provide higher quality recipes for other users, so columns such as the total number of recipes posted by a contributor could make missing descriptions MAR.

Now, I'll look at the missingness of the `avg_rating` column.

## Hypothesis Testing


## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis