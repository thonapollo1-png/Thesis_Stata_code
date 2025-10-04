# Masters Thesis Analysis: Does Corruption Deters Foreign Direct Investment (FDI) in Low-Income Countries (Africa)? Panel Data Analysis in Stata

This repository contains Stata code for conducting panel data analysis on the determinants of Foreign Direct Investment (FDI).  
The analysis includes data preparation, exploratory data analysis, data visualization, and panel regression modeling (pooled OLS, fixed effects, random effects, and model extensions).

---

## Preliminaries

```stata
/*******************************************************************************
* Thesis Analysis: Determinants of Foreign Direct Investment (FDI)
*
* This script performs panel data analysis in Stata to investigate the factors
* affecting FDI. It includes data preparation, exploratory data analysis,
* data visualization, and panel regression modeling.
*******************************************************************************/

* --- Preliminaries ---
clear all       // Clears memory
set more off    // Prevents Stata from pausing after each screen of output
version 17      // Sets Stata version for compatibility

* --- Set your working directory ---
* Replace the path below with the path to your project folder
cd "C:\path\to\your\folder"
pwd             // Shows the current working directory

## Setup and Data Loading
* --- Install necessary packages (only needs to be run once) ---
* ssc install estout, replace
* ssc install heatplot, replace

* --- Import the dataset ---
import delimited "panel_data.csv", clear

* Stata is case-sensitive; for consistency, let's make variable names lowercase
rename *, lower

## Data Preparation and Exploration
* --- Declare the dataset as a panel ---
encode country, gen(country_id) // Create a numeric ID for country strings
xtset country_id year

* --- Descriptive Statistics ---
sum fdi gdp inflation exchange_rate corruption_con pol_sta trade

tabstat fdi gdp inflation exchange_rate corruption_con pol_sta trade, ///
    stats(count mean sd min p25 p50 p75 max) c(stat)

* --- Correlation Matrix ---
correlate fdi gdp inflation exchange_rate corruption_con pol_sta trade

* --- Heatmap of the correlation matrix ---
heatplot fdi gdp inflation exchange_rate corruption_con pol_sta trade, ///
    title("Correlation Matrix Heatmap")

## Data Visualization
* --- Boxplot of FDI by country (in billions) ---
graph box fdi, over(country, sort(1)) medtype(line) ///
    title("FDI by Country") ytitle("FDI (Billions USD)") ///
    ylab( , labsize(small) format(%9.1fc))

* --- Line plot of FDI trends over time ---
xtline fdi, overlay title("FDI Trends by Country") ///
    ytitle("FDI Inflows") xtitle("Year")

* --- Scatter plot of FDI vs. Corruption Control ---
twoway (scatter fdi corruption_con), ///
    title("FDI vs. Corruption Control") ///
    ytitle("FDI") xtitle("Corruption Control Index")

## Panel Regression Models
* --- Create transformed variables ---
gen log_fdi_plus1 = log(fdi + 1)
label var log_fdi_plus1 "Log(FDI + 1)"
gen log_gdp = log(gdp)
label var log_gdp "Log(GDP)"

* --- Model 1: Pooled OLS ---
regress log_fdi_plus1 log_gdp inflation exchange_rate corruption_con pol_sta trade i.year
eststo pooled_ols

* --- Model 2: Fixed Effects (FE) ---
xtreg log_fdi_plus1 log_gdp inflation exchange_rate corruption_con pol_sta trade i.year, fe
eststo fixed_effects

* --- Model 3: Random Effects (RE) ---
xtreg log_fdi_plus1 log_gdp inflation exchange_rate corruption_con pol_sta trade i.year, re
eststo random_effects

## Specification Tests
di "--- SPECIFICATION TESTS ---"

* --- Test 1: Chow Test (FE vs Pooled OLS) ---
* Check the F-test at the bottom of the FE model output.

* --- Test 2: Breusch-Pagan LM Test (RE vs Pooled OLS) ---
xtreg log_fdi_plus1 log_gdp inflation exchange_rate corruption_con pol_sta trade i.year, re
xttest0

* --- Test 3: Hausman Test (FE vs RE) ---
quietly xtreg log_fdi_plus1 log_gdp inflation exchange_rate corruption_con pol_sta trade i.year, fe
estimates store fe_hausman
quietly xtreg log_fdi_plus1 log_gdp inflation exchange_rate corruption_con pol_sta trade i.year, re
estimates store re_hausman
hausman fe_hausman re_hausman, sigmamore

## Final Models and Publication Table
* --- RE with Interaction Term ---
xtreg log_fdi_plus1 log_gdp inflation exchange_rate c.corruption_con##c.pol_sta trade i.year, re
eststo re_interaction
* --- RE with Quadratic Term ---
xtreg log_fdi_plus1 log_gdp inflation exchange_rate corruption_con c.corruption_con#c.corruption_con pol_sta trade i.year, re
eststo re_quadratic

* --- Publication-quality regression table ---
esttab pooled_ols fixed_effects random_effects re_interaction re_quadratic, ///
    b(3) t(2)                                   /* Coefficients 3 decimals, t-stats 2 decimals */ ///
    star(* 0.10 ** 0.05 *** 0.01)               /* Significance levels */ ///
    title("Panel Regression Results of FDI Determinants") ///
    depvars("Dependent Variable: Log(FDI + 1)") ///
    modelwidth(15) ///
    mtitles("Pooled OLS" "Fixed Effects" "Random Effects" "RE Interaction" "RE Quadratic") ///
    addnotes("t-statistics in parentheses")


