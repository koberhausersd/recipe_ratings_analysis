# Recipes Analysis – What Actually Constitutes “Healthy”?

*Author: Kate Oberhauser*

## Introduction
> “Eat food. Not too much. Mostly plants.” — Michael Pollan

In his 2007 essay "Unhappy Meals", journalist Michael Pollan offered this simple advice on healthy eating as a counter to reductionist nutritionism, the tendency to fixate on individual nutrients rather than the overall quality of the food we consume. Nearly twenty years later, however, nutrition guidance feels increasingly complicated and contradictory, as our tendency to focus on a single nutrient as the key to health persists. Each year seems to bring a new food fad, including the Atkins diet, the Mediterranean diet, and the ketogenic diet. Each celebrates one nutrient while demonizing another. In today’s age of protein, products ranging from cereal and chips to coffee and even Pop-Tarts are marketed with added protein. As different macronutrients rise and fall in popularity, the definition of what counts as “healthy” becomes increasingly blurry amid the constant shifts of dietary trends.

With this in mind, this project investigates **what actually makes a recipe healthy and how a recipe’s healthiness relates to the nutrients it contains**. Specifically, I examine whether certain nutrients are more strongly associated with recipes labeled as healthy, or whether combinations of nutrients better explain these patterns. 

To explore this question, I analyze two datasets from Food.com, `RAW_recipes` and `interactions`, containing recipe postings and ratings.

`Raw_recipes` contains 83782 rows and 12 columns:

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


## Data Cleaning and Exploratory Data Analysis

## Assessment of Missingness

## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis