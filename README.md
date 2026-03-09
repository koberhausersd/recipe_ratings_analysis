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

| name                                    |   minutes | tags                                        | ingredients                                                                      |   n_ingredients |   avg_rating |
|:----------------------------------------|----------:|:--------------------------------------------|:---------------------------------------------------------------------------------|----------------:|-------------:|
| 1 brownies in the world    best ever    |        40 | 60-minutes-or-less, time-to-make, course    | bittersweet chocolate, unsalted butter, eggs, granulated sugar                   |               9 |            4 |
| 1 in canada chocolate chip cookies      |        45 | 60-minutes-or-less, time-to-make, cuisine   | white sugar, brown sugar, salt, margarine                                        |              11 |            5 |
| 412 broccoli casserole                  |        40 | 60-minutes-or-less, time-to-make, course    | frozen broccoli cuts, cream of chicken soup, sharp cheddar cheese, garlic powder |               9 |            5 |
| millionaire pound cake                  |       120 | time-to-make, course, cuisine               | butter, sugar, eggs, all-purpose flour                                           |               7 |            5 |
| 2000 meatloaf                           |        90 | time-to-make, course, main-ingredient       | meatloaf mixture, unsmoked bacon, goat cheese, unsalted butter                   |              13 |            5 |
| 5 tacos                                 |        20 | weeknight, 30-minutes-or-less, time-to-make | ground beef, taco seasoning, taco shells, lettuce                                |               9 |            4 |
| 50 chili   for the crockpot             |       345 | course, main-ingredient, cuisine            | stewing beef, stewing pork, white onion, bell peppers                            |              22 |            5 |
| blepandekager   danish   apple pancakes |        50 | danish, 60-minutes-or-less, time-to-make    | eggs, milk, flour, sugar                                                         |              10 |            5 |
| lplermagrone                            |        50 | 60-minutes-or-less, time-to-make, course    | milk, salt, macaroni, cheese                                                     |               8 |            5 |
| lplermagrone  herdsman s macaroni       |        40 | 60-minutes-or-less, time-to-make, course    | potato, salt water, macaroni, heavy cream                                        |              10 |            5 |


## Assessment of Missingness

## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis