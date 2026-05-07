# IMC Prosperity 4 – Alpha Search Team

## About Us

Our approach combined systematic quantitative research with execution-aware trading infrastructure, emphasizing robust signal discovery, market microstructure analysis, and inventory-aware optimization across multiple rounds of competition.

## Core Focus Areas
- Alpha signal research (microstructure, statistical, and cross-asset)
- Systematic strategy development
- Execution optimization and fill-probability modeling
- Risk and inventory management

## Results
- **4th place globally** out of 22,000+ teams in Round 4
- **23rd place** in the Round 3 algorithmic competition
- **27th place** in the Round 2 algorithmic competition

Through iterative research, simulation, and execution optimization, we developed scalable systematic trading strategies designed to perform under highly competitive market conditions.


## Overview

- [Algorithmic Challenge](#algorithmic-challenge)
  - [Round 1 and 2](#round-1-and-2)
  - [Round 3 and 4: Options Trading](#round-3-and-4-Options-Trading)
  - [Round 5: 50 assets tradable](#round-5-50-assets-tradable)

 - [Manual Challenge](#manual-challenge)
   - [Round 1: Walrasian Auction](#round-1-walrasian-auction)
   - [Round 2: Speed](#round-2-speed)
   - [Round 3: Choosing bids](#round-3-choosing-bids)
   - [Round 4: Options Portfolio Optimization](#round-4-options-portfolio-optimization)
   - [Round 5: News-based Portfolio](#round-5-news-based-portfolio)


## Algorithmic Challenge
Before the competition officially started, we prepared by building a backtester that satisfied our requirements. E.g.  we wanted some additional metrics to specify the risk of a trading strategy over another and how likely it is to be overfitted. We also built a dashboard to check if we can see some obvious patterns, like the informed trader "Olivia" in Prosperity 3, who always traded with volume 15. For the dashboard, we also added additional features, which we thought were useful.

### Round 1 and 2
#### ASH COATED OSMIUM
For Osmium, the strategy was completely different from that of Pepper root. We first started our search for an appropriate fair value estimate. For this, we took a similar approach at the Frankfurt Hedgehogs, defining a wall mid, but with volumes as a filter for it. We would forward fill this if a value is missing on either side. We found a pattern of volumes quoted in the market, which was symmetric for bids and asks.

*Figure: Osmium bots over time*
![Osmium Bots](pictures/osmium_bots.png)


Given this and the fact that Osmium was mean reverting we formulated the following strategy. If the ask price is smaller than the wall mid, we take it, analogously for the bid side. Additionally, we do market making with two different modes, depending, if the current quotes in the market are made by market makers or not. We classify the quote into an MM quote if the distance of it to the wall mid is larger than 5. In this case, we post at the best bid + 1 or best ask -1. If the quote isn't by an MM, then we quote around the wall mid shifted by an inventory skew and a half spread of 4. 

*Figure: Osmium over time*
![Osmium](pictures/osmium.png)





#### INTARIAN PEPPER ROOT
Pepper root had a clear pattern with an underlying upward trend and mean reversion around this trend. So the baseline approach for this asset was to just buy as aggressively as possible at the start and hold until the end. One major feature was also that the spread was increasing over time; thus, using an if statement, if the mid price is above the fair value and then selling and buying it back immediately, might not be the best approach. 

*Figure: Pepper root over time*
![Pepper Root](pictures/pepper_root.png)


We structured the trading strategy for this asset into three phases: accumulation, mean reversion and position unwinding.
For the accumulation phase, we wanted to get to the position limit of 80 as fast as possible, but not pay too much for this. E.g. if the third best ask price would be 20 points away from the mid price, we could just wait one time step and, on average, get a better execution price at the best ask (as the rate of the trend isn't that extreme; it was around 0.1 per tick). To optimize this approach, we also took into consideration the current spread and a future time steps mid price. Additionally, we always posted bid orders to match bots who would sell. This kind of strategy we used throughout all rounds: we take all the LOs of the bots quoted, which satisfy a criterion we would want to trade, and if we predict the direction, we use the remaining size to trade to post LOs. To decide where to post those LOs, we analysed at which prices the bots are most likely to post their orders to match us and used an empirical estimate of it.
After we accumulated the maximum volume we are allowed to hold, we went into a mean reversion around the trend strategy phase. We first needed to figure out which fair value estimate we would like to use.
In terms of logic, it would be a bad approach to think that the fair value at time t should be the midprice/microprice or any other common fair value estimate, as the underlying trend would be neglected. So we decided to take a future mid price as our fair value (which we could just easily calculate by the slope times time) and trade the deviations of quoted prices from it. For this, we had two approaches: one using the deviations and one using the spread behaviour.
For the first approach, we had the following logic: If the currently quoted best bid is above a future time steps mid price, we would take it. If this isn't the case, but I could post an ask order such that it is still above this fair value, I would do it and keep track, if it got executed. The second approach is more of a statistical one. It abuses the distribution of the spread, given a specific time frame. For this, we record a certain amount of spread values (not the entire distribution, as the spreads mean increases in time) and if the current spread value is larger than a specific quantile, we post an ask order at the best ask - 1. If it gets executed, we just buy back at the next time steps quoted best ask. On average, this should be highly profitable, as we are trading the outliers of the spread distribution for a time period. 
Lastly, as the spread was increasing and the last time steps mid price is used to settle the position we are holding, we would lose money at the end of the trading period. To reduce this loss, we start posting orders below the best ask to capture any incoming order flow. We start with this process close to the end of the period. In the worst case, if no bot sends out a buy order, we get the same mid price settlement as without using this strategy. 


*Figure: Pepper root spread over time*
![Pepper Root Spread](pictures/pepper_root_spread.png)

What we missed for both assets was that if one side isn't quoted, we could quote extremely wide and still get filled.
Also, we expected a massive regime change due to the simplicity of this task, so we had a generalisation of this strategy to any slope of the trend, also negative implemented. And the regime change prediction was correct, just in another way, as round 3 was completely different from the years before, and the two assets for the first two rounds weren't traded anymore and the PnL was reset to 0.







### Round 3 and 4: Options Trading
For rounds 3 and 4, the assets of previous rounds weren't available to trade anymore. We first looked into potential relationships between Hydrogel and Velvetfruit, but didn't find any besides them both being mean reverting.


#### HYDROGEL PACK and VELVETFRUIT EXTRACT (VFE)
We tried several market making algorithms and different mean reversion approaches, but the market making approach didn't work, as there were too few incoming trades to get rid of the inventory before the market moved away. For the mean reversion part, we tested z-score-based approaches as well as VWAP, but the final solution, which performed best OOS a simple mean over time and taking the deviations from it. To optimize this approach, we used a prior based on the data from the previous days' data to start trading as soon as possible. Additionally, we posted limit orders to maximize the directional position size. We just posted the remaining position size after taking the orders quoted by bots, thus, in the worst case, getting no execution, in the best case, getting an execution in the direction we already wanted to trade at a better price. 

*Figure: Hydrogel and VFE over time*
![Hydrogel VFE](pictures/hydrogel_vfe.png)


#### Options on VFE
There were ten vanilla call options on VFE with strikes 4000, 4500, 5000, 5100, 5200, 5300, 5400, 5500, 6000, 6500.
For the options pricing, it was clear from the Wiki that we should use standard Black-Scholes pricing. The first thing we did was to get a volatility surface and see if there were any anomalies for the Greeks. After that, we checked for any convexity (butterfly spread arbitrage) violations, where there weren't any in the data. The next step was to check if there is any lead-lag relationship to find, given the current option prices and the approximate changes they should follow to the next time step. Again, nothing to be found here. As the underlying VFE was mean reverting, we couldn't trust any of the mean reverting options combinations. Through time, the volatility surface wasn't stable, so we couldn't trade that either. As the price of the underlying VFE was around 5200-5300, 6000 and 6500 strikes were far OTM and traded between 0 and 1. For round 3, we didn't trade those at all and only included them in round 4 after seeing which bots took what kind of trades. For the other options, they all followed the underlying's direction perfectly;  thus, just trading the underlying and doing the same trades with each of the options seemed to be the best option (or choice, if you don't like the word play).
We also briefly tried an IV scalping variant on the vouchers themselves. The setup used a fitted smile, Black-Scholes fair values, and then traded the residual between market mid and smile-implied fair value. We tested both pooled z-score triggers across vouchers and per-voucher absolute residual thresholds around a rolling mean, especially for the strikes around the money such as 5100-5500. The idea was that relative value dislocations might mean revert even if outright price direction was noisy. In practice, the edge was too inconsistent once spread costs, fill uncertainty, and unstable surface dynamics were taken into account, so this remained an experiment rather than part of the final production strategy.

One reason we did not push further on pure smile trading was that even the cleaned fit came at a fairly high cost in sample size. We could get a visually reasonable parabola only after excluding low-extrinsic and low-vega observations, but that removed more than half of the raw points, which made us uncomfortable treating the resulting surface as a robust production signal rather than a diagnostic tool.

*Figure: Filtered Round 3 voucher smile fit — excluded points are grey, retained points blue, fitted parabola in black. The fit looks cleaner after filtering, but only about 45.6% of observations survive the quality screen.*
![Filtered Round 3 voucher smile fit](pictures/round3_filtered_smile_fit.png)



#### Informed/Uninformed Traders

For round 4, we were given the data set with added bot names for each trade. We extended this data set by also getting who is quoting at which price for every time step, and at which prices bots are most likely to take it (only for the website data, thus we had too few data points for the trades analysis).
We tried to analyze which bots make positive EV predictions over several different time frames. We split that into two parts, one observing the quoted side, and classifying if they are informed or not and for the aggressive sides trades. Only Mark 67 seemed to have a clear edge here, but the edge was too insignificant to outperform our round 3 algorithm due to high transaction costs. Some of the bots were also trading only at a specific price (one was always selling at the price of 7). We also tried to classify into informed/uninformed based on the volumes of the Mark's. 
We posted bids at 0 and asks at 1 for the 6000 and 6500 strikes, after seeing that Mark 22 sells them at 0. 

### Round 5: 50 assets tradable
In round 5, due to the enormous amount of tradable assets, we needed to rethink our approach. We first tried to find pairs which would cointegrate and classify the assets into common behaviour. For the former, we found that almost all apparent cointegrations did not hold out of sample.

The main lesson was that basket discovery is a multiple-testing problem. If thousands of candidate baskets are tested, a raw p-value below 0.05 is not strong evidence by itself. In fact, accepting a basket when any one of three folds has p < 0.05 gives a false-positive probability of about 1 - 0.95^3 = 14.3% under a no-edge null. At the scale of our search, this produces many visually plausible but spurious baskets.

To reduce this, we treated the statistical tests as a screening layer, not as the final decision rule. The clean chronological validation was:

- train on day 2, test on day 3
- train on days 2 and 3, test on day 4

We also used leave-one-day-out folds such as train on days 2 and 4, test on day 3, and train on days 3 and 4, test on day 2. Those folds are not a true live trading simulation, because they use future data relative to the test day. We used them only as stability diagnostics: if a basket only worked in one split and broke in the others, it was more likely to be a day-specific accident.

Another way we controlled the search was by moving from arbitrary continuous hedge ratios to a small integer lattice and by searching mostly within product categories. A free OLS basket can always find fragile decimal coefficients that make the in-sample residual look stationary, especially when many unrelated products are tested. Those coefficients are also harder to interpret, harder to size under position limits, and more likely to drift between days. Restricting the search to small round-number weights, usually in a range such as -3 to 3, reduced the number of candidate baskets. Restricting the search to products from the same category added a structural prior: it is more plausible that variants of the same product family share a pricing relationship than that an arbitrary mix of unrelated assets does. Neither restriction eliminates overfitting, but together they reduce the hypothesis space and make the surviving baskets more interpretable: if a relationship only worked with a very specific decimal coefficient or a cross-category mixture with no story, we treated it as less trustworthy than one that survived with simple ratios like 3:2:1 or 1:3:3:1 inside a category.

For each candidate basket, we looked at the train ADF p-value, the held-out test ADF p-value, a same-mean ADF test around the train-fitted mean, the shift in test mean measured in train sigmas, the ratio of test sigma to train sigma, half-life, crossing rate, and after-cost backtest PnL. We also inspected the spread plots. The plots helped catch problems that a p-value can hide, such as slow trending, one large reversal driving the result, a mean shift between days, or a spread that is statistically stationary but too slow or too expensive to trade. However, those plots were diagnostic rather than independent proof; once a plot is selected after a large search, it has the same selection-bias issue as the p-value.

The Bonferroni correction makes the same point. If we correct only across the final shortlist of seven baskets, the threshold is 0.05 / 7 = 0.0071, and UV, Microchip, Snackpack B, Robot, and Translator pass on their descriptive all-day ADF p-values. Sleep and Snackpack A do not. But this is too generous because the shortlist was chosen after a much larger search. Correcting across the true search space would require p-values on the order of 10^-6 or smaller, and essentially none of the discovered baskets should be described as Bonferroni-clean. We therefore used p-values as a sanity check, and relied more on structural simplicity, cross-day PnL consistency, and execution feasibility.

The final basket-style signals we used were:

- Purification Pebbles: hard structural relationships, especially XS + S following a downward time trend, M + L around a fixed level, and XL priced from the rest of the basket.
- UV-Visors: 3 * MAGENTA + 2 * AMBER + RED around 60k. This was the cleanest integer basket.
- Domestic Robots: MOPPING + 3 * VACUUMING + 3 * DISHES + IRONING.
- Sleep Pods: LAMB_WOOL + 2 * SUEDE - 3 * POLYESTER + 2 * COTTON. This was weaker, so we treated it more cautiously.
- Instant Translators: SPACE_GRAY + 2 * ASTRO_BLACK - 3 * ECLIPSE_CHARCOAL + GRAPHITE_MIST + 3 * VOID_BLUE. This was also weaker statistically, but had useful after-cost behaviour.
- Organic Microchips: RECTANGLE - 2 * OVAL + 3 * TRIANGLE, used mainly as a signal to trade TRIANGLE rather than as a fully hedged basket.
- Protein Snack Packs: VANILLA - CHOCOLATE, and PISTACHIO + STRAWBERRY - RASPBERRY, traded more as short-horizon/EMA mean reversion than as perfectly clean long-run cointegration.

We explicitly rejected many statistically tempting baskets in Oxygen, Galaxy Sounds, and Construction Panels because they did not survive the held-out tests well enough or did not have convincing after-cost behaviour. Some products from those categories were still traded directionally or market-made, but not because of a strong cointegration claim. In particular, OXYGEN_SHAKE_GARLIC, GALAXY_SOUNDS_BLACK_HOLES, and PANEL_2X4 showed positive long-only PnL on every historical day we had. The fact that these came from categories where basket mean reversion failed was also useful: it suggested that the edge in those categories was more likely an outright drift/trend effect in a specific product than a stable intra-category relative-value relationship.


#### Purification Pebbles: XS - XL
For Pebbles, there was a strict relationship that all components added had the value 50k. We used this to improve the fair value for XL to do market making around 50k - midprice of the components. For XS and S, there was a relationship similar to Pepper root, but now downwards trending: XS+S = 20k - slope times time. We used the same strategy as for Pepper root here, just with the combination of XS and S. For M and L, we used a mean reversion approach.

#### Organic Microchips
For Microchips, we found a lead-lag relationship between Circle and the other assets, with Circle leading (see the figure below). Knowing such a relationship exists, we only traded the 4 other assets of microchips, excluding Circle. The strategy was simply that if Circle made a large enough directional movement over a specified period, we would trade it in all of the other assets with respect to the lag. For risk management, as it wasn't a perfect correlation, we would exit if the price change didn't behave as expected. Also, we would continue to stay in this position as long as Circle is moving in this direction, if our prediction is correct.

*Figure: Lead-lag relationship of microchips*
![Microchips](pictures/microchips.png)


#### UV-Visors
UV had one of the cointegrations, which held strongly. It had the weights and assets: 3 magenta, 2 amber and 1 red, with a constant mean of around 60k. We traded the deviations from this mean with the appropriate weights and scaled them up to the position limit of 10. In the final trader this was implemented with deterministic integer targets/rounding, not stochastic rounding.

#### Market Making
For every volume, which wasn't used by another strategy, we used market making, if the asset allowed for it. This only uses up to size 2 per asset (tested the optimal ratio). For the market making algorithm itself, we needed to select an approach which has minimal parameters across all of the traded assets. For this, we had a percentage-based approach to make assets comparable and decide on their spread given some simple heuristics, which we can tune. Additionally, due to the fact that some had smaller spreads, we needed some logic for the edge cases, e.g. if the quoted spread is 3 or smaller.





 ## Manual Challenge

 ### Round 1: Walrasian Auction
 There were two auctions, which you can participate in, both with price-time priority, where you were the last person to submit an order before the clearing price was decided to maximize traded volume. The solution was straightforward, if you know how the Walrasian auction works (it wasn't mentioned in the challenge, of course). We knew that after the clearing price is decided, we would be able to settle the position at a specified price, 30 for the first auction and 20 for the second auction. There were two approaches we followed: just looking at the orders of the others in the market and brute-forcing the answer. We did both to double-check our result, and they agreed. To brute-force it, you just need to calculate the clearing price, given the intended trade you will do (with both specified price and volume, over which you loop) and just calculate the PnL of it. Then you take the maximum of those outputs as your intended trade.

### Round 2: Speed

### Round 3: Choosing bids

### Round 4: Options Portfolio Optimization

### Round 5: News-based Portfolio


## Code availability

The full codebase includes:

- Systematic signal research
- Strategy logic (pricing, risk, signal integration)
- Execution and queue-priority handling

> **If you are a recruiter from a quantitative trading firm**, feel free to email me for access to the full implementation and additional technical details.
