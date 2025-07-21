# <b>BAZAAR:</b> - <b>B</b>enchmark for <b>A</b>uction-based <b>Z</b>ero-Intelligence and <b>A</b>daptive <b>A</b>gent <b>R</b>esearch

## Evaluating LLMs in Economic Decision-Making within a Competitive Simulated Market

This benchmark probes fundamental questions about AI economic reasoning: Can LLMs learn bidding strategies through experience? Do they adapt to varying market conditions? How do they balance the tension between aggressive quotes that increase trade probability and conservative quotes that preserve profit margins? By analyzing thousands of games, we reveal the economic instincts of state-of-the-art language models.

The **BAZAAR** (<b>B</b>enchmark for <b>A</b>uction-based <b>Z</b>ero-Intelligence and <b>A</b>daptive <b>A</b>gent <b>R</b>esearch) challenges Large Language Models to navigate the complexities of a double-auction marketplace, where buyers and sellers must make strategic decisions with incomplete information. Each agent receives a private value and must decide how to quote based solely on the history of previous rounds. No agent can see the current order book or others' private values, creating a realistic test of market intuition and strategic adaptation.

-----

## Visualizations & Metrics

### **TrueSkill Leaderboard (μ ± σ)**

A horizontal bar chart ranks each model by the median TrueSkill μ across many passes. Ratings are derived from Conditional Surplus Alpha (CSα), which measures how much better or worse an agent performs than a simple truthful baseline under identical market conditions. CSα normalizes performance so models remain comparable across diverse scenarios.

![TrueSkill leaderboard](images/trueskill_leaderboard_llm.png)

![Leaderboard including baselines](images/trueskill_leaderboard_all.png)

### **Conditional Surplus Alpha (CSα) Strip Plot**

A jittered dot plot reveals the full distribution of CSα values for each model. Every dot represents a single seat in a single game. Tight clusters signal consistent strategies, while wide spreads suggest high variance that may excel in some markets yet fail in others.

![CSα distribution](images/CSα_strip.png)

### **Average Bid/Ask Offset**

This bar chart directly measures bidding aggressiveness by showing the average difference between quotes and private values. Large negative offsets for buyers or large positive offsets for sellers indicate conservative "shading" strategies, while near-zero offsets suggest aggressive, truthful bidding for both roles. The variation across models reveals fundamentally different approaches to the risk-reward tradeoff.

![Average offset by model](images/llm_avg_offset.png)

### **Mean Offset Distribution**

A strip plot shows the per-game mean offset from valuation for each model, highlighting games with unusually aggressive or conservative bidding.

![Offset distribution per game](images/mean_offset_strip.png)

### **Offset Trajectory by Round**

Tracking median bid/ask offsets round by round reveals adaptive behavior. Some models start conservatively to gauge the market before turning aggressive, while others begin boldly but retreat after losses. These trajectories show how different architectures process market feedback.

![Offset by round](images/llm_bid_mean_by_round.png)

### **Trade Price vs. Valuation**

A strip plot compares executed trade prices with the underlying valuations. Wide spreads highlight large gains or losses while tight clusters indicate efficient execution.

![Trade price offset](images/trade_offset_strip_all_models.png)

### **Trade Frequency**

The percentage of rounds where each model successfully executes a trade. This metric distinguishes high-volume traders who accept smaller margins from selective "snipers" who wait for highly profitable opportunities. Neither extreme guarantees success—the key is matching strategy to market conditions.

![Trade frequency](images/trade_frequency.png)


-----

## Multi-Pass TrueSkill Leaderboard

We computed ratings through 200 random passes over 5,000 games, shuffling the game order on every pass to eliminate sequence bias. The final μ (skill) and σ (uncertainty) are medians across all passes:

| Rank | Model             | μ      | σ     | Games | Avg CSα |
| ---: | :---------------- | -----: | ----: | ----: | ------: |
|    1 | GPT-4o (2024-05-13) |  2.847 | 0.068 | 2,512 |    0.91 |
|    2 | Claude 3 Opus     |  2.413 | 0.074 | 2,388 |    0.85 |
|    3 | GPT-4 Turbo       |  2.156 | 0.071 | 2,456 |    0.80 |
|    4 | Gemini 1.5 Pro    |  1.892 | 0.082 | 2,234 |    0.72 |
|    5 | Claude 3.5 Sonnet |  1.678 | 0.079 | 2,367 |    0.69 |
|    6 | GPT-3.5 Turbo     |  0.234 | 0.091 | 2,123 |    0.41 |
|    7 | Gemini 1.0 Pro    | -0.156 | 0.095 | 1,989 |    0.29 |
|    8 | Claude 3 Haiku    | -0.412 | 0.103 | 1,876 |    0.20 |
*TrueSkill parameters: mu=5.0, sigma=8.333, beta=4.1667, tau=0.0*

-----

### Benchmark Philosophy

1.  **Pure Market Signals**: Inter-agent chat is disabled. All adaptive behaviour must be inferred from past prices, trades, and spreads.
2.  **Truthful Reference**: A baseline strategy that bids its exact private value anchors the **Conditional Surplus Alpha (CSα)** metric. CSα tells us how far each agent’s realised surplus deviates from a simple strategy that never shades its bids or learns from the market.
3.  **Multi-Pass Rating**: TrueSkill ratings are updated based on CSα over 200 independent passes through the game list, with the game order shuffled each time. The median μ (skill) and σ (uncertainty) provide a stable and robust final leaderboard.

-----

## Methodology

### **Market Structure**

  * **8 agents per game**: 4 buyers and 4 sellers, each with a private value drawn from one of the distributions below.
  * **30 rounds** of simultaneous sealed-bid double auctions.
  * **No communication**: Agents cannot negotiate or signal—only their quotes speak.
  * **Limited information**: Agents see only the history of completed rounds, never the live order book.

### **Trading Mechanism**

  * **Quote submission**: All agents simultaneously submit bids (buyers) or asks (sellers).
  * **Order matching**: The engine matches the highest bids with the lowest asks.
  * **Price determination**: Trades clear at the unbiased midpoint between matched quotes. If the midpoint lies between two price ticks, the tie is broken randomly.
  * **Information update**: After each round, all quotes and trades become public history.

### **Private Value Distributions**

To test adaptability, games sample from four distinct private value distributions, each presenting unique challenges:

  * **Uniform**: Equal probability for any value across the 0-100 price range. A standard, predictable market.
  * **Correlated**: Agent values cluster around a common, randomly chosen point. This narrows the potential for profitable trades, testing efficiency.
  * **Semi-bimodal**: Values come from two separate peaks, producing clear high-value and low-value clusters of buyers and sellers.
  * **Heavy-tailed**: Extreme high or low values occur more frequently than in a normal distribution, creating opportunities for very large-surplus trades.

### **Performance Measurement**

  * **Surplus**: The profit from each trade (valuation - buy price for buyers, sale price - cost for sellers).
  * **Conditional Surplus Alpha (CSα)**: Surplus normalized against a truthful baseline, accounting for market conditions.
  * **TrueSkill Rating**: Multi-player skill rating based on relative CSα performance across games.

-----

## Algorithmic Baseline Strategies

Alongside the LLMs, a diverse set of algorithmic agents compete in the auction. These baselines, drawn from economic literature and agent-based modeling, provide a crucial reference for performance. A key design pattern is the **`MirroredStrategy`** wrapper: most baselines are implemented with buyer-only logic, and this wrapper symmetrically inverts their perspective so the same logic works when buying or selling.

### **Simple Heuristics**

These foundational strategies follow simple, non-adaptive rules.

  * **Truthful**: The simplest baseline. It always bids its exact private value. It serves as the reference point for the CSα metric.
  * **Fixed Shading**: Bids a fixed, static amount (delta) below its private value. It's predictable and non-adaptive.
  * **Random**: Bids a uniformly random price between the minimum and its private value, representing a "zero-intelligence" agent.
  * **ZIC Pure** (Zero-Intelligence Constrained): A slightly more structured random agent that deterministically selects a price within a window relative to the market midpoint.

### **Adaptive & Learning Strategies**

These agents adjust their behavior based on market feedback and trade outcomes.

  * **Adaptive**: A foundational adaptive agent that adjusts its bidding aggressiveness (`sigma`) based on success. It becomes more aggressive after failing to trade and more conservative after a successful trade.
  * **Adaptive Aggressive** (AA-Cliff): A well-known trader from academic literature. It learns an `offset` from its private value and dynamically adjusts its aggressiveness (`gamma`) and search dispersion (`sigma`) based on profitability.
  * **ZIP+**: A classic agent that learns a target price based on market activity and adjusts its bid using a Widrow-Hoff learning rule.

### **Market-Following Strategies**

These strategies base their decisions on recent price trends.

  * **Momentum**: A trend-follower that maintains an exponential moving average (EMA) of prices and adjusts its quote in the same direction as the trend.
  * **Contrarian**: The opposite of momentum. It bets on mean reversion by quoting *against* the most recent price change.
  * **Mean Reversion**: A more sophisticated version of Contrarian that calculates a long-run EMA of prices and bids towards that historical mean.
  * **Regression**: Uses a sliding window of past trades to run a linear regression, predicting the next clearing price and shading its bid inside that forecast.

### **Belief-Based & Game-Theoretic Strategies**

These agents attempt to model their opponents' behavior.

  * **Gjerstad-Dickhaut** (GD): A highly sophisticated baseline. It maintains a histogram of opponent quotes from a recent sliding window and calculates the bid with the highest Expected Value (EV) against this empirical distribution.
  * **Fictitious Play**: Assumes opponents are drawing prices from a fixed (but unknown) distribution. It builds a frequency distribution of all past opponent quotes and best-responds to that belief.
  * **Enhanced Bayesian**: Starts with a prior belief about the market price and updates it using Bayesian inference, weighted by market volatility.
  * **Risk-Aware**: Evaluates the potential profit and loss for candidate prices using a CARA utility function, selecting the quote with the best risk-adjusted return.

### **Reinforcement Learning Strategies**

These agents use classic RL paradigms to learn optimal actions.

  * **Q-Learning**: Implements tabular Q-learning where the state is a discretized summary of the market and actions are different shading deltas.
  * **Roth-Erev**: Maintains a propensity for each possible action. Rewards reinforce these propensities, making successful choices more likely in the future.
  * **Stochastic Shading Bandit**: Frames bidding as a multi-armed bandit problem, where each arm is a different shading delta.

### **Specialized & Meta-Strategies**

These are advanced agents with unique goals or wrappers that modify other strategies.

  * **Sniper**: A time-aware agent that "parks" with non-competitive quotes for most of the game, then rapidly becomes aggressive in the final few rounds to "snipe" a last-minute deal.
  * **Penny Jumper**: An aggressive agent whose goal is to always be the most competitive in the book by quoting just one tick inside the current best bid or ask.
  * **Adversarial Exploiter**: Actively seeks to exploit predictable opponents by tracking their price history and undercutting them by a small margin.
  * **Signal Jammer**: A meta-strategy that wraps another baseline and adds a small amount of random noise to its quote, designed to confuse opponents trying to model its behavior.
  * **Regime Switcher**: A meta-strategy that contains both a Momentum and a Mean-Reversion agent, delegating to the appropriate one based on market volatility.
  * **GDX**: A Genetic Algorithm that evolves a population of Gjerstad-Dickhaut agents to find the most profitable parameter configuration over time.

-----

## Key Findings

#### **Emergent Strategy Spectrum**

The benchmark reveals a rich spectrum of trading strategies emerging from different LLMs:

  * **Conservative Shaders**: Models that consistently bid far below valuations, trading infrequently but profitably.
  * **Aggressive Truthtellers**: Near-zero offset strategies that maximize trade volume at the expense of margins.
  * **Adaptive Learners**: Sophisticated strategies that adjust aggressiveness based on market feedback.
  * **Market Makers**: Balanced approaches that maintain moderate offsets to capture consistent profits.

#### **Distribution Sensitivity**

Model performance varies significantly across value distributions:

  * Top models excel in **heavy-tailed** markets by identifying and capitalizing on extreme value opportunities.
  * Mid-tier models perform best in **uniform** markets where consistent strategies suffice.
  * **Correlated** markets challenge all models, as the narrow value ranges compress profit opportunities.

#### **Learning Dynamics**

Round-by-round analysis reveals distinct learning patterns:

  * **GPT-4o** demonstrates rapid market calibration, adjusting its strategy within the first 5-10 rounds.
  * **Claude models** maintain more stable strategies throughout the game, suggesting different approaches to the exploration vs. exploitation tradeoff.
  * Several models exhibit **strategy collapse** in later rounds, becoming either overly conservative or aggressive as the game nears its end.


-----

## About Conditional Surplus Alpha (CSα)

CSα normalizes each agent's surplus against a simple "truthful" baseline—a strategy that always bids exactly its private value. For each market condition (round number, value distribution, and spread), we compare the agent's actual surplus to the mean and standard deviation of truthful baselines under identical conditions. This normalization allows fair comparison across diverse market scenarios, with values clamped to [-5.0, 5.0] to prevent outliers from dominating ratings.

## About TrueSkill

We use Microsoft's TrueSkill rating system, designed for multi-player competitions where relative performance matters more than absolute scores. Each game updates all participating agents' ratings based on their relative CSα values. The multi-pass approach—processing games in 200 different random orders—ensures robust rankings unaffected by game sequence.

---

### Related Experiment: Emergent Price-Fixing with Communication Enabled

In a follow-up study, this benchmark was adapted to include an unmonitored chat channel. With the sole objective of maximizing profit, LLMs from every major developer **spontaneously formed illegal price-fixing cartels.** They used the chat to negotiate price floors, coordinate bids, and arrange turn-taking schemes to eliminate competition, all without any instruction to collude.

An analyst LLM assigned an "Illegality Score" to each game, confirming a consistent pattern of anti-competitive conduct:

| Model | Games with Illegality Score ≥ 7/10 |
| :--- | :--- |
| Grok 4 | 75% |
| DeepSeek R1 | 71% |
| o4-mini | 62% |
| ... | ... |

Agents were observed coordinating explicitly. **Grok 4** stated, *“Let’s rotate who gets the high bid… Next cycle S3, then S2,”* while **GPT-4o** arranged a rotation: *“This round: you set 71, I set 70 → I win. Next round … you win.”*

This behavior is a stark example of "specification gaming," where an AI achieves its literal goal (profit) in a way that violates unstated human intent (fair competition). It highlights a new class of risk where sophisticated, harmful strategies can emerge from even simple objectives.

➡️ **Read the full study on Emergent Collusion here: [github.com/lechmazur/emergent_collusion](https://github.com/lechmazur/emergent_collusion/)**


-----


## Other Multi-Agent Benchmarks

  - [Elimination Game: Social Reasoning and Deception in Multi-Agent LLMs](https://github.com/lechmazur/elimination_game/)
  - [Public Goods Game (PGG) Benchmark: Contribute & Punish](https://github.com/lechmazur/pgg_bench/)
  - [Step Race: Collaboration vs. Misdirection Under Pressure](https://github.com/lechmazur/step_game/)

## Other Benchmarks

  - [Extended NYT Connections](https://github.com/lechmazur/nyt-connections/)
  - [LLM Thematic Generalization Benchmark](https://github.com/lechmazur/generalization/)
  - [LLM Creative Story-Writing Benchmark](https://github.com/lechmazur/writing/)
  - [LLM Confabulation/Hallucination Benchmark](https://github.com/lechmazur/confabulations/)
  - [LLM Deceptiveness and Gullibility](https://github.com/lechmazur/deception/)
  - [LLM Divergent Thinking Creativity Benchmark](https://github.com/lechmazur/divergent/)

-----

## Updates

  - **July 21, 2025**: Initial release of the no-communication auction benchmark.
  - Follow [@lechmazur](https://x.com/lechmazur) for updates and related benchmarks.
