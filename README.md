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
The resulting dataset contains 13 columns (the original 12 columns from `recipes` plus `avg_rating`). Because there are quite a few columns, a preview of the first 10 rows of a few relevant columns in the resulting dataset are shown below.

| name                                    |   minutes | tags                                        | ingredients                                                                      | nutrition                                      |   avg_rating |
|:----------------------------------------|----------:|:--------------------------------------------|:---------------------------------------------------------------------------------|:-----------------------------------------------|-------------:|
| 1 brownies in the world    best ever    |        40 | 60-minutes-or-less, time-to-make, course    | bittersweet chocolate, unsalted butter, eggs, granulated sugar                   | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0]       |            4 |
| 1 in canada chocolate chip cookies      |        45 | 60-minutes-or-less, time-to-make, cuisine   | white sugar, brown sugar, salt, margarine                                        | [595.1, 46.0, 211.0, 22.0, 13.0, 51.0, 26.0]   |            5 |
| 412 broccoli casserole                  |        40 | 60-minutes-or-less, time-to-make, course    | frozen broccoli cuts, cream of chicken soup, sharp cheddar cheese, garlic powder | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0]      |            5 |
| millionaire pound cake                  |       120 | time-to-make, course, cuisine               | butter, sugar, eggs, all-purpose flour                                           | [878.3, 63.0, 326.0, 13.0, 20.0, 123.0, 39.0]  |            5 |
| 2000 meatloaf                           |        90 | time-to-make, course, main-ingredient       | meatloaf mixture, unsmoked bacon, goat cheese, unsalted butter                   | [267.0, 30.0, 12.0, 12.0, 29.0, 48.0, 2.0]     |            5 |
| 5 tacos                                 |        20 | weeknight, 30-minutes-or-less, time-to-make | ground beef, taco seasoning, taco shells, lettuce                                | [249.4, 26.0, 4.0, 6.0, 39.0, 39.0, 0.0]       |            4 |
| 50 chili   for the crockpot             |       345 | course, main-ingredient, cuisine            | stewing beef, stewing pork, white onion, bell peppers                            | [270.2, 19.0, 26.0, 48.0, 52.0, 21.0, 4.0]     |            5 |
| blepandekager   danish   apple pancakes |        50 | danish, 60-minutes-or-less, time-to-make    | eggs, milk, flour, sugar                                                         | [358.2, 30.0, 62.0, 14.0, 19.0, 54.0, 12.0]    |            5 |
| lplermagrone                            |        50 | 60-minutes-or-less, time-to-make, course    | milk, salt, macaroni, cheese                                                     | [1003.8, 72.0, 21.0, 103.0, 69.0, 143.0, 37.0] |            5 |
| lplermagrone  herdsman s macaroni       |        40 | 60-minutes-or-less, time-to-make, course    | potato, salt water, macaroni, heavy cream                                        | [708.6, 52.0, 19.0, 24.0, 46.0, 104.0, 25.0]   |            5 |

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

The first five rows of these new columns are shown below.

|   total_fat_density |   sugar_density |   sodium_density |   protein_density |   saturated_fat_density |   carbs_density |
| total_fat_density | sugar_density | sodium_density | protein_density | saturated_fat_density | carbs_density |
|------------------:|--------------:|---------------:|----------------:|----------------------:|--------------:|
| 0.07 | 0.36 | 0.02 | 0.02 | 0.14 | 0.04 |
| 0.08 | 0.35 | 0.04 | 0.02 | 0.09 | 0.04 |
| 0.10 | 0.03 | 0.16 | 0.11 | 0.18 | 0.02 |
| 0.07 | 0.37 | 0.01 | 0.02 | 0.14 | 0.04 |
| 0.11 | 0.04 | 0.04 | 0.11 | 0.18 | 0.01 |

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

Insert head of final DataFrame here

### Univariate Analysis
Oftentimes, we interpret healthy foods as being low calorie. This is likely due to the nature of whole, unprocessed foods being calorically less dense than fast food and other processed foods. As an overview of our data, here is the calorie distribution within the recipe dataset.

<iframe
  src="assets/calorie_distribution.html"
  width="900"
  height="600"
  frameborder="0"
></iframe>



## Assessment of Missingness


## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis