# DATA 607 Final Project- Housing Affordability Dynamics in New York State: A Multi-Source Analysis Using HUD, Census ACS, and FRED Economic Indicators

## INTRODUCTION

This project analyzes the housing affordability crisis in New York State by
combining data from the U.S. Census Bureau, the Federal Reserve, and the
Department of Housing and Urban Development. Using county-level data from 2009
to 2024, the analysis tracks how income, rent, home values, and macroeconomic
conditions have changed over time — and builds predictive models to identify what
drives housing burden and how sensitive those models are to recent interest rate
shocks.

## ACS 5-Year Data Pull

Median household income, home values, and gross rent are pulled for all 62 New
York counties from the Census ACS 5-Year Estimates API, covering 2009 to 2024.
County names are cleaned and all numeric fields are standardized before the data
is stacked into a single panel and saved as a CSV. The preview table immediately
highlights geographic disparity — Bronx County in 2009 had a median income of
only $33,794, yet a median home value of $369,600, nearly double Albany County's
$192,500 despite Albany having significantly higher income.

## ACS Exploratory Data Analysis

Two charts summarize statewide housing trends across the study period. The left
panel plots median income, home values, and rent as separate trend lines. The most
striking feature is the steep acceleration in home values after 2019, driven by
pandemic-era demand and constrained supply, while income growth remained gradual
and nearly flat in real terms. Rent increased steadily but at a slower pace than
home values, suggesting that the ownership market absorbed the larger share of the
post-2020 shock.

The right panel converts these trends into a price-to-income (PTI) ratio — home
value divided by median income — which is a standard affordability benchmark. A
PTI above 3.0× is conventionally considered unaffordable. The statewide mean PTI
has remained above 3.0× throughout the entire 2009–2024 period, meaning New York
counties have been in persistent affordability stress for over a decade. The
interquartile ribbon widens over time, indicating that the gap between lower-burden
rural counties and higher-burden urban counties has grown, not narrowed. The sharpest
increase occurs after 2020, where the mean PTI climbs toward 4.5× — a level that
signals severe structural unaffordability rather than a temporary cyclical dip.


## FRED Data Pull


Three macroeconomic series are retrieved from the FRED API: the 30-year fixed
mortgage rate, the effective Federal Funds Rate, and the CPI. A retry-enabled
function handles intermittent server errors, attempting each request up to five
times before failing. All three series are aligned to monthly frequency, joined
together, and exported for use in the master dataset.


## FRED ACS-Aligned Yearly Aggregate


To align with the ACS panel, the monthly FRED data is aggregated into annual
averages for 2009–2024. This produces a compact 16-row table with one row per
year, capturing the average mortgage rate, Fed Funds Rate, and CPI level — ready
to join with the county-level data. The table confirms the historically low-rate
environment of 2010–2021 (mortgage rate averaging around 3.5–5%) before the sharp
rise to 6.28% in 2022–2024.


## FRED Exploratory Data Analysis


Two charts provide macroeconomic context for the housing analysis. The top panel
plots the 30-year mortgage rate alongside the Federal Funds Rate from 1971 to the
present. The two lines track closely, confirming that retail mortgage costs move
in step with monetary policy. The most relevant feature for this project is the
sharp rate reversal beginning in early 2022 — the Fed Funds Rate climbed from near
zero to over 5%, pushing the 30-year mortgage rate above 7% for the first time
since 2001. This rate shock directly reduced purchasing power for homebuyers and
added upward pressure on rents as demand shifted toward the rental market.

The bottom panel shows annual CPI inflation as a bar chart. Inflation ran near or
below the 2% Fed target for most of 2010–2020, then surged to nearly 9% in 2022
— the highest level since 1981. This inflation spike eroded real household income
at the same time that mortgage rates were rising, creating a dual affordability
squeeze. The 2% target line shows how far the post-pandemic inflation deviated from
the prior decade's baseline, and the rapid return toward target by 2023–2024
provides important context for interpreting the rate environment during the machine
unlearning experiment in RQ2.


## HUD Fair Market Rent Data Pull


HUD Fair Market Rent data for all five bedroom sizes — studio through four-bedroom
— are downloaded for each New York county from fiscal years 2009 to 2024. Files
are read from GitHub and stacked into a single panel using `map_dfr()`. FMRs
represent the 40th percentile of gross rents paid by recent movers and serve as
the federal benchmark for housing voucher payment standards under the Section 8
program. The preview confirms the expected structure — in 2009, Bronx County FMRs
already ranged from $1,091 (studio) to $1,817 (4-BR), substantially above most
upstate counties.


## HUD FMR Exploratory Data Analysis


A dot plot of 2024 HUD FMRs displays the full spread of rental costs across all
62 counties by unit size, sorted from lowest to highest two-bedroom rent. The chart
is split into lower-rent and higher-rent panels for readability.

The findings reveal two important patterns. First, geographic variation is
enormous: 2-BR FMRs in 2024 range from under $850 in Allegany, Cattaraugus, and
Wyoming counties to over $2,500 in New York City and Rockland County — nearly a
3× difference from the cheapest to the most expensive market in the same state.
Second, the width of each county's connecting line — which spans all five bedroom
sizes — is much larger in high-cost markets. For example, the spread between a
studio and a 4-BR unit in New York City exceeds $1,000/month, while the same
spread in a rural county is under $300. This bedroom-size premium effect reflects
tighter supply of family-sized units in dense urban areas, where larger apartments
command a disproportionate market premium beyond the base rent level.


## Master Dataset Build


The three data sources are merged into a single county-year panel by joining ACS,
HUD, and FRED data onto a complete 62-county × 16-year grid. Four derived features
are calculated: housing burden (annualized rent divided by median income), HUD
burden (2-BR FMR divided by median income), real income index (CPI-adjusted income),
and macro housing pressure (burden multiplied by mortgage rate). The final dataset
has 992 rows and 21 variables, providing a rich foundation for both exploratory
analysis and predictive modeling.


## Shiny App

Link: https://khandker.shinyapps.io/shinyapp/

An interactive Shiny application maps any variable in the master dataset across all
62 New York counties for any year from 2009 to 2024. Users select a metric from a
dropdown and step through years with an animated slider. Hovering over a county
displays a tooltip with income, rent, population, and the selected metric value.
The app makes it easy to observe, for example, how the housing burden ratio
darkens across downstate counties after 2020, or how FMR increases spread
geographically over time — patterns that are difficult to detect from tables alone.


## RQ1: Model Setup and Training


To identify the key drivers of housing burden, four models are trained and compared:
Linear Regression, Elastic Net, Random Forest, and XGBoost. The target variable is
the rent-to-income housing burden ratio. Predictors include median home value,
population, all five FMR bedroom sizes, macroeconomic indicators, and county fixed
effects encoded as dummy variables. An 80/20 stratified train-test split and five-fold
cross-validation are used to tune hyperparameters and evaluate performance.


## RQ1: Model Comparison and Performance Plot


The cross-validated results table and bar chart compare all four models on RMSE,
MAE, and R². The Random Forest achieves the best overall performance with RMSE =
0.00705 and R² = 0.964, followed closely by Elastic Net and Linear Regression
(both R² ≈ 0.960). XGBoost performs slightly worse (R² = 0.947) despite its
greater complexity, suggesting the data structure does not benefit from deep
boosting iterations. Importantly, all four models cluster within a narrow RMSE
range of 0.007–0.008, which means the strength of the predictive signal is
consistent and not dependent on model choice. This is a strong indicator that the
features — particularly population, home value, and FMR — carry genuine, stable
information about affordability burden rather than noise.


## RQ1: Test Set Evaluation and Feature Importance


The winning Random Forest model is evaluated on the held-out test set, confirming
RMSE = 0.00638 and R² = 0.959 — both consistent with cross-validation, indicating
no overfitting. The variable importance plot reveals a clear hierarchy of predictors.
Population is the single strongest driver, reflecting the well-documented relationship
between housing demand density and rent-to-income stress. County fixed effects rank
second, capturing persistent local market character that numeric variables alone
cannot explain. Median home value ranks third — counties where purchasing is already
expensive tend to have proportionally higher rents as well. The FMR bedroom-size
series and CPI follow, while mortgage rate and the Fed Funds Rate rank near the
bottom. This pattern is significant: it suggests that affordability burden is
primarily structural and geographic, not cyclical — a finding that has direct
implications for what kinds of policy interventions are most likely to be effective.


## RQ1: Correlation Table and Conclusion


The correlation matrix provides a quantitative summary of variable relationships
and validates the feature importance findings. Housing burden correlates most
strongly with population (r = 0.73) and median home value (r = 0.60), confirming
these as the primary structural drivers. In contrast, mortgage rate (r = −0.04)
and the Fed Funds Rate (r = −0.04) show near-zero correlation with housing burden,
reinforcing that macroeconomic rate variables play a limited direct role at the
county level. The FMR columns are highly intercorrelated with each other
(r = 0.96–1.00), indicating multicollinearity across bedroom sizes — the Random
Forest handles this naturally, but it would require regularization in a linear
setting. The year variable correlates strongly with CPI (r = 0.94) and the Fed
Funds Rate (r = 0.73), confirming that both variables largely reflect long-run
time trends rather than independent signals.


## RQ2: Machine Unlearning Experiment


Research Question 2 tests how sensitive Fair Market Rent predictions are to the
removal of high-rate years (2022–2024), when the 30-year mortgage rate exceeded 5%.
Both Linear Regression and Random Forest are retrained at five forgetting levels
(case weight 1.0 down to 0.0 for high-rate-year observations).

**R² forgetting curve:** Both models maintain high R² across all
forgetting steps, declining only marginally from ~0.947 to ~0.940 at full
unlearning. This stability confirms that FMR is structurally driven — a model
with no knowledge of the 2022–2024 rate environment can still explain 94% of the
cross-county variation in rents.

**Predicted FMR trajectory:** Prior to 2022, the full and unlearned
models predict nearly identical statewide rent levels. After 2022, a visible gap
opens, with the full model predicting approximately $70/month higher rents on
average. This gap — the shaded region in the chart — represents the direct
imprint of the rate-hike era on model predictions.

**County-level sensitivity:** Geographic variation in the shift is
striking. Downstate counties show the largest negative shifts: Rockland (−$533),
New York County (−$519), the Bronx (−$518), Queens (−$474), and Kings (−$469)
— meaning rate-hike-era data caused the model to learn significantly higher rents
in these markets. Conversely, upstate counties like Erie (+$313), Onondaga (+$271),
and Monroe (+$253) show positive shifts, suggesting rate hikes suppressed rents in
those markets relative to their structural trend. Rural counties cluster near zero,
indicating minimal sensitivity to the rate environment.

**Shift distribution by era:** The boxplot confirms that prediction
shifts are concentrated in the high-rate era observations, with a wider and
off-center distribution compared to the normal-rate era. The spread across
county-year pairs in the high-rate era reflects the geographic heterogeneity
documented in Figure 3 — some markets were strongly shaped by the rate environment
while others were largely unaffected. Together, the four figures show that machine
unlearning is a useful diagnostic tool for identifying which housing markets are
most exposed to rate-cycle distortions in model training data.


## CONCLUSION


This analysis finds that housing affordability in New York State is primarily a
structural and geographic problem. Local factors — population density, home price
levels, and regional rent benchmarks — explain approximately 96% of the variation
in housing burden across counties, and these relationships have remained stable
even through the interest rate shock of 2022–2024. Macroeconomic variables such
as mortgage rates and the Fed Funds Rate, while economically significant at the
national level, contribute minimally to predicting county-level burden once local
structural conditions are accounted for.

The machine unlearning experiment adds an important nuance: while the high-rate era
did influence what models learned about rental markets — producing a mean statewide
prediction shift of −$70/month — that influence was heavily concentrated in
high-cost downstate counties. Rural and mid-size upstate markets were largely
unaffected, reflecting their lower sensitivity to rate-driven demand swings. These
findings suggest that meaningful affordability improvements in New York will require
addressing local supply constraints and income gaps directly, rather than relying
on monetary policy or cyclical rate relief to restore affordability.
