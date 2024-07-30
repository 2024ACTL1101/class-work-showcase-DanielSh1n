
# CAPM Analysis

## Introduction

In this assignment, you will explore the foundational concepts of the Capital Asset Pricing Model (CAPM) using historical data for AMD and the S&P 500 index. This exercise is designed to provide a hands-on approach to understanding how these models are used in financial analysis to assess investment risks and returns.

## Background

The CAPM provides a framework to understand the relationship between systematic risk and expected return, especially for stocks. This model is critical for determining the theoretically appropriate required rate of return of an asset, assisting in decisions about adding assets to a diversified portfolio.

## Objectives

1. **Load and Prepare Data:** Import and prepare historical price data for AMD and the S&P 500 to ensure it is ready for detailed analysis.
2. **CAPM Implementation:** Focus will be placed on applying the CAPM to examine the relationship between AMD's stock performance and the overall market as represented by the S&P 500.
3. **Beta Estimation and Analysis:** Calculate the beta of AMD, which measures its volatility relative to the market, providing insights into its systematic risk.
4. **Results Interpretation:** Analyze the outcomes of the CAPM application, discussing the implications of AMD's beta in terms of investment risk and potential returns.

## Instructions

### Step 1: Data Loading

- We are using the `quantmod` package to directly load financial data from Yahoo Finance without the need to manually download and read from a CSV file.
- `quantmod` stands for "Quantitative Financial Modelling Framework". It was developed to aid the quantitative trader in the development, testing, and deployment of statistically based trading models.
- Make sure to install the `quantmod` package by running `install.packages("quantmod")` in the R console before proceeding.

```r
# Set start and end dates
start_date <- as.Date("2019-05-20")
end_date <- as.Date("2024-05-20")

# Load data for AMD, S&P 500, and the 1-month T-Bill (DTB4WK)
amd_data <- getSymbols("AMD", src = "yahoo", from = start_date, to = end_date, auto.assign = FALSE)
gspc_data <- getSymbols("^GSPC", src = "yahoo", from = start_date, to = end_date, auto.assign = FALSE)
rf_data <- getSymbols("DTB4WK", src = "FRED", from = start_date, to = end_date, auto.assign = FALSE)

# Convert Adjusted Closing Prices and DTB4WK to data frames
amd_df <- data.frame(Date = index(amd_data), AMD = as.numeric(Cl(amd_data)))
gspc_df <- data.frame(Date = index(gspc_data), GSPC = as.numeric(Cl(gspc_data)))
rf_df <- data.frame(Date = index(rf_data), RF = as.numeric(rf_data[,1]))  # Accessing the first column of rf_data

# Merge the AMD, GSPC, and RF data frames on the Date column
df <- merge(amd_df, gspc_df, by = "Date")
df <- merge(df, rf_df, by = "Date")
```

#### Data Processing 
```r
colSums(is.na(df))
# Fill N/A RF data
df <- df %>%
  fill(RF, .direction = "down") 
```

### Step 2: CAPM Analysis

The Capital Asset Pricing Model (CAPM) is a financial model that describes the relationship between systematic risk and expected return for assets, particularly stocks. It is widely used to determine a theoretically appropriate required rate of return of an asset, to make decisions about adding assets to a well-diversified portfolio.

#### The CAPM Formula
The formula for CAPM is given by:

$$
E(R_i) = R_f + \beta_i (E(R_m) - R_f)
$$

Where:

- $E(R_i)$ is the expected return on the capital asset,
- $R_f$ is the risk-free rate,
- $\beta_i$ is the beta of the security, which represents the systematic risk of the security,
- $E(R_m)$ is the expected return of the market.



#### CAPM Model Daily Estimation

- **Calculate Returns**: First, we calculate the daily returns for AMD and the S&P 500 from their adjusted closing prices. This should be done by dividing the difference in prices between two consecutive days by the price at the beginning of the period.
  
$$
\text{Daily Return} = \frac{\text{Today's Price} - \text{Previous Trading Day's Price}}{\text{Previous Trading Day's Price}}
$$

```r
# Daily Return for AMD
df <- df %>%
mutate(AMD_returns = (lead(AMD) - AMD) / AMD)
# Daily Return for SP500
df <- df %>%
mutate(GSPC_returns = (lead(GSPC) - GSPC) / GSPC)
```

- **Calculate Risk-Free Rate**: Calculate the daily risk-free rate by conversion of annual risk-free Rate. This conversion accounts for the compounding effect over the days of the year and is calculated using the formula:
  
$$
\text{Daily Risk-Free Rate} = \left(1 + \frac{\text{Annual Rate}}{100}\right)^{\frac{1}{360}} - 1
$$

```r
# Daily Risk-Free Rate
df <- df %>%
mutate(daily_RF = (1 + RF / 100)ˆ(1/360) - 1)
```


- **Calculate Excess Returns**: Compute the excess returns for AMD and the S&P 500 by subtracting the daily risk-free rate from their respective returns.

```r
# Excess Returns
df <- df %>%
mutate(AMD_excess_returns = AMD_returns - daily_RF,
GSPC_excess_returns = GSPC_returns - daily_RF)
```


- **Perform Regression Analysis**: Using linear regression, we estimate the beta (\(\beta\)) of AMD relative to the S&P 500. Here, the dependent variable is the excess return of AMD, and the independent variable is the excess return of the S&P 500. Beta measures the sensitivity of the stock's returns to fluctuations in the market.

```r
library(tidyverse)
library(broom)
model <- lm(AMD_excess_returns ~ GSPC_excess_returns, data = df)
summary(model)
```


#### Interpretation

What is your \(\beta\)? Is AMD more volatile or less volatile than the market?

Our β is 1.5700073. A β of 1.5700073 means that the dependent variable (AMD’s stock price) is approximately
57% more volatile than the market represented by the independent variable (S&P500). A β that is
greater than one, such as the one for this task, means that the dependent variable is more likely to move
(more sensitive) in respect to the independent variable. So in this example, if the market goes up by 1%,
then AMD’s stock will go up by around 1.57%, and if the market goes down by 1%, then AMD share will
also go down by 1.57%. Higher volatility suggests that AMD’s stock is more sensitive to market movements.
What this means to investors is, that if the market experiences a rise or fall, AMD’s stock is likely to rise or
fall more steeply in comparison. A stock that is more volatile are considered greater risks, but offer greater
gains, so investors who are more bullish can utilize such stocks in their portfolio, if investors want to be more
safe, they can opt for stock options that have lower β values, for less volatile investment options.


#### Plotting the CAPM Line
Plot the scatter plot of AMD vs. S&P 500 excess returns and add the CAPM regression line.

```r
# Load the necessary library for plotting
library(ggplot2)
# Create a scatter plot with a regression line
ggplot(data = df, aes(x = GSPC_excess_returns, y = AMD_excess_returns)) +
geom_point(color = 'blue', alpha = 0.6) +
geom_smooth(method = "lm", color = 'red', se = TRUE) +
labs(title = "Scatter Plot of AMD vs. S&P 500 Excess Returns",
x = "S&P 500 Excess Returns",
y = "AMD Excess Returns",
caption = "Data source: Calculated Excess Returns") +
theme_minimal()
```

### Step 3: Predictions Interval
Suppose the current risk-free rate is 5.0%, and the annual expected return for the S&P 500 is 13.3%. Determine a 90% prediction interval for AMD's annual expected return.



So given that the risk free rate is 5%, with the expected market return to be 13.3% and the beta value from
previous question, the expected annual return for AMD is 0.1803106. The prediction interval is [-0.4904554
0.8510766].

```r
# Calculating Expected Annual Return of AMD:
risk_free_rate <- 0.05 # Current risk-free rate is 5%
expected_market_return <- 0.133 # Expected annual return for the S&P 500 is 13.3%
beta_AMD <- 1.5700073 # Placeholder for beta of AMD, replace if you have a specific value
expected_annual_ERI <- risk_free_rate + beta_AMD * (expected_market_return - risk_free_rate)
print(expected_annual_ERI)
# Prediction Interval
# Prepping for Calculation
model_sum<-summary(model)
mean<-mean(df$GSPC_excess_returns[!is.na(df$GSPC_excess_returns)])
n<-length(df$GSPC_excess_returns[!is.na(df$GSPC_excess_returns)])
daily_standard_error<-sqrt(sum(residuals(model)ˆ2/(n-1-1)))
annual_standard_error<-sqrt(252)*daily_standard_error
alpha <- 0.10
n <- nrow(df[-1,])
t_value <- qt(1-alpha/2, df = n-2)
print(t_value)
lower_bound <- expected_annual_ERI - t_value*annual_standard_error
upper_bound <- expected_annual_ERI + t_value*annual_standard_error
pi <- c(lower_bound, upper_bound)
print(pi)
```
