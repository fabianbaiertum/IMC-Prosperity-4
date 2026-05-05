# IMC Prosperity 4 – Alpha Search Team

## About us: with linkedin maybe? 

Summary of our systematic trading approach across Rounds 1–5.
Focus: alpha generation, algorithm design and execution modeling.

- Final result: 4th / 22,000+ teams (Round 4), 23th Round 3 algo 25th overall, 27th algo Round 2
- Core approach:
  - Systematic signal research across asset classes
  - Execution-aware strategy design (fill probability modeling)
  - Iterative refinement across rounds

- Key components:
  - Alpha signals (microstructure, statistical, cross-asset)
  - Strategy development
  - Execution optimization
  - Risk and inventory management

## Overview

- [Algorithmic Challenge](#algorithmic-challenge)
  - [Round 1 and 2](#round-1-and-2)
  - [Round 3 and 4: Options Trading](#round-3-and-4-Options-Trading)
  - [Round 5: 50 assets tradable](#round-5-50-assets-tradable)

 - [Manual Challenge](#manual-challenge)
   - [Round 1: Walrasian Auction](#round-1-walrasian-auction)


## Algorithmic Challenge

### Round 1 and 2
#### ASH COATED OSMIUM

Maybe Giovanni/Caleb do this one.




#### INTARIAN PEPPER ROOT
Pepper root had a clear pattern with an underlying upward trend and mean reversion around this trend. So the baseline approach for this asset was to just buy as aggressively as possible at the start and hold until the end. One major feature was also that the spread was increasing over time; thus, using an if statement, if the mid price is above the fair value and then selling and buying it back immediately, might not be the best approach. 

We structured the trading strategy for this asset into three phases: accumulation, mean reversion and position unwinding.
For the accumulation phase, we wanted to get to the position limit of 80 as fast as possible, but not pay too much for this. E.g. if the third best ask price would be 20 points away from the mid price, we could just wait one time step and, on average, get a better execution price at the best ask (as the rate of the trend isn't that extreme; it was around 0.1 per tick). To optimize this approach, we also took into consideration the current spread and a future time steps mid price. Additionally, we always posted bid orders to match bots who would sell. This kind of strategy we used throughout all rounds: we take all the LOs of the bots quoted, which satisfy a criterion we would want to trade, and if we predict the direction, we use the remaining size to trade to post LOs. To decide where to post those LOs, we analysed at which prices the bots are most likely to post their orders to match us and used an empirical estimate of it.
After we accumulated the maximum volume we are allowed to hold, we went into a mean reversion around the trend strategy phase. We first needed to figure out which fair value estimate we would like to use.
In terms of logic, it would be a bad approach to think that the fair value at time t should be the midprice/microprice or any other common fair value estimate, as the underlying trend would be neglected. So we decided to take a future mid price as our fair value (which we could just easily calculate by the slope times time) and trade the deviations of quoted prices from it. For this, we had two approaches: one using the deviations and one using the spread behaviour.
For the first approach, we had the following logic: If the currently quoted best bid is above a future time steps mid price, we would take it. If this isn't the case, but I could post an ask order such that it is still above this fair value, I would do it and keep track, if it got executed. The second approach is more of a statistical one. It abuses the distribution of the spread, given a specific time frame. For this, we record a certain amount of spread values (not the entire distribution, as the spreads mean increases in time) and if the current spread value is larger than a specific quantile, we post an ask order at the best ask - 1. If it gets executed, we just buy back at the next time steps quoted best ask. On average, this should be highly profitable, as we are trading the outliers of the spread distribution for a time period. 
Lastly, as the spread was increasing and the last time steps mid price is used to settle the position we are holding, we would lose money at the end of the trading period. To reduce this loss, we start posting orders below the best ask to capture any incoming order flow. We start with this process close to the end of the period. In the worst case, if no bot sends out a buy order, we get the same mid price settlement as without using this strategy. 

Also, we expected a massive regime change due to the simplicity of this task, so we had a generalisation of this strategy to any slope of the trend, also negative implemented. And the regime change prediction was correct, just in another way, as round 3 was completely different from the years before, and the two assets for the first two rounds weren't traded anymore.







### Round 3 and 4: Options Trading
#### HYDROGEL PACK


#### VELVETFRUIT EXTRACT (VFE)


#### Options on VFE
There were ten options on VFE with strikes 4000, 4500, 5000, 5100, 5200, 5300, 5400, 5500, 6000, 6500
For the options pricing, it was clear from the Wiki that we should use standard Black-Scholes pricing. The first thing we did was to get a volatility surface and see if there were any anomalies for the Greeks. After that, we checked for any convexity (butterfly spread arbitrage) violations, where there weren't any in the data. The next step was to check if there is any lead-lag relationship to find, given the current option prices and the approximate changes they should follow to the next time step. Again, nothing to be found here. As the underlying VFE was mean reverting, we couldn't trust any of the mean reverting options combinations




#### Informed/Uninformed Traders

We posted bids at 0 and asks at 1 for the 6000 and 6500 strikes, after seeing that Mark 22 sells them at 0.

### Round 5: 50 assets trabable
we did this ...





 ## Manual Challenge

 ### Round 1: Walrasian Auction
 

## Core Framework

### Signal Research
- Cross-sectional and time-series signals
- Feature engineering: imbalance, microprice, spread dynamics

### Strategy Types
- Market making
- Mean reversion around an underlying trend 
- Statistical arbitrage and pairs trading
- Options-based strategies

### Execution Modeling
- Fill probability estimation
- Queue positioning and priority
- Trade-off: aggressiveness vs adverse selection



