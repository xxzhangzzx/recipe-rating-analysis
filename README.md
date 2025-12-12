# Recipe Ratings and Cooking Time
This site presents my DSC 80 final project on Food.com recipes and ratings.

## Introduction

This project uses the Food.com **recipes** dataset together with a **user interactions** dataset.  
The recipes dataset contains one row per recipe posted on Food.com since 2008, for a total of **83,782 recipes**. The interactions dataset contains one row per user interaction (a rating and, sometimes, a written review) for a recipe, for a total of **731,927 interactions**.

To study recipe quality at the recipe level, I first merge the two datasets on recipe ID, replace ratings of 0 with missing values (since 0 usually means “no real rating given”), and compute the **average rating** for each recipe. Then I merge these average ratings back into the recipes table and apply some basic cleaning: I keep only recipes with a non-missing average rating and restrict cooking times to a reasonable range (between 1 minute and 24 hours). After these steps, my cleaned analysis dataset contains **80,630 recipes**, each with an average user rating and basic information about the recipe.

The central question of this project is:

> **What is the relationship between the cooking time (`minutes`) and the average rating (`avg_rating`) of recipes on Food.com?**

This question matters because home cooks care about both taste and convenience. If very time consuming recipes are not rated much higher than quick recipes, then spending extra time in the kitchen might not be “worth it.” On the other hand, if longer recipes consistently earn better ratings, that suggests a trade-off between time and perceived quality.

For this question, the main columns I use are:

- `minutes`: total time (in minutes) required to prepare and cook the recipe.  
- `avg_rating`: the average user rating for the recipe on a 1–5 scale, after treating 0 as a missing rating.  
- `n_steps`: the number of instruction steps in the recipe, which roughly measures how involved the procedure is.  
- `n_ingredients`: the number of distinct ingredients used in the recipe, another indicator of complexity.  
- `calories`: the number of calories per serving, extracted from the `nutrition` field.  
- `protein_pdv`: the percent daily value of protein per serving, also extracted from `nutrition`.  
- `log_minutes`: a log-transformed version of `minutes` that makes the highly skewed cooking-time distribution more symmetric.  
- `log_calories`: a log-transformed version of `calories`, for similar reasons.  

These variables allow me to study both how long recipes take and how “good” they are perceived to be, and to explore whether cooking time by itself is strongly related to user ratings once other recipe characteristics are taken into account.

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

I started from the original `RAW_recipes` and `RAW_interactions` datasets from Food.com. First, I performed a left merge of recipes and interactions on the recipe ID, so that every recipe is kept even if it has no reviews. In the merged table, I replaced ratings of 0 with `NaN`, since a rating of 0 usually corresponds to a user choosing not to rate the recipe rather than a true “zero-star” rating.

Next, I computed the average rating for each recipe and merged this back into the recipes table as a new column called `avg_rating`. I then focused on a cleaned version of this table, `recipes_clean`, in which I:

- Parsed the `nutrition` string column into separate numeric columns such as `calories`, `protein_pdv`, etc.
- Created log-transformed columns `log_minutes` and `log_calories` to make highly skewed distributions more manageable.
- Kept only recipes with non-missing `avg_rating`, `minutes`, `n_steps`, and `n_ingredients`.

All of my later analysis and modeling steps use this cleaned `recipes_clean` DataFrame.

### Univariate Analysis

The first figure below shows the distribution of average recipe ratings (`avg_rating`) across all recipes.

<iframe
  src="assets/avg_rating_hist.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

Most recipes have very high average ratings, with a large spike around 4–5 stars. This suggests that users on Food.com tend to rate recipes positively, and that it is relatively rare for a recipe to receive a low average rating.

The second figure shows the distribution of `log_minutes`, the natural log of the reported cooking time in minutes.

<iframe
  src="assets/log_minutes_hist.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

After the log transformation, the distribution of cooking times becomes much more concentrated and roughly bell-shaped, instead of being extremely right-skewed. Most recipes fall into a moderate time range (roughly 20–60 minutes in original units), with relatively few very short or very long recipes.

### Bivariate Analysis

To explore the relationship between cooking time and average rating more directly, I first plotted a scatter plot of `log_minutes` versus `avg_rating`.

<iframe
  src="assets/minutes_rating_scatter.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

The scatter plot shows that average ratings are high across almost all cooking times, and there is no strong linear relationship between `log_minutes` and `avg_rating`. If anything, very long recipes appear slightly more variable in their ratings, but the overall trend is quite flat.

Next, I binned recipes into four cooking-time categories: `<=30`, `30–60`, `60–120`, and `>120` minutes, and plotted the distribution of `avg_rating` within each bin using box plots.

<iframe
  src="assets/minutes_bin_box.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

The box plots confirm that the median rating is around 4–5 stars for all four cooking-time categories. Differences between the groups are small, which supports the idea that cooking time by itself does not strongly determine how well a recipe is rated.

### Interesting Aggregates

To summarize the relationship between cooking time and ratings numerically, I computed the number of recipes, mean rating, and median rating within each cooking-time category:

| minutes_bin | count |   mean   | median |
|------------:|------:|---------:|-------:|
| <=30        |  36418| 4.64465  |      5 |
| 30-60       |  24570| 4.60655  |      5 |
| 60-120      |  11840| 4.62744  |       5|
| >120        |  7802 | 4.58969  |      5 |

Across all four categories, the mean and median ratings are very similar and remain close to the maximum of 5 stars. This table reinforces what we saw in the visualizations: longer recipes are not systematically rated higher or lower than quicker recipes, at least in this dataset.