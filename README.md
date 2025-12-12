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

## Assessment of Missingness

### NMAR Analysis

I believe that the `review` column is a good candidate for NMAR (Not Missing At Random). In this dataset, a `review` is a free text comment that a user can choose to leave in addition to a numeric rating. Whether someone writes a review is likely driven by their unobserved personal reaction to the recipe. For example, they might only take the time to write something if they absolutely loved the dish or if they had a very negative experience.

This underlying opinion is not recorded elsewhere in the data when the review is missing, and it is not just a simple function of observed variables like `minutes` or `avg_rating`. Because the decision to leave a review depends on the user’s internal motivation, which we do not observe, the missingness mechanism for `review` is likely NMAR.

To make this closer to MAR, I would need extra information about user behavior, such as how often they write reviews across different recipes, or whether they were prompted to leave a comment. These additional variables could help explain why some users skip the review text field and turn the missingness of `review` into something that is more predictable from observed data.

### Missingness Dependency

I focused on the missingness of the `rating` column in the merged dataset. I defined a Boolean variable `rating_missing` and asked whether this missingness depends on other recipe features.

First, I tested dependence on `n_steps`. As the test statistic, I used the absolute difference in the mean `n_steps` between recipes with missing ratings and recipes with observed ratings, and ran a permutation test by repeatedly shuffling the missingness labels. The permutation distribution is shown below, with the observed difference marked in red:

<iframe
  src="assets/rating_missing_nsteps_perm.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

The observed statistic lies far in the upper tail of the permutation distribution, and the empirical p-value is essentially 0. This suggests that the missingness of `rating` is not independent of `n_steps`: recipes with more steps are slightly more likely to have missing ratings.

For comparison, I repeated the same style of test using `minutes` (cooking time) instead of `n_steps`. In that case, the p-value was around 0.12, so I do not have strong evidence that rating missingness depends on cooking time alone.