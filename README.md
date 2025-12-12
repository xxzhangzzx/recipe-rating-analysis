# Recipe Ratings and Cooking Time
This site presents my DSC 80 final project on Food.com recipes and ratings.

## Introduction

This project uses the Food.com **recipes** dataset together with a **user interactions** dataset.  
The recipes dataset contains one row per recipe posted on Food.com since 2008, for a total of **83,782 recipes**. The interactions dataset contains one row per user interaction (a rating and, sometimes, a written review) for a recipe, for a total of **731,927 interactions**.

To study recipe quality at the recipe level, I first merge the two datasets on recipe ID, replace ratings of 0 with missing values (since 0 usually means “no real rating given”), and compute the **average rating** for each recipe. Then I merge these average ratings back into the recipes table and apply some basic cleaning: I keep only recipes with a non-missing average rating and restrict cooking times to a reasonable range (between 1 minute and 24 hours). After these steps, my cleaned analysis dataset contains **80,630 recipes**, each with an average user rating and basic information about the recipe.

The central question of this project is:

> **What is the relationship between the cooking time (`minutes`) and the average rating (`avg_rating`) of recipes on Food.com?**

This question matters because home cooks care about both taste and convenience. If very time-consuming recipes are not rated much higher than quick recipes, then spending extra time in the kitchen might not be “worth it.” On the other hand, if longer recipes consistently earn better ratings, that suggests a trade-off between time and perceived quality.

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