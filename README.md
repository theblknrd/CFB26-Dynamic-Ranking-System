# 🏈 College Football 26 — Dynamic Rankings Tracker

A spreadsheet-based ranking engine for *College Football 26*.  
Tracks head-to-head results, applies a weighted scoring formula, and generates dynamic rankings with advanced stats like **Strength of Schedule**, **Expected Wins**, and an **Upset Index**.

---

## 📂 Sheets Overview

<details>
<summary><b>📜 Dynamic Ranking Tracker</b> (Game Log + Live Calc)</summary>

- **Columns:**  
  `Game #`, `Home Team`, `Visitor`, `Winner`, `Home Team Rank`, `Visitor Rank`,  
  `Home Team Accumulative Score`, `Visitor Accumulative Score`, `Ranking`, `Team`,  
  `Accumumalitve Score`, `Games Played`, `Team Record`  

- **Logic:**  
  - Wins = **+1 point** (boosted if Away).  
  - Losses = **−0.25**, scaled by opponent rank.  
  - Example loss formula:  
    ```excel
    =-0.25 * (1 + 0.06 * (10 - MIN([@OpponentRank],10)) / 10)
    ```
    → Loss vs #4 ≈ −0.27, loss vs #5 ≈ −0.25.  
  - Accumulative scores roll into team totals dynamically.  

</details>

---

<details>
<summary><b>📊 Ranking</b> (Current Standings)</summary>

- **Columns:**  
  `Ranking`, `Team`, `Accumumalitve Score`, `Games Played`, `Team Record`

- **Logic:**  
  ```excel
  =SORT(DynamicRankingTracker!K:M, 2, FALSE)
  ```
  Sorts by cumulative score → updates ranks automatically.

</details>

---

<details>
<summary><b>📈 Team Totals</b> (Aggregates & Advanced Stats)</summary>

- **Columns:**  
  `Participation %`, `Games Played`, `Wins`, `Losses`, `Total Score`,  
  `Win %`, `Strength of Schedule`, `Expected Wins`,  
  `Delta (Actual-Expected)`, `Upset Index`  

- **Key Formulas:**  
  - Win %:  
    ```excel
    =Wins / GamesPlayed
    ```  
  - Strength of Schedule:  
    ```excel
    =AVERAGEIFS(OpponentRankRange, OpponentRankRange, "<=25")
    ```  
  - Expected Wins (logistic model):  
    ```excel
    =1/(1+EXP(-(TeamRating - OppRating + IF(Home,35,0))/350))
    ```  
  - Delta:  
    ```excel
    =ActualWins - ExpectedWins
    ```  
  - Upset Index:  
    ```excel
    =SUMIFS( IF(Result="Win", 1-ExpWinProb, -ExpWinProb) )
    ```

</details>

---

<details>
<summary><b>🏅 Seed Ranks</b> (Baseline Reference)</summary>

- **Columns:**  
  `Team`, `Rank`  

- Used as lookup for game calculations in **Dynamic Ranking Tracker**.  
- Example:  
  ```excel
  =XLOOKUP([@Team], SeedRanks!A:A, SeedRanks!B:B)
  ```

</details>

---

<details>
<summary><b>🤝 H2H Matrix</b> (Head-to-Head Tracker)</summary>

- **Rows:** Teams  
- **Columns:** Opponents  
- **Cells:** Games won vs opponent.  

- Example formula:  
  ```excel
  =COUNTIFS(Games[Winner], "TeamA", Games[Loser], "TeamB")
  ```

</details>

---

## ⚙️ Scoring Framework

| Scenario                  | Formula Logic                                                   |
|----------------------------|----------------------------------------------------------------|
| **Win**                    | `=1 * (IF(Away,1.1,1)) + OpponentQualityWeight`               |
| **Loss (base)**            | `=-0.25`                                                      |
| **Loss (rank-adjusted)**   | `=-0.25 * (1 + α * (10 - MIN(OppRank,10))/10)`                 |
| **Farm-prevention**        | `=MAX(1 - 0.2*RepeatWithinWindow, 0.5)`                        |
| **Strength of Schedule**   | `=AVERAGE(OpponentRatings)`                                   |
| **Expected Wins**          | `=SUM(ExpectedWinProbabilityPerGame)`                         |
| **Upset Index**            | `=SUM( IF(Won, 1-ExpProb, -ExpProb) )`                        |

---

## 🚀 How to Use

1. Log games in **Dynamic Ranking Tracker**.  
2. Check **Ranking** tab for live standings.  
3. Analyze team-level metrics in **Team Totals**.  
4. Use **Seed Ranks** as lookup for recalibration.  
5. Break ties with **H2H Matrix**.  

---

## 🧩 Customization

- **Away boost** → adjust `1.1` multiplier.  
- **Loss scaling** → tweak α in rank-adjust formula.  
- **Farm-prevention decay** → change `0.2` step or 0.5 floor.  
- **SoS weights** → bucket opponents differently (Top 5 vs Top 25).  

---

## ✅ Example Insight

- Team A (19–8) outranks Team B (9–20) because accumulation scoring + SoS outpaces raw win count.  
- Losses vs #1 or #4 sting a little more than losses vs #5.  
- Delta shows overachievers/underachievers compared to expectations.  

---

## 📜 License
MIT (or your preference).  
