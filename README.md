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

1. Filter the rows corresponding to team stats (`'position'` == 'team') because we are focusing on aggregated team data.
2. Filter the rows corresponding to matches played in regional leagues ('LCK, 'LPL', ...) because we will be comparing stats between different leagues.
3. Removed columns relating to individual players (`'player name'`, `'damage share'`, `'kills'`, ...) that aren't related to the team.
4. Add columns corresponding to differences between team and opposing team (`'killsdif'`, `'towersdif`', ...) to use as features when evaluating the state of a game.
5. Change appropriate columns to `bool` type (`'result'`, `'firstblood`', `'firstdragon`', `'firsttower`', ...).
6. Replace `'side'` with `'red_side'` column (`True` if on played on Red side and `False` if played on Blue).
7. Insert `'multikills'` column (sum of `'doublekills'`, `'triplekills'`, `'quadrakills'`, and `'pentakills'`) to use as a feature when evaluating the state of a game.
8. Insert `'objectives'` column (sum of `'dragons'`, `'heralds'`, `'baronds'`) to use as a feature when evaluating the state of a game.
9. Combine `'pick1'`, `'pick2`', ..., `'pick5'` into a single column `'picks'` corresponding to a list of the champions picked for less cluttered columns.
10. Combine `'ban1'`, `'ban2`', ..., `'ban5'` into a single column `'bans'` corresponding to a list of the champions banned for less cluttered columns.
11. Filtered rows that had complete data (`'datacompleteness'` == 'complete') because some observations with 'partial' or 'ignore' completeness were missing key data points that we could not reasonably infer/impute.
12. Removed irrelevant columns (`'url'`, `'datacompleteness'`, `'year'`, `'split'`, `'void_grubs'`, ...) to reduce cluttered columns.
13. Filled na values of `'dragons (type unknown)'` with 0.

The head of the final dataframe is shown below

| league   | teamname     | teamid                                  |   gamelength | result   |   assists |   teamkills |   teamdeaths |   doublekills |   triplekills |   quadrakills |   pentakills | firstblood   |    kpm |   ckpm | firstdragon   |   dragons |   opp_dragons |   elementaldrakes |   opp_elementaldrakes |   infernals |   mountains |   clouds |   oceans |   chemtechs |   hextechs |   dragons (type unknown) |   elders |   opp_elders | firstherald   |   heralds |   opp_heralds | firstbaron   |   barons |   opp_barons | firsttower   |   towers |   opp_towers | firstmidtower   | firsttothreetowers   |   turretplates |   opp_turretplates |   inhibitors |   opp_inhibitors |   damagetochampions |     dpm |   damagetakenperminute |   damagemitigatedperminute |   wardsplaced |    wpm |   wardskilled |   wcpm |   controlwardsbought |   visionscore |   vspm |   totalgold |   earnedgold |   earned gpm |   goldspent |        gspd |   gpr |   total cs |   minionkills |   monsterkills |   monsterkillsownjungle |   monsterkillsenemyjungle |    cspm |   goldat10 |   xpat10 |   csat10 |   opp_goldat10 |   opp_xpat10 |   opp_csat10 |   golddiffat10 |   xpdiffat10 |   csdiffat10 |   killsat10 |   assistsat10 |   deathsat10 |   opp_killsat10 |   opp_assistsat10 |   opp_deathsat10 |   goldat15 |   xpat15 |   csat15 |   opp_goldat15 |   opp_xpat15 |   opp_csat15 |   golddiffat15 |   xpdiffat15 |   csdiffat15 |   killsat15 |   assistsat15 |   deathsat15 |   opp_killsat15 |   opp_assistsat15 |   opp_deathsat15 |   killsdif |   eldersdif |   heraldsdif |   baronsdif |   towersdif |   inhibsdif | red_side   |   multikills |   objectives captured | picks                                                  | bans                                                         |
|:---------|:-------------|:----------------------------------------|-------------:|:---------|----------:|------------:|-------------:|--------------:|--------------:|--------------:|-------------:|:-------------|-------:|-------:|:--------------|----------:|--------------:|------------------:|----------------------:|------------:|------------:|---------:|---------:|------------:|-----------:|-------------------------:|---------:|-------------:|:--------------|----------:|--------------:|:-------------|---------:|-------------:|:-------------|---------:|-------------:|:----------------|:---------------------|---------------:|-------------------:|-------------:|-----------------:|--------------------:|--------:|-----------------------:|---------------------------:|--------------:|-------:|--------------:|-------:|---------------------:|--------------:|-------:|------------:|-------------:|-------------:|------------:|------------:|------:|-----------:|--------------:|---------------:|------------------------:|--------------------------:|--------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|-----------:|---------:|---------:|---------------:|-------------:|-------------:|---------------:|-------------:|-------------:|------------:|--------------:|-------------:|----------------:|------------------:|-----------------:|-----------:|------------:|-------------:|------------:|------------:|------------:|:-----------|-------------:|----------------------:|:-------------------------------------------------------|:-------------------------------------------------------------|
| LCK      | DRX          | oe:team:101f8589e58c724c1dcd5a9c1555277 |         2195 | False    |         9 |           5 |           14 |             0 |             0 |             0 |            0 | True         | 0.1367 | 0.5194 | False         |         1 |             4 |                 1 |                     4 |           0 |           0 |        0 |        1 |           0 |          0 |                        0 |        0 |            0 | True          |         2 |             0 | False        |        0 |            2 | True         |        8 |            9 | True            | True                 |             10 |                  0 |            0 |                2 |               50038 | 1367.78 |                2227.52 |                    2178.86 |           127 | 3.4715 |            57 | 1.5581 |                   55 |           305 | 8.3371 |       63747 |        39978 |      1092.79 |       60525 |  0.00190185 |  2.24 |        nan |           945 |            287 |                     nan |                       nan | 33.6765 |      15121 |    18570 |      330 |          14840 |        18166 |          324 |            281 |          404 |            6 |           0 |             0 |            0 |               0 |                 0 |                0 |      27198 |    30325 |      511 |          22441 |        28785 |          510 |           4757 |         1540 |            1 |           4 |             7 |            1 |               1 |                 1 |                4 |         -9 |           0 |            2 |          -2 |          -1 |          -2 | False      |            0 |                     3 | ['Aphelios', 'Sona', 'Viego', 'Graves', 'Ryze']        | ['Diana', 'Caitlyn', 'Twisted Fate', 'LeBlanc', 'Viktor']    |
| LCK      | Liiv SANDBOX | oe:team:c75f1f337fc5867914749d438a4871d |         2195 | True     |        39 |          14 |            5 |             2 |             0 |             0 |            0 | False        | 0.3827 | 0.5194 | True          |         4 |             1 |                 4 |                     1 |           0 |           1 |        0 |        2 |           0 |          1 |                        0 |        0 |            0 | False         |         0 |             2 | True         |        2 |            0 | False        |        9 |            8 | False           | False                |              0 |                 10 |            2 |                0 |               59071 | 1614.7  |                2236.78 |                    1898.73 |           114 | 3.1162 |            64 | 1.7494 |                   43 |           292 | 7.9818 |       67669 |        43900 |      1200    |       60410 | -0.00190185 | -2.24 |        nan |           986 |            204 |                     nan |                       nan | 32.5285 |      14840 |    18166 |      324 |          15121 |        18570 |          330 |           -281 |         -404 |           -6 |           0 |             0 |            0 |               0 |                 0 |                0 |      22441 |    28785 |      510 |          27198 |        30325 |          511 |          -4757 |        -1540 |           -1 |           1 |             1 |            4 |               4 |                 7 |                1 |          9 |           0 |           -2 |           2 |           1 |           2 | True       |            2 |                     6 | ['Yuumi', 'Xin Zhao', 'Jhin', 'Syndra', 'Tryndamere']  | ['Renekton', 'Lee Sin', 'Leona', 'Jayce', 'Akali']           |
| LCK      | DRX          | oe:team:101f8589e58c724c1dcd5a9c1555277 |         2070 | False    |        21 |           7 |           15 |             0 |             0 |             0 |            0 | False        | 0.2029 | 0.6377 | False         |         0 |             4 |                 0 |                     4 |           0 |           0 |        0 |        0 |           0 |          0 |                        0 |        0 |            0 | True          |         1 |             1 | False        |        1 |            1 | True         |        3 |            9 | False           | False                |              3 |                  1 |            0 |                1 |               66774 | 1935.48 |                2318.49 |                    2678.09 |           117 | 3.3913 |            59 | 1.7101 |                   60 |           262 | 7.5942 |       60674 |        38182 |      1106.72 |       60660 |  0.00914149 | -1.31 |        nan |           994 |            186 |                     nan |                       nan | 34.2029 |      15495 |    17872 |      318 |          16695 |        19149 |          333 |          -1200 |        -1277 |          -15 |           2 |             5 |            4 |               4 |                 5 |                2 |      23612 |    29371 |      528 |          24657 |        30106 |          546 |          -1045 |         -735 |          -18 |           2 |             5 |            4 |               4 |                 5 |                2 |         -8 |           0 |            0 |           0 |          -6 |          -1 | False      |            0 |                     2 | ['Aphelios', 'Jarvan IV', 'Thresh', 'Ryze', 'Graves']  | ['Diana', 'Caitlyn', 'Yuumi', 'Samira', 'Syndra']            |
| LCK      | Liiv SANDBOX | oe:team:c75f1f337fc5867914749d438a4871d |         2070 | True     |        31 |          15 |            7 |             2 |             0 |             0 |            0 | True         | 0.4348 | 0.6377 | True          |         4 |             0 |                 4 |                     0 |           0 |           0 |        0 |        2 |           1 |          1 |                        0 |        0 |            0 | False         |         1 |             1 | True         |        1 |            1 | False        |        9 |            3 | True            | True                 |              1 |                  3 |            1 |                0 |               57616 | 1670.03 |                2938.81 |                    2685.86 |           104 | 3.0145 |            60 | 1.7391 |                   45 |           253 | 7.3333 |       67152 |        44660 |      1294.49 |       60108 | -0.00914149 |  1.31 |        nan |          1051 |            227 |                     nan |                       nan | 37.0435 |      16695 |    19149 |      333 |          15495 |        17872 |          318 |           1200 |         1277 |           15 |           4 |             5 |            2 |               2 |                 5 |                4 |      24657 |    30106 |      546 |          23612 |        29371 |          528 |           1045 |          735 |           18 |           4 |             5 |            2 |               2 |                 5 |                4 |          8 |           0 |            0 |           0 |           6 |           1 | True       |            2 |                     6 | ['Leona', 'Xin Zhao', 'Akali', 'Ezreal', 'Tryndamere'] | ['Renekton', 'Lee Sin', 'Twisted Fate', 'Viktor', 'LeBlanc'] |
| LCK      | T1           | oe:team:ce499dea30cfce118f4fe85da0227e8 |         2233 | True     |        26 |          12 |            7 |             1 |             1 |             0 |            0 | True         | 0.3224 | 0.5105 | True          |         5 |             0 |                 4 |                     0 |           1 |           0 |        0 |        2 |           0 |          1 |                        0 |        1 |            0 | True          |         2 |             0 | True         |        1 |            0 | False        |        7 |            2 | True            | True                 |              2 |                  1 |            1 |                0 |               74327 | 1997.14 |                2789.72 |                    2453.12 |           122 | 3.2781 |            83 | 2.2302 |                   50 |           332 | 8.9207 |       66455 |        42304 |      1136.69 |       59950 |  0.0386823  |  1    |        nan |           970 |            236 |                     nan |                       nan | 32.4048 |      15662 |    18130 |      324 |          15510 |        19078 |          337 |            152 |         -948 |          -13 |           1 |             1 |            1 |               1 |                 1 |                1 |      24357 |    29835 |      518 |          23048 |        30005 |          533 |           1309 |         -170 |          -15 |           3 |             2 |            1 |               1 |                 1 |                3 |          5 |           1 |            2 |           1 |           5 |           1 | False      |            2 |                     8 | ['Karma', 'Ezreal', 'Jarvan IV', 'Gragas', 'Zoe']      | ['Lee Sin', 'Ryze', 'Viktor', 'LeBlanc', 'Graves']           |

### Univariate Analysis

This figure plots the distribution of kpm across all games played in regional leagues. We can observe that the the distribution is roughly skewed right with a mean of about 0.37.
<iframe
  src="assets/kpm_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This figure plots the distribution of multikills across all games played in regional leauges. We can observe that the number of multikills seen in a game trends downward with a mean around 2.
<iframe
  src="assets/multikills_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This figure plots the distributions of the various kinds of multikills across all games played in regional leagues. We can observe that it is indeed less common to see larger multikills, and that there were never more than one pentakill in the games observed in 2022 but 408 games included pentakills.
<iframe
  src="assets/all_multikills_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

This figure depicts the distributions of the `'kpm'` of different leagues. We can see that the VCS league seems to have the highest median `'kpm'`whereas the lowest appears to be the LCK.
<iframe
  src="assets/kpm_by_league.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This figure depicts the distributions of the `'multikills'` of different leagues. We can see that the CBLOL and VCS leagues have the highest median observed `'multikills'` of 2 whereas the rest of the leagues of a median observed `'multikills'` of 1.
<iframe
  src="assets/multikills_by_league.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This figure depicts the distributions of the `'objectives'` captured by different leagues. We can observe that the distributions are very similar across all leagues, but the LCK and the PCS have the highest maxes of 10 while the LEC has the lowest max of 6.
<iframe
  src="assets/multikills_by_league.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

