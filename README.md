# 2022 LOL Analysis

## Introduction

This project conducts an in-depth analysis on the 2022 public match data for League of Legends which can be found [here](https://oracleselixir.com/tools/downloads). The dataset contains 148992 observations; for the sake of this project we will only use the aggregated team stats and we will be focusing on the following columns

- `result`: The outcome of the game. (`int`)
- `league`: The league the match was played in. (`str`)
- `side`: The side that the team played on. (`str`)
- `teamkills`: The total number of kills from the team. (`int`)
- `teamdeaths`: The total number of deaths from the team. (`int`)
- `doublekills`: The total number of double kills earned from the team. (`int`)
- `triplekills`: The total number of triple kills earned from the team. (`int`)
- `quadrakills`: The total number of quadra kills earned from the team. (`int`)
- `pentakills`: The total number of penta kills earned from the team. (`int`)
- `dragons`: The total number of dragons captured by the team. (`int`)
- `heralds`: The total number of heralds captured by the team. (`int`)
- `barons`: The total number of barons captured by the team. (`int`)
- `goldat15`/`opp_goldat15`: The total gold earned by the team/opponent at the 15 minute mark. (`int`)
- `xpat15`/`opp_xpat15`: The total xp earned by the team/opponent at the 15 minute mark. (`int`)
- `csat15`/`opp_csat15`: The total creep score earned by the team/opponent at the 15 minute mark. (`int`)

In this project, we aim to answer the question: **Does `side` have a significant impact on the outcome of a game?**

We will explore the importance these factors have on the outcome of a game with an emphasis on `side`. Additionally, we will explore the differences between matches in different leagues such as the LPL and LCK by comparing metrics that evaluate the action of a game. 

## Data Cleaning and EDA

### Data Cleaning Steps

1. Filter the rows corresponding to team stats (`'position'` == 'team').
2. Filter the rows corresponding to matches played in regional leagues ('LCK, 'LPL', ...).
3. Removed columns relating to individual players (`'player name'`, `'damage share'`, `'kills'`, ...).
4. Add columns corresponding to differences between team and opposing team (`'killsdif'`, `'towersdif`', ...).
5. Change appropriate columns to `bool` type (`'result'`, `'firstblood`', `'firstdragon`', `'firsttower`', ...).
6. Replace `'side'` with `'red_side'` column (`True` if on played on Red side and `False` if played on Blue).
7. Insert `'multikills'` column (sum of `'doublekills'`, `'triplekills'`, `'quadrakills'`, and `'pentakills'`).
8. Insert `'objectives'` column (sum of `'dragons'`, `'heralds'`, `'baronds'`).
9. Combine `'pick1'`, `'pick2`', ..., `'pick5'` into a single column `'picks'` corresponding to a list of the champions picked.
10. Combine `'ban1'`, `'ban2`', ..., `'ban5'` into a single column `'bans'` corresponding to a list of the champions banned.
11. Filtered rows that had complete data (`'datacompleteness'` == 'complete').
12. Removed irrelevant columns (`'url'`, `'datacompleteness'`, `'year'`, `'split'`, `'void_grubs'`, ...)
13. Filled na values of `'dragons (type unknown)'` with 0.

