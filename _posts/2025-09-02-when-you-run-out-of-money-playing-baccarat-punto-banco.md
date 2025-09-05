---
layout: post
title: "When You Run Out of Money Playing Baccarat (Punto Banco)"
date: 2025-09-02 04:56:28 +0200
tags: gambling
---

If someone tells you about «a secret baccarat strategy» with a win rate of around 50%, it is a lie!

![Baccarat table in casino (GPT-4o Image Generation)](/assets/2025-09-02/00-baccarat-table.jpg)

_Baccarat table in casino (GPT-4o Image Generation)_

Baccarat is a highly popular card game found in casinos worldwide, from Macau to Las Vegas. Punto banco is a simplified version of baccarat. During the game of punto banco, the player’s moves are restricted by drawn cards. He simply chooses a hand: Punto (player), Banco (banker), or Égalité (tie), and then the entire process unfolds automatically with no skill required, unlike the _chemin de fer_ version of baccarat, where the player can make a decision to receive the third card. It was this version that [James Bond played](https://jamesbond.fandom.com/wiki/Baccarat).

Baccarat (punto banco) is fundamentally a game of chance where each hand is statistically independent of the previous outcomes, and there is no strategy to beat the house in the long run. If that’s really the case (that you cannot beat the house), I want to find out how many rounds you can play before you run out of money (bankroll).

To solve this question, I need the game itself (the rules are given below), a simulator (the source code is available in the [GitHub repository](https://github.com/adequatica/punto-banco-golango)), and a list of different baccarat strategies for the simulations.

Table of contents:

- [Game Rules](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#game-rules)
- [Simulator Features](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#simulator-features)
- [Baccarat (Punto Banco) Strategies Simulations](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#baccarat-punto-banco-strategies-simulations)
  - Flat Betting Strategies
    - [Always Bet on Punto (Player) or Always Bet Banco (Banker)](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#always-bet-on-punto-player-or-always-bet-banco-banker)
    - [Always Bet on Égalité (Tie)](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#always-bet-on-égalité-tie)
    - [Bet on Last Hand](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#bet-on-last-hand)
    - [Bet on Random](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#bet-on-random)
  - Progression Strategies
    - [Martingale System](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#martingale-system)
    - [Paroli](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#paroli)
    - [Fibonacci](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#fibonacci)
    - [D’Alembert](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#dalembert)
    - [1-3-2-6](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#1-3-2-6)
- [Summary](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#summary)
  - [Threats to Validity](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#threats-to-validity)
  - [References](https://adequatica.github.io/2025/09/02/when-you-run-out-of-money-playing-baccarat-punto-banco.html#references)

## Game Rules

Punto banco is a simplified version of baccarat where the drawn cards determine each move. The game proceeds according to fixed rules, and players make no decisions during the coup (round).

The game uses standard baccarat card values:

- Ace = 1
- 2–9 = pip value
- 10 and face cards (Jack, Queen, and King) = 0

The objective is to predict which of the two hands, the Punto (player) or the Banco (banker), will have a total closest to nine.

Player bets on either:

- Punto (player)
- Banco (banker)
- Égalité (tie)

The dealer deals two cards to both the player and the banker.

**If the total of a hand is 10 or more, only the last digit is counted** ([modulo](https://en.wikipedia.org/wiki/Modulo) 10).

If either the player or the banker (or both) has a total of 8 or 9, the round ends immediately; it is referred to as a «natural».

If neither hand has a natural, additional cards may be drawn according to a fixed set of rules known as the «tableau». Players do not make decisions on drawing cards; the dealer follows these rules automatically.

Player’s third card rules:

- If the player’s total is 0–5, they draw a third card.
- If the player’s total is 6 or 7, they stand.

Banker’s third card rules:

- If the player stands (has two cards with a total of 6 or 7), the banker acts according to the same rule as the player: draws on 0–5 and stands on 6 or 7.
- If the player draws a third card, the banker’s decision depends on their own total and the value of the player’s third card:
- Banker total 0–2: Draws a third card regardless of the player’s third card.
- Banker total 3: Draws a third card unless the player’s third card is an 8.
- Banker total 4: Draws a third card if the player’s third card is 2, 3, 4, 5, 6, or 7.
- Banker total 5: Draws a third card if the player’s third card is 4, 5, 6, or 7.
- Banker total 6: Draws a third card if the player’s third card is 6 or 7.
- Banker total 7: Stands.

After all cards are drawn according to the «tableau», the **hand with the total closest to nine wins.**

## Simulator Features

> _The simulator for this article is written on Go and presented in the [GitHub repository](https://github.com/adequatica/punto-banco-golango) along with the punto blanco game._

This simulator runs the punto banco game, and during each round, it bets on Punto (player), Banco (banker), or Égalité (tie) depending on the chosen strategy.

«The game» is a game session, in which the **simulation starts with the bankroll of $1000** and ends when it cannot afford to bet the next bet.

**The default bet in the simulator is $10**, because it is the [minimum bet for baccarat in Las Vegas](https://vegasadvantage.com/las-vegas-table-game-survey/baccarat/). Therefore, for flat betting strategies, the simulator bets 1% of its initial bankroll in each round.

The simulator features a logic-based betting approach: the game ends if the current simulator’s bankroll falls below the amount required for the next round.

- In flat betting strategies, the game ends when the bankroll becomes 0.
- In progression strategies, the game ends when the number of consecutive wins becomes too favorable (or too negative) that the simulator needs to bet more than the bankroll allows. In this case, the bankroll can be higher than 0 (even too big for edge cases).

Payouts (or pop-up of the bankroll in the context of the simulator) in simulation are made according to standard baccarat rules:

- **1-to-1** on Punto bets.
- **19-to-20** on Banco bets (5% commission is designed to balance Banco’s statistical advantage of a slightly higher probability of winning).
- **8-to-1** on Égalité bet.

**Each strategy has been run on 1.000.000 game simulations.**

The statistics of simulations include the following items:

- Mean rounds per game session until the moment when the gambler can no longer bet.
- Minimum number of played rounds per game session across all simulations.
- Maximum number of played rounds per game session across all simulations.
- Mean wins per game session.
- Minimum wins per session across all simulations. For progression strategies, if a gambler gets into a series of losses, he may run out of bankroll before the first win, and therefore, the minimum number of wins will be 0.
- Maximum wins per game session across all simulations.
- Win rate — the percentage of rounds that a gambler wins over the number of played rounds. It is the way to measure the effectiveness of a strategy.
- The rate of zero-win games indicates the percentage of game sessions that ended without a single win occurring. It may serve as an indicator of the amount of risk associated with a strategy.
- Mean winning streak.
- Maximum winning streak per game session across all simulations.
- Mean losing streak.
- Maximum losing streak per game session across all simulations.
- Mean peak bankroll per game session.
- Maximum recorded bankroll across all simulations. It is the maximum winning amount that occurred in the simulation session of a chosen strategy.
- Profitable games — the percentage of game sessions with a profit opportunity, or the percentage of game sessions in which the bankroll exceeded 101% of the initial value. It shows the percentage of games in which the gambler hit a profit target (in this case $1010 and above) and could have been in profit (won money) if he had stopped betting.
- Profitably ended games — the percentage of game sessions ended with profit, or the percentage of game sessions that end when the current bankroll exceeds 101% of the initial value. This edge case was explained above.

Now that the game’s simulator has been explained, let’s dive into the strategies of baccarat (punto banco), which have been simulated.

## Baccarat (Punto Banco) Strategies Simulations

All baccarat (punto banco) strategies can be divided into two types: flat betting and progression betting strategies.

### Flat Betting Strategies

In flat betting strategies, a gambler bets the same amount every hand (e.g., always $10 per round).

### Always Bet on Punto (Player) or Always Bet Banco (Banker)

These are the simplest strategies for playing punto banco; just always stick to one side during the game.

| Strategy                       | Bet on Punto | Bet on Banco | Average |
| ------------------------------ | ------------ | ------------ | ------- |
| Mean rounds per game           | 930.8        | 941.6        | 936.2   |
| Minimum played rounds per game | 260          | 265          | 262.5   |
| Maximum played rounds per game | 3344         | 3377         | 3360.5  |
|                                |              |              |         |
| Mean wins per game             | 415.4        | 431.9        | 423.65  |
| Minimum wins per game          | 80           | 85           | 82.5    |
| Maximum wins per game          | 1622         | 1681         | 1651.5  |
| Win rate                       | 44.13%       | 45.38%       | 44.76%  |
| Rate of zero-wins games        | 0%           | 0%           | 0%      |
|                                |              |              |         |
| Mean winning streak            | 7.8          | 8.1          | 7.95    |
| Maximum winning streak         | 25           | 26           | 25.5    |
| Mean losing streak             | 10.7         | 10.3         | 10.5    |
| Maximum losing streak          | 34           | 33           | 33.5    |
|                                |              |              |         |
| Mean peak bankroll per game    | 1041.57      | 1040.59      | 1041.08 |
| Maximum recorded bankroll      | 1840         | 1776.5       | 1808.25 |
| Profitable games               | 65.01%       | 68.07%       | 66.54%  |
| Profitably ended games         | 0%           | 0%           | 0%      |

With a bankroll of $1000 and a bet amount of $10, a gambler will run out of bankroll in an average of 936 rounds.

Betting always on Banco is more optimal because it has a slightly better win rate: 45,38% instead of 44,13% in the case of betting always on Punto, but a 5% commission on payoffs undermines the profit of this slight advantage.

### Always Bet on Égalité (Tie)

This is the only strategy a gambler must avoid due to its poor win rate.

| Strategy                       | Bet on Égalité |
| ------------------------------ | -------------- |
| Mean rounds per game           | 692.3          |
| Minimum played rounds per game | 100            |
| Maximum played rounds per game | 9820           |
|                                |                |
| Mean wins per game             | 65.8           |
| Minimum wins per game          | 0              |
| Maximum wins per game          | 1080           |
| Win rate                       | 8.79%          |
| Rate of zero-wins games        | 0%             |
|                                |                |
| Mean winning streak            | 2.3            |
| Maximum winning streak         | 8              |
| Mean losing streak             | 46             |
| Maximum losing streak          | 144            |
|                                |                |
| Mean peak bankroll per game    | 1233.56        |
| Maximum recorded bankroll      | 6390           |
| Profitable games               | 81.94%         |
| Profitably ended games         | 0%             |

Don’t be fooled by 81,94% of profitable games and a maximum recorded bankroll (but it was one time in a million simulations). Due to a high payout (8:1), a bankroll may exceed the initial bankroll during the game, but in the long run, a gambler will run out of bankroll in 692 rounds. It’s 1,35 times faster than betting always on Punto or Banco.

### Bet on Last Hand

This is an attempt to improve the «always bet on one side» strategy. The mechanics are that if Banco won the previous hand, a gambler bets on Banco for the current hand. If Punto won the previous hand, a gambler bets on Punto for the current hand, etc.

| Strategy                       | Bet on Last Hand | Bet on Last Hand PB | Average |
| ------------------------------ | ---------------- | ------------------- | ------- |
| Mean rounds per game           | 903.6            | 934.5               | 919.05  |
| Minimum played rounds per game | 178              | 240                 | 209     |
| Maximum played rounds per game | 4422             | 3571                | 3996.5  |
|                                |                  |                     |         |
| Mean wins per game             | 378.2            | 423.4               | 400.8   |
| Minimum wins per game          | 40               | 71                  | 55.5    |
| Maximum wins per game          | 1953             | 1761                | 1857    |
| Win rate                       | 41.34%           | 44.82%              | 43.08%  |
| Rate of zero-wins games        | 0%               | 0%                  | 0%      |
|                                |                  |                     |         |
| Mean winning streak            | 7.8              | 8                   | 7.9     |
| Maximum winning streak         | 24               | 25                  | 24.5    |
| Mean losing streak             | 11.9             | 10.5                | 11.2    |
| Maximum losing streak          | 36               | 32                  | 34      |
|                                |                  |                     |         |
| Mean peak bankroll per game    | 1065.57          | 1041.04             | 1053.31 |
| Maximum recorded bankroll      | 2057.5           | 1636                | 1846.75 |
| Profitable games               | 71.06%           | 66.95%              | 69.01%  |
| Profitably ended games         | 0%               | 0%                  | 0%      |

For «Bet on Last Hand PB», a gambler switches to Banco in case Égalité won the previous hand, and this strategy performs somewhere in between «Always bet on Punto» and «Always bet on Banco», but in the end, a gambler will run out of bankroll in about 935 rounds.

### Bet on Random

Because baccarat is a game of chance, and no strategy can guarantee consistent wins, maybe a random strategy can deal with it? Unfortunately, not.

| Strategy                       | Bet on Random PB |
| ------------------------------ | ---------------- |
| Mean rounds per game           | 933.8            |
| Minimum played rounds per game | 243              |
| Maximum played rounds per game | 3849             |
|                                |                  |
| Mean wins per game             | 422.5            |
| Minimum wins per game          | 73               |
| Maximum wins per game          | 1899             |
| Win rate                       | 44.75%           |
| Rate of zero-wins games        | 0%               |
|                                |                  |
| Mean winning streak            | 8                |
| Maximum winning streak         | 26               |
| Mean losing streak             | 10.5             |
| Maximum losing streak          | 32               |
|                                |                  |
| Mean peak bankroll per game    | 1041.17          |
| Maximum recorded bankroll      | 1660.5           |
| Profitable games               | 67.1%            |
| Profitably ended games         | 0%               |

Because Égalité is the worst choice for betting, I excluded it from random betting. The simulator randomly bets only on Punto or Banco (hence its name, «Bet on random PB»). So, its win rate is similar to that of the same semi-random «Bet on Last Hand PB».

Thus, among all flat betting strategies, «Bet Always on Banco» is the best way to stretch the game before running out of bankroll.

### Progression Strategies

In progression betting strategies, a gambler adjusts the bet size based on previous outcomes.

### Martingale System

The [martingale betting system](<https://en.wikipedia.org/wiki/Martingale_(betting_system)>) is a classic betting strategy that dates back to the 18th century. The core idea behind the martingale is to **double your bet every time you lose**, with the goal of recovering all previous losses and securing a profit when you eventually win.

This strategy is also categorized as a negative progression system, meaning you increase your bet after a loss and decrease it after a win.

Explanation of the martingale system algorithm:

1. Place an initial small bet on either Punto or Banco hand.
2. If your bet wins, your stake remains the same, and you repeat the original bet amount for the next hand. You are in profit.
3. If your bet loses, you double your stake for the next hand.
4. Continue doubling your bet after each subsequent loss.
5. Once you win a hand, you return to your original base bet amount and start the process over again. The theory is that this single win will recover all previous losses and provide a profit equal to your initial betting unit.

| Strategy                       | Martingale on Punto | Martingale on Banco | Average  |
| ------------------------------ | ------------------- | ------------------- | -------- |
| Mean rounds per game           | 112.1               | 123.5               | 117.8    |
| Minimum played rounds per game | 6                   | 6                   | 6        |
| Maximum played rounds per game | 11800               | 9744                | 10772    |
|                                |                     |                     |          |
| Mean wins per game             | 49.9                | 56.7                | 53.3     |
| Minimum wins per game          | 0                   | 0                   | 0        |
| Maximum wins per game          | 5364                | 4501                | 4932.5   |
| Win rate                       | 39.27%              | 40.69%              | 39.98%   |
| Rate of zero-wins games        | 2.88%               | 2.52%               | 2.7%     |
|                                |                     |                     |          |
| Mean winning streak            | 4.4                 | 4.7                 | 4.55     |
| Maximum winning streak         | 23                  | 24                  | 23.5     |
| Mean losing streak             | 6.5                 | 6.5                 | 6.5      |
| Maximum losing streak          | 12                  | 11                  | 11.5     |
|                                |                     |                     |          |
| Mean peak bankroll per game    | 1498.69             | 1453.32             | 1476.01  |
| Maximum recorded bankroll      | 35290               | 31306.5             | 33298.25 |
| Profitable games               | 94.35%              | 94.85%              | 94.6%    |
| Profitably ended games         | 5.26%               | 4.49%               | 4.88%    |

Due to a negative progression, this strategy has a poor win rate of approximately 40% and significantly devastates the bankroll. A gambler may run out of bankroll only in 118 rounds. It’s 7,9 times faster than betting always on Punto or Banco. In 2,7% of game sessions, a game may end (bankroll will be emptied) without a single win, just six rounds, and you are done.

On the other hand, this risky strategy may bring profit. In 0,48% of game sessions, a game ended profitably and resulted in a record bankroll of $35290 (but this occurred only once in a million simulations). This edge case was described in the section of the simulator’s logic.

### Paroli

The paroli system, **also known as the «Reverse Martingale», is a positive progression betting system.** Unlike negative progression systems, which increase bets after losses, the paroli system encourages gamblers to **increase bets after each win.** The name comes from the word «[par](https://www.etymonline.com/word/par)», which is Latin for «equal». This is a reference to how the bets are doubled in each subsequent round. Its core idea is to capitalize on winning streaks and maximize profits from those streaks, rather than chasing losses.

Explanation of the paroli algorithm:

1. Place an initial small bet on either Punto or Banco hand.
2. If your initial bet wins, you double your stake for the next hand.
3. Continue doubling your bet after each consecutive win.
4. If you lose a hand at any point, you return to your original base bet amount for the next hand and start the process over again. This strategy aims to limit losses to your initial betting unit, as you are primarily risking accumulated winnings.

| Strategy                       | Paroli on Punto | Paroli on Banco | Average |
| ------------------------------ | --------------- | --------------- | ------- |
| Mean rounds per game           | 733.5           | 734.7           | 734.1   |
| Minimum played rounds per game | 199             | 188             | 193.5   |
| Maximum played rounds per game | 3787            | 4133            | 3960    |
|                                |                 |                 |         |
| Mean wins per game             | 327.3           | 337             | 332.15  |
| Minimum wins per game          | 57              | 55              | 56      |
| Maximum wins per game          | 1829            | 2058            | 1943.5  |
| Win rate                       | 44.05%          | 45.29%          | 44.67%  |
| Rate of zero-wins games        | 0%              | 0%              | 0%      |
|                                |                 |                 |         |
| Mean winning streak            | 7.5             | 7.7             | 7.6     |
| Maximum winning streak         | 26              | 32              | 29      |
| Mean losing streak             | 10.2            | 9.9             | 10.05   |
| Maximum losing streak          | 38              | 33              | 35.5    |
|                                |                 |                 |         |
| Mean peak bankroll per game    | 1073.01         | 1072.29         | 1072.65 |
| Maximum recorded bankroll      | 2090            | 2213            | 2151.5  |
| Profitable games               | 71.65%          | 74.21%          | 72.93%  |
| Profitably ended games         | 0%              | 0%              | 0%      |

«Paroli on Banco» has the closest win rate to «Always on Banco», but it still falls short of the simplest flat betting strategy. However, the rate of profitable games and maximum recorded bankroll are higher, which can be falsely perceived by risky gamblers as a «better» strategy. A gambler will still run out of bankroll in 734 rounds, or a quarter faster than betting always on Punto or Banco.

### Fibonacci

The Fibonacci strategy is a **negative progression betting system that uses the famous [Fibonacci sequence](https://en.wikipedia.org/wiki/Fibonacci_sequence) to determine bet sizes.** In this mathematical sequence, each number is the sum of the two preceding numbers, starting typically with 0 and 1 (e.g., 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, etc.). The system is designed to help gamblers gradually recover losses rather than all at once, as a single win will move you back in the sequence, offsetting previous losses.

Explanation of the Fibonacci algorithm:

1. Establish a base betting unit and bet on either Punto or Banco hand.
2. If your bet loses, you move forward one step in the Fibonacci sequence to determine the stake for your next bet. For example, if you bet $10 and lose, your next bet would remain $10 (as the sequence starts with 1, 1). If you lose again, your next bet would be $20 (moving to the «2» in the sequence 1, 1, 2).
3. If your bet wins, you move back two steps in the Fibonacci sequence to determine your next bet amount. For example, if you won a $30 bet (representing «3» in the sequence 1, 1, 2, 3), you would move back two steps to the «1» in the sequence, making your next bet $10.
4. Continue this pattern of increasing bets after a loss and decreasing them after a win.

| Strategy                       | Fibonacci on Punto | Fibonacci on Banco | Average |
| ------------------------------ | ------------------ | ------------------ | ------- |
| Mean rounds per game           | 205.1              | 238.4              | 221.75  |
| Minimum played rounds per game | 9                  | 9                  | 9       |
| Maximum played rounds per game | 7354               | 7553               | 7453.5  |
|                                |                    |                    |         |
| Mean wins per game             | 91.5               | 109.4              | 100.45  |
| Minimum wins per game          | 0                  | 0                  | 0       |
| Maximum wins per game          | 3361               | 3534               | 3447.5  |
| Win rate                       | 41.01%             | 42.52%             | 41.76%  |
| Rate of zero-wins games        | 0.49%              | 0.41%              | 0.45%   |
|                                |                    |                    |         |
| Mean winning streak            | 5.3                | 5.7                | 5.5     |
| Maximum winning streak         | 26                 | 21                 | 23.5    |
| Mean losing streak             | 7.7                | 7.7                | 7.7     |
| Maximum losing streak          | 14                 | 12                 | 13      |
|                                |                    |                    |         |
| Mean peak bankroll per game    | 1290.54            | 1229.03            | 1259.79 |
| Maximum recorded bankroll      | 11150              | 5919.5             | 8534.75 |
| Profitable games               | 92.82%             | 92.8%              | 92.81%  |
| Profitably ended games         | 0.19%              | 0.02%              | 0.11%   |

The Fibonacci strategy is less aggressive and carries a lower risk than the Martingale system. But anyway, a gambler will run out of bankroll in 222 rounds, or four times faster than betting always on Punto or Banco

### D’Alembert

The D’Alembert system is one of the oldest strategies named after the [French mathematician](https://en.wikipedia.org/wiki/Jean_le_Rond_d%27Alembert) who defined it in the 18th century. It is a negative progression betting system, the core principle of which is to **balance wins and losses over a gaming session.**

Explanation of D’Alembert algorithm:

1. Establish a base betting unit — for example, $10.
2. Place an opening bet equal to your chosen base amount.
3. If your bet loses, you increase your stake by one unit for the next bet. For example, if you bet $10 and lose, your next bet would be $20.
4. If your bet wins, you decrease your stake by one unit for the next bet. For example, if you win the $20 bet, your next bet would return to $10. This gradual method aims to slowly recoup losses through a series of smaller wins, rather than risking a large amount to recover everything at once.

| Strategy                       | D’Alembert on Punto | D’Alembert on Banco | Average |
| ------------------------------ | ------------------- | ------------------- | ------- |
| Mean rounds per game           | 113.4               | 125.6               | 119.5   |
| Minimum played rounds per game | 13                  | 13                  | 13      |
| Maximum played rounds per game | 2098                | 2282                | 2190    |
|                                |                     |                     |         |
| Mean wins per game             | 50.6                | 57.6                | 54.1    |
| Minimum wins per game          | 0                   | 0                   | 0       |
| Maximum wins per game          | 1038                | 1136                | 1087    |
| Win rate                       | 41.13%              | 42.39%              | 41.76%  |
| Rate of zero-wins games        | 0.05%               | 0.03%               | 0.04%   |
|                                |                     |                     |         |
| Mean winning streak            | 4.8                 | 5.1                 | 4.95    |
| Maximum winning streak         | 21                  | 26                  | 23.5    |
| Mean losing streak             | 6.9                 | 6.9                 | 6.9     |
| Maximum losing streak          | 20                  | 19                  | 19.5    |
|                                |                     |                     |         |
| Mean peak bankroll per game    | 1315.55             | 1276.88             | 1296.22 |
| Maximum recorded bankroll      | 8320                | 6581                | 7450.5  |
| Profitable games               | 91.31%              | 91.9%               | 91.61%  |
| Profitably ended games         | 0%                  | 0%                  | 0%      |

Due to its negative progression nature, a gambler quickly runs out of their bankroll. It takes only 120 rounds to lose, and it’s 7,8 times faster than betting always on Punto or Banco. These numbers are very close to the martingale system.

### 1-3-2-6

The 1-3-2-6 betting strategy is a positive progression betting system. It is a variant of the reverse martingale system, differing in how bets are increased. The core idea behind this strategy is to capitalize on winning streaks by adjusting bet size in a specific sequence, aiming to maximize profits from these runs while limiting losses.

Explanation of 1-3-2-6 algorithm:

1. Establish a base betting unit. This unit will be the multiplier for your bets.
2. First Bet (1 Unit): Place a 1-unit bet ($10) as your initial bet on either the Player or Banker hand.
3. After a Win (Stage 1): If your first bet wins, your next bet (Stage 2) will be 3 units (e.g., $30).
4. After a Win (Stage 2): If your second bet wins, your next bet (Stage 3) will be 2 units (e.g., $20).
5. After a Win (Stage 3): If your third bet wins, your next bet (Stage 4) will be 6 units (e.g., $60).
6. After a Win (Stage 4), cycle completion: If your fourth bet wins, you have completed one full cycle of the strategy. You should then return to the original 1-unit bet ($10) to restart the sequence.
7. After any Loss: If you lose a hand at any point during the sequence (Stage 1, 2, 3, or 4), you must return to your original 1-unit base bet ($10) for the next hand and restart the progression from Stage 1.

| Strategy                       | 1-3-2-6 on Punto | 1-3-2-6 on Banco | Average |
| ------------------------------ | ---------------- | ---------------- | ------- |
| Mean rounds per game           | 490.8            | 489.5            | 490.15  |
| Minimum played rounds per game | 104              | 107              | 105.5   |
| Maximum played rounds per game | 4070             | 3224             | 3647    |
|                                |                  |                  |         |
| Mean wins per game             | 219.1            | 224.5            | 221.8   |
| Minimum wins per game          | 28               | 29               | 28.5    |
| Maximum wins per game          | 1952             | 1562             | 1757    |
| Win rate                       | 43.8%            | 45.03%           | 44.42%  |
| Rate of zero-wins games        | 0%               | 0%               | 0%      |
|                                |                  |                  |         |
| Mean winning streak            | 6.9              | 7.1              | 7       |
| Maximum winning streak         | 26               | 25               | 25.5    |
| Mean losing streak             | 9.5              | 9.2              | 9.35    |
| Maximum losing streak          | 30               | 28               | 29      |
|                                |                  |                  |         |
| Mean peak bankroll per game    | 1119.65          | 1117.53          | 1118.59 |
| Maximum recorded bankroll      | 3070             | 2922.5           | 2996.25 |
| Profitable games               | 77.1%            | 78.69%           | 77.9%   |
| Profitably ended games         | 0%               | 0%               | 0%      |

There is also a variation called the 1-3-2-4 system, which reduces the risk on the final bet by wagering 4 units instead of 6, helping to preserve some profits even if the last bet loses. Unfortunately, none of these strategies saves a gambler’s bankroll from being out-run. A gambler will run out of it in 490 rounds. It’s even faster than betting only on Égalité.

So, progression strategies are much riskier than flat betting ones. Following negative progressions, a gambler will run out of bankroll about 6 times faster than betting always on Punto or Banco. However, **these strategies may be considered as a «winning» if a gambler can _stop the game himself_ in case of a profitable bankroll.**

## Summary

So, when you run out of money playing punto banco? Betting always on Banco, you will run out of money in 942 rounds (with an initial bankroll of $1000 and a fixed bet of $10). Betting on Punto with the martingale system, you will run out of money in 112 rounds.

![Mean rounds per game session for different baccarat strategies](/assets/2025-09-02/01-mean-rounds-per-game.png)

_Mean rounds per game session for different baccarat strategies_

And a few other ideas after the performed simulations:

- **No strategy bets the house in the long run.**
- **Yes, the win rate of betting on Banco is slightly higher than betting on Punto**, but the 5% commission on payoffs undermines the profit of this slight advantage.
- **Do not bet on Égalité.**
- Progression strategies (Martingale, Paroli, Fibonacci, D’Alembert, and 1-3-2-6 systems) have a lower win rate compared to flat betting ones but allow for winning more money. **However, by following such a strategy, you will lose the game more quickly, and even without winning a single round.**
- If someone tells you about «a secret baccarat strategy» with a win rate of around 50%, it is a lie!

I can imagine how some bloggers claim to have an «unbeatable baccarat strategy» with ridiculous win rate numbers. They take a progression strategy, like the reverse martingale, cherry-pick a winning streak, and present it as «the best baccarat strategy» they discovered. Unfortunately, these cherry-picks don’t work in the long run. Do not be fooled; baccarat is a game of chance where each hand is statistically independent of the previous outcomes.

However, trying to catch a trend of a winning streak is a strategy by itself. This topic is out of the scope of this article, but here are studies on it:

1. Hiroyuki Muto, Ryusuke Nakai, Toshiya Murai, Sakiko Yoshikawa, Nobuhito Abe, “Baccarat gamblers follow trends rather than adhere to the gambler’s fallacy: Analyses of field data from a casino,” Acta Psychologica, Vol. 258, August 2025. [https://doi.org/10.1016/j.actpsy.2025.105172](https://www.sciencedirect.com/science/article/pii/S0001691825004858)
2. Weicheng Zhua, Changsoon Park, “Probabilities of Baccarat by Simulation,” Communications of the Korean Statistical Society, Vol. 19, No. 1, 117–128, 2012. [http://dx.doi.org/10.5351/CKSS.2012.19.1.117](http://dx.doi.org/10.5351/CKSS.2012.19.1.117)

### Threats to Validity

The results of this article are subject to several threats, including the correctness of the simulator implementation, the correctness of the selected statistical categories, and the correctness of interpreting the results.

As for the correctness of the simulator implementation, the way to end the game can be changed to a true «run out of money» when the bankroll is really empty (zero). Otherwise, with the current logic, the player may run out of available bankroll to bet further due to the extra-large next progressive bet, and the game ends with a profit. This flaw was described in the section of the simulator’s logic.

The simulator has no betting limits for the martingale, paroli, Fibonacci, and D’Alembert strategies; however, real casinos often have maximum betting limits in place. For example, €1000 in [Casino de Monte-Carlo](https://www.montecarlosbm.com/en/casino-monaco/games/punto-banco-baccara) in Monaco.

All simulations were made for a 6-deck shoe. However, an 8-deck shoe is also commonly used in casinos. Game sessions with an 8-deck shoe require further simulations.

1М game session simulation (as it was made here) may not be enough for precise results.

The behavior of a bankroll over time during a game session requires further study.

### References

1. [The House Always Wins: Simulating 5,000,000 Games of Baccarat a.k.a. Punto Banco](https://paulvanderlaken.com/2018/01/10/baccarat-simulation-payoff/) by Paul van der Laken
2. [Baccarat Basics — Rules, Strategy, and Tips for Beginners](https://wizardofodds.com/games/baccarat/basics/), Wizard of Odds
3. [Baccarat Strategies and Systems for Advanced Players](https://www.baccarat.net/baccarat-specials/advanced-guide-of-how-to-play-baccarat-and-strategies/) by Caroline Richardson
4. [The Best Baccarat Strategy Guide — Learn How to Win at Baccarat](https://readwrite.com/gambling/guides/baccarat-strategy/) by Djordje Bogdanovic
5. [How To Beat Baccarat: The Best Baccarat Strategies Revealed](https://www.casino.org/blog/baccarat-strategy/) by Stephen Tabone
6. S.N. Ethier and Jiyeon Lee, “The evolution of the game of baccarat,” 2015. [https://doi.org/10.48550/arXiv.1308.1481](https://doi.org/10.48550/arXiv.1308.1481)

Copy @ [Medium](https://adequatica.medium.com/when-you-run-out-of-money-playing-baccarat-punto-banco-deaa5ca23fb7)
