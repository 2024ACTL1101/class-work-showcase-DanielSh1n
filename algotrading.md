
## Algorithmic Trading Strategy

## Introduction

In this assignment, you will develop an algorithmic trading strategy by incorporating financial metrics to evaluate its profitability. This exercise simulates a real-world scenario where you, as part of a financial technology team, need to present an improved version of a trading algorithm that not only executes trades but also calculates and reports on the financial performance of those trades.

## Background

Following a successful presentation to the Board of Directors, you have been tasked by the Trading Strategies Team to modify your trading algorithm. This modification should include tracking the costs and proceeds of trades to facilitate a deeper evaluation of the algorithm’s profitability, including calculating the Return on Investment (ROI).

After meeting with the Trading Strategies Team, you were asked to include costs, proceeds, and return on investments metrics to assess the profitability of your trading algorithm.

## Objectives

1. **Load and Prepare Data:** Open and run the starter code to create a DataFrame with stock closing data.

2. **Implement Trading Algorithm:** Create a simple trading algorithm based on daily price changes.

3. **Customize Trading Period:** Choose your entry and exit dates.

4. **Report Financial Performance:** Analyze and report the total profit or loss (P/L) and the ROI of the trading strategy.

5. **Implement a Trading Strategy:** Implement a trading strategy and analyze the total updated P/L and ROI. 

6. **Discussion:** Summarise your finding.


## Instructions

### Step 1: Data Loading

Start by running the provided code cells in the "Data Loading" section to generate a DataFrame containing AMD stock closing data. This will serve as the basis for your trading decisions. First, create a data frame named `amd_df` with the given closing prices and corresponding dates. 

```r
# Load data from CSV file
amd_df <- read.csv("AMD.csv")
# Convert the date column to Date type and Adjusted Close as numeric
amd_df$date <- as.Date(amd_df$Date)
amd_df$close <- as.numeric(amd_df$Adj.Close)
amd_df <- amd_df[, c("date", "close")]
```

#### Plotting the Data
Plot the closing prices over time to visualize the price movement.
```r
plot(amd_df$date, amd_df$close,'l')
```

### Step 2: Trading Algorithm
Implement the trading algorithm as per the instructions. You should initialize necessary variables, and loop through the dataframe to execute trades based on the set conditions.

- Initialize Columns: Start by ensuring dataframe has columns 'trade_type', 'costs_proceeds' and 'accumulated_shares'.
- Change the algorithm by modifying the loop to include the cost and proceeds metrics for buys of 100 shares. Make sure that the algorithm checks the following conditions and executes the strategy for each one:
  - If the previous price = 0, set 'trade_type' to 'buy', and set the 'costs_proceeds' column to the current share price multiplied by a `share_size` value of 100. Make sure to take the negative value of the expression so that the cost reflects money leaving an account. Finally, make sure to add the bought shares to an `accumulated_shares` variable.
  - Otherwise, if the price of the current day is less than that of the previous day, set the 'trade_type' to 'buy'. Set the 'costs_proceeds' to the current share price multiplied by a `share_size` value of 100.
  - You will not modify the algorithm for instances where the current day’s price is greater than the previous day’s price or when it is equal to the previous day’s price.
  - If this is the last day of trading, set the 'trade_type' to 'sell'. In this case, also set the 'costs_proceeds' column to the total number in the `accumulated_shares` variable multiplied by the price of the last day.



```r
# Initialize columns for trade type, cost/proceeds, and accumulated shares in amd_df
amd_df$trade_type <- NA
amd_df$costs_proceeds <- NA  # Corrected column name
amd_df$accumulated_shares <- 0  # Initialize if needed for tracking

# Initialize variables for trading logic
previous_price <- 0
share_size <- 100
accumulated_shares <- 0

for (i in 1:nrow(amd_df)) {
if (i == 1 || amd_df$close[i] < previous_price) {
# Buy
amd_df$trade_type[i] <- 'buy'
amd_df$costs_proceeds[i] <- -amd_df$close[i] * share_size
accumulated_shares <- accumulated_shares + share_size
} else if (i == nrow(amd_df)) {
# Last day, sell
amd_df$trade_length[i] <- 'sell'
amd_df$costs_proceeds[i] <- amd_df$close[i] * accumulated_shares
accumulated_shares <- 0
}
# Update previous price
previous_price <- amd_df$close[i]
# Update accumulated shares in DataFrame
amd_df$accumulated_shares[i] <- accumulated_shares
}
# Print the updated DataFrame
print(amd_df)
```


### Step 3: Customize Trading Period
- Define a trading period you wanted in the past five years 
```r
#Define a trading period you wanted in the past five years.
#{r period}
# Make sure that the 'date' column is in Date format
amd_df$date <- as.Date(amd_df$date)
# Filter rows for the years January 2021 to June 2021
trading_period_df <- amd_df[amd_df$date >= as.Date("2021-01-01") & amd_df$date <= as.Date("2021-06-30"), # Print filtered DataFrame
print(trading_period_df)
```


### Step 4: Run Your Algorithm and Analyze Results
After running your algorithm, check if the trades were executed as expected. Calculate the total profit or loss and ROI from the trades.

- Total Profit/Loss Calculation: Calculate the total profit or loss from your trades. This should be the sum of all entries in the 'costs_proceeds' column of your dataframe. This column records the financial impact of each trade, reflecting money spent on buys as negative values and money gained from sells as positive values.
- Invested Capital: Calculate the total capital invested. This is equal to the sum of the 'costs_proceeds' values for all 'buy' transactions. Since these entries are negative (representing money spent), you should take the negative sum of these values to reflect the total amount invested.
- ROI Formula: $$\text{ROI} = \left( \frac{\text{Total Profit or Loss}}{\text{Total Capital Invested}} \right) \times 100$$

```r
# Initialize Columns
trading_period_df$trade_type <- NA
trading_period_df$costs_proceeds <- NA
trading_period_df$accumulated_shares <- 0
# Initialize variables
previous_price <- 0
share_size <- 100
accumulated_shares <- 0
for (i in 1:nrow(trading_period_df)) {
if (i == 1 || trading_period_df$close[i] < previous_price) {
# Buy
trading_period_df$trade_type[i] <- 'buy'
trading_period_df$costs_proceeds[i] <- -trading_period_df$close[i] * share_size
accumulated_shares <- accumulated_shares + share_size
} else if (i == nrow(trading_period_df)) {
# Last day, sell
trading_period_df$trade_length[i] <- 'sell'
trading_period_df$costs_proceeds[i] <- trading_period_df$close[i] * accumulated_shares
accumulated_shares <- 0
}
# Update previous price
previous_price <- trading_period_df$close[i]
# Update accumulated shares in DataFrame
trading_period_df$accumulated_shares[i] <- accumulated_shares
}
# Total profits and loss
total_profit_loss <- sum(trading_period_df$costs_proceeds, na.rm = TRUE)
# Calculating Total Capital Invested
total_invested <- -sum(trading_period_df$costs_proceeds[trading_period_df$trade_type == 'buy'], na.rm = # Calculating ROI
ROI <- (total_profit_loss / total_invested) * 100
print(ROI)
```

### Step 5: Profit-Taking Strategy or Stop-Loss Mechanisum (Choose 1)
- Option 1: Implement a profit-taking strategy that you sell half of your holdings if the price has increased by a certain percentage (e.g., 20%) from the average purchase price.
- Option 2: Implement a stop-loss mechanism in the trading strategy that you sell half of your holdings if the stock falls by a certain percentage (e.g., 20%) from the average purchase price. You don't need to buy 100 stocks on the days that the stop-loss mechanism is triggered.


```r
##METHOD 1:
previous_price <- 0
accumulated_shares <- 0
share_size <- 100
avg_purchase_price <-0
total_cost <- 0
profit_margin <- 0.2
for (i in 1:nrow(trading_period_df)) {
if (i == nrow(trading_period_df)) {
trading_period_df$trade_type[i] <- 'sell'
trading_period_df$costs_proceeds[i] <- accumulated_shares * trading_period_df$close[i]
}
else if (previous_price == 0 || trading_period_df$close[i] < previous_price) {
trading_period_df$trade_type[i] <- 'buy'
trading_period_df$costs_proceeds[i] <- -trading_period_df$close[i] * share_size
total_cost <- total_cost - trading_period_df$costs_proceeds[i]
accumulated_shares <- accumulated_shares + share_size
}
else if (i > 1 && avg_purchase_price * (1 + profit_margin) <= trading_period_df$close[i]) {
trading_period_df$trade_type[i] <- 'sell'
shares_to_sell <- accumulated_shares / 2
trading_period_df$costs_proceeds[i] <- trading_period_df$close[i] * shares_to_sell
accumulated_shares <- accumulated_shares - shares_to_sell
total_cost <- total_cost - (shares_to_sell * avg_purchase_price )
}
else {
trading_period_df$trade_type[i] <- 'hold'
}
previous_price <- trading_period_df$close[i]
trading_period_df$accumulated_shares[i] <- accumulated_shares
avg_purchase_price = total_cost / accumulated_shares
}
# Set accumulated shares to 0
trading_period_df$accumulated_shares[nrow(trading_period_df)] <- 0
print(trading_period_df)

##METHOD 2
# Make copy
if (!exists("original_trading_period_df")) {
original_trading_period_df <- trading_period_df
}
trading_period_df <- data.frame(original_trading_period_df)
# Initialize Columns
trading_period_df$returns <- rep(0, nrow(trading_period_df))
trading_period_df$purchase_price <- rep(0, nrow(trading_period_df))
trading_period_df$number_of_shares_bought_at_price <- rep(0, nrow(trading_period_df))
# Initialize Variables
profit_percentage <- 0.2 # Profit target (20% above purchase price)
# Setting purchase prices and calculating returns
for (i in 1:nrow(trading_period_df)) {
if (!is.na(trading_period_df$trade_type[i]) && trading_period_df$trade_type[i] == "buy") {
trading_period_df$purchase_price[i] <- trading_period_df$close[i]
trading_period_df$number_of_shares_bought_at_price[i] <- 100
}
}
# Applying the Profit-Taking Strategy
for (i in 1:nrow(trading_period_df)) {
if (trading_period_df$purchase_price[i] > 0) {
for (j in (i + 1):nrow(trading_period_df)) {
if (trading_period_df$close[j] >= (1 + profit_percentage) * trading_period_df$purchase_price[trading_period_df$returns[i] <- trading_period_df$close[j] * 0.5 * trading_period_df$number_break
}
}
}
}
## Update Values for number of shares
for (i in 1:nrow(trading_period_df))
if (trading_period_df$returns[i] > 0 ) {
trading_period_df$number_of_shares_bought_at_price[i] <- trading_period_df$number_of_shares_bought_at_}
# Compute shares sold during profit-taking events
trading_period_df$shares_sold <- 0
for (i in 1:nrow(trading_period_df)) {
if (trading_period_df$returns[i] > 0) {
trading_period_df$shares_sold[i] <- 50
}
}
# Sum total shares sold
total_shares_sold <- sum(trading_period_df$shares_sold)
# Adjust the second last value of accumulated shares by subtracting total shares sold
if (nrow(trading_period_df) > 1) {
trading_period_df$accumulated_shares[nrow(trading_period_df) - 1] <- trading_period_df$accumulated_shares[}
## Total Profits for trading period
# Sell all shares
if (nrow(trading_period_df) > 1) {
# Get the number of remaining shares
remaining_shares <- trading_period_df$accumulated_shares[nrow(trading_period_df) - 1]
# Get the closing price
final_day_closing_price <- trading_period_df$close[nrow(trading_period_df)]
# Calculate the final value for the costs_proceeds column
final_costs_proceeds <- remaining_shares * final_day_closing_price
# Update the final row of the costs_proceeds column
trading_period_df$costs_proceeds[nrow(trading_period_df)] <- final_costs_proceeds
}
# Calculate total returns
total_returns <- sum(trading_period_df$returns, na.rm = TRUE)
# Calculate total costs
total_costs <- sum(trading_period_df$costs_proceeds[-nrow(trading_period_df)], na.rm = TRUE)
# Calculate final profit for the last day
final_profit_for_period <- total_returns + total_costs + final_costs_proceeds
# Set the final day's 'accumulated_shares' to 0
trading_period_df$accumulated_shares[nrow(trading_period_df)] <- 0
# Set the 'trade_length' column's last value to 'sell'
trading_period_df$trade_length[nrow(trading_period_df)] <- 'sell'
print(trading_period_df)

print(final_profit_for_period)
```


### Step 6: Summarize Your Findings
- Did your P/L and ROI improve over your chosen period?
- Relate your results to a relevant market event and explain why these outcomes may have occurred.


```r
# ROI when implementing profit taking strategy
ROI_profit_strat <- (final_profit_for_period / -total_costs) * 100
print(ROI)
```

Discussion: The first method has the exact same ROI and total_profit_loss as the original algorithm.
AMD released many new products during the span of Jan 1 2021 to Jun 30 2021. On Thursday, March
18 2021, AMD officially announced the RX 6700 XT card and on Monday, 31 may 2021 AMD announced the RX 6000M series of GPUs designed for laptops. Investors got confident with AMD’s releases following these two dates, which led to increase in share prices. The original algorithm and method 1 was better
positioned to take advantage of the increased share prices as it did not restrict selling to a specific profit
margin. Method 2 (direct profit taking strategy), while more safe and protecting against sudden downturns,
was not able to produce a higher profit compared to strategy 1 or Method 1 as it missed out by selling too
early. The original algorithm was able to produce a profit of: 67 172.0006 dollars and Method 2 was able to
produce a profit of: 66 510.5002 dollars. The 661.5004 dollar difference in profit decreased the ROI (from
Original algo to Method 2), with original algo having an ROI of 13.53161 and Method 2 having an ROI of
13.39835.




