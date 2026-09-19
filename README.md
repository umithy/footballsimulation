## Football Simulator — a single-file tactical match simulator

A self-contained HTML file that simulates full 90-minute football matches on a 9-zone tactical pitch. No server, no dependencies, no installation — open it in any modern browser and play. Teams are generated with realistic identities (attack-minded, defence-minded, or balanced), matches unfold through a dice-driven zone engine, and every tunable constant is exposed in a live editor.

**Live demo:** [your GitHub Pages URL]
**Source:** [your GitHub repo URL]

### Overview

The pitch is split into a 3×3 grid: three **depths** (own third, midfield, final third) × three **channels** (left wing, centre, right wing). Every minute of match time, a duel is resolved in the current square using weighted attribute averages of the players occupying that square (and its two adjacent support squares), a random dice roll, and — if enabled — a home-advantage bonus. The outcome of the duel determines whether the ball advances, switches channel, is lost, goes out for a throw-in, earns a corner, a free kick, a penalty, or ends up in the back of the net.

A 90 + 3–8 minute match at default settings typically produces **2.5–3.5 total goals** with the weaker side scoring in roughly a third of fixtures. Blowouts are rare by design: a quality gap between teams is capped before it enters the duel equation, so even a mismatched pair stays competitive for most of the match.

### Features

- **Single-file HTML** — vanilla JavaScript + CSS, no build step, no dependencies.
- **Tactical 9-zone engine** with depth-aware attribute weights for both attackers and defenders.
- **Support-square pooling** — players in adjacent squares contribute at reduced strength, so formations matter.
- **Team identities** — squads are tagged as *balanced*, *attack-strong* or *defence-strong*; tilts are asymmetric so no team is strong everywhere.
- **Goalkeeper, corners, penalties, free kicks, throw-ins, goal kicks, counters** — all resolved as staged two-minute events, never skipped.
- **Live event log** with coloured side-borders (goals, shots, corners, fouls, counters).
- **Zone heat map** with ratio-based colour scale showing possession distribution per team.
- **Full statistics panel** — goals, shots, on/off target, saves, corners, penalties, fouls, turnovers, counters, possession %, and per-zone minute totals.
- **Hide-field mode** — toggle the pitch off to focus on statistics without stopping the match.
- **Speed control** — x1 (4 s/min), x2 (2 s/min), x4 (1 s/min), x8 (0.5 s/min).
- **Editable squads** — change any of the 6 outfield attributes (pace, shooting, passing, dribbling, defending, physic) before the next match.
- **Five formations** — 4-4-2, 4-3-3, 4-2-3-1, 5-3-2, 3-5-2, 3-4-2-1, with automatic positional labels (LCB/CB/RCB, LCM/CM/RCM, LOM/CF/ROM, etc.).
- **Home-advantage toggle** — a flat per-roll bonus for the home side, switchable before kick-off.
- **24 tunable constants** in an Advanced panel — every threshold, weight and cap is adjustable and committed on match start.
- **Reset-defaults button** — one click returns all constants to factory values.
- **Staged commit model** — squad edits and constant changes apply only at the next match start, never mid-game.

### How to play

1. Open the `.html` file in any modern browser (Chrome, Firefox, Safari, Edge).
2. Two random squads load automatically. Edit the team name, change formation, tweak player ratings, or hit **New Team** to regenerate.
3. Press **Start**. Pick a speed with **Speed x1/x2/x4/x8**.
4. Watch the ball move square by square. The active square is outlined in white. Click **Hide field** at any time to collapse the pitch and focus on the log + statistics — the match keeps running.
5. Toggle **Home Team advantage** on or off before kick-off to simulate neutral-venue matches.
6. After full-time, press **Start** again for a fresh match with the same (or edited) squads, or **New Team** on either side to regenerate that team only. **Cancel** aborts the current match and resets the scoreboard.

### How the engine works

#### 1. Squares and ownership

Every square has an absolute coordinate `{col: 1–3, row: L/C/R}` in **home-team perspective**: column 1 is the home goal line, column 3 is the away goal line. The team in possession "looks" at the pitch from its own attacking direction; the away side's view is mirrored automatically.

#### 2. Per-square value (the pool rule)

For any square `k` and role (attack `A` or defence `D`), the value is:

```
own  = weighted mean of players IN k, using the role's depth-specific weights
sup  = weighted mean of players in k's two SUPPORT squares, minus a support penalty (default 6)
base = (own · nOwn + sup · nSup) / (nOwn + nSup)        [fallbacks when one side is empty]
bonus = min((nOwn−1) · bonusPerPlayer + nSup · supportBonus, bonusCap)
value = base + bonus
```

For defence in the defending team's `(1,C)` (the goalmouth), the result is blended with the goalkeeper's rating: `v = (1 − gkBoxWeight) · v + gkBoxWeight · GK`.

The weights depend on **depth** — attackers in the final third lean heavily on shooting and pace; defenders in their own third lean heavily on defending and physic; midfield duels are balanced around passing and dribbling. The support relation is strictly **lateral** within each depth: `(1,C)` is supported by `(1,L)` and `(1,R)`; `(2,C)` by `(2,L)` and `(2,R)`; and so on. A team that leaves a whole channel empty pays a real cost.

#### 3. The duel

```
D = (atk + d25) − (def + d25) ± homeAdv
```

The quality gap `atk − def` is **compressed and capped** (`gapScale 0.6`, `gapCap 3` by default) before entering the formula, so a huge squad-rating gap cannot make every duel deterministic. The d25 rolls inject variance; the home-advantage bonus (default +3) is added to the home side's roll regardless of which team has the ball.

#### 4. Outcome bands

| D | Result |
|---|---|
| ≥ +10 | **Breakthrough**: ball advances one square; in the opponent's `(3,C)` this is a goal, in `(3,L/R)` a corner |
| +5 … +9 | **Channel switch**: lateral move (L↔C↔R) |
| +2 … +4 | **Foul**: staged free kick (one square forward); in `(3,C)` has a 40 % chance to become a penalty |
| −2 … +1 | **Throw-in** (L/R squares only) or **scramble / turnover** (centre) |
| −3 … −10 | **Shot on target** if in `(3,C)` (saved by keeper), otherwise **turnover** |
| −11 … −10 | **Shot off target** if in `(3,C)` (goal kick), otherwise **turnover** |
| ≤ −13 | **Counter**: possession flips and the new attacker jumps one square forward (capped at depth 3) |

#### 5. Set pieces are real events

Corners, free kicks, penalties and throw-ins each take **their own minute**. The ball is parked on its staging square (corner flag, free-kick spot, centre circle, touchline) and visibly travels to its delivery point one tick later. A corner that is *won* on minute N is *taken* on minute N+1 and *resolved* on minute N+2 — no event is ever skipped.

### Parameter reference

Every value below is editable live in the **Advanced Calculation Constants** panel at the bottom of the page and is committed at the next match start.

| Parameter | Default | What it does | Raising it | Lowering it |
|---|---:|---|---|---|
| `Dice sides` | 25 | Variance of every roll | More upsets, more goals | More deterministic, fewer goals |
| `Break threshold` | +10 | D needed to advance a square | More goals | Fewer goals, more lateral play |
| `Side-switch threshold` | +5 | D needed to change channel | Lots of channel switching | Straight vertical attacks |
| `Foul threshold` | +2 | D needed to win a free kick | More set pieces | Fewer set pieces |
| `Out-of-play floor` | −2 | D at which the ball goes out | More throw-ins | Fewer throw-ins |
| `Turnover floor` | −10 | D at which possession flips | Lots of counters | Possession is sticky |
| `Counter threshold` | −13 | (reserved, informational) | — | — |
| `Shot ON band min / max` | −6 / −3 | D window for shots on target | More saves, more goals | Fewer saves, more misses |
| `Shot OFF band min / max` | −10 / −7 | D window for shots off target | More goal kicks | Fewer goal kicks |
| `GK box weight` | 0.35 | How much the keeper counts in `(1,C)` defence | Fewer goals conceded | Goals flow more freely |
| `GK corner weight` | 0.30 | Same, during corner defence | Fewer corner goals | More corner goals |
| `Bonus per extra own player` | 5 | Crowd bonus per additional player in the ball's square | Formations that pack a square dominate | Solo duels matter more |
| `Bonus per support-square player` | 3 | Bonus per player in an adjacent square | Full-width formations rewarded | Narrow formations viable |
| `Total bonus cap` | 15 | Hard ceiling on crowd bonus | Bigger crowds matter more | Skill dominates over numbers |
| `Penalty goal threshold` | +10 | Shooter-vs-keeper D needed to convert | Lower conversion rate | Higher conversion rate |
| `Corner goal threshold` | +14 | D needed to score from a corner | Corners rarely score | Corners score often |
| `Box foul → penalty chance` | 0.40 | Probability a `(3,C)` foul becomes a penalty | More penalties | Fewer penalties |
| `Support / empty-zone penalty` | 6 | Points subtracted from support-square contributions | Support squares matter less | Wing play is punished harder |
| `Gap scale` | 0.60 | How much of the squad gap enters the duel | Mismatches blow out | Close games even with mismatched squads |
| `Gap cap` | 3 | Maximum gap contribution (points) | Same as above, harder ceiling | Same as above, softer ceiling |
| `Home advantage` | 3 | Flat per-roll bonus for the home side | Stronger home effect | Neutral-venue feel (set to 0) |

### Calibration tips

- **0–0 finishes too often** → lower `Break threshold` by 1, or raise `Dice sides` to 27.
- **Too many 4+ goal blowouts** → lower `Gap cap` to 2 or `Gap scale` to 0.5.
- **Weak side never scores** → lower `Gap cap`, raise `Dice sides`, and make sure `Home advantage` is off when testing symmetry.
- **Too many corners scoring** → raise `Corner goal threshold` to 16.
- **Wing play feels free** → raise `Support / empty-zone penalty` to 8–10.
- **Goalkeepers are too dominant / too weak** → adjust `GK box weight` in 0.05 steps.

A good calibration target on 100 simulated matches at Speed x8:
- Average total goals: **2.4 – 3.2**
- 0–0 rate: **< 12 %**
- 4+ goal margin rate: **< 10 %**
- Underdog scores in: **20 – 35 %** of fixtures

### Tips & tricks

- Turn on **Compact log** to filter the log down to just goals, shots, corners and penalties.
- Turn on **Zone heat map** after a few minutes to instantly see which channels a team is using — great for diagnosing why a side is struggling.
- The **Hide field** toggle is your analysis mode: hide the pitch, watch the statistics build up, then re-enable the field to see how the match is playing out.
- Edit squads **before** pressing Start. Changes mid-match are ignored by design — they take effect on the next kick-off.
- Press **New Team** on one side to regenerate just that squad; the other side is preserved for a rematch.

### Roadmap (contributions welcome)

The engine is intentionally lean in v1. Planned additions:

- Assist chains, own goals, and per-match xG timeline
- Cards, injuries, and fatigue (stamina drain per minute)
- League / season mode with fixtures, tables and promotion / relegation
- Post-match report export (JSON, PDF)
- Automatic calibration tool: run N matches and suggest optimal constants
- Additional formations (4-5-1, 3-4-3, 4-1-4-1, …)
- Internationalisation (TR, ES, DE, FR, PT in addition to EN)
- Light theme and customisable team colours
- AI-vs-AI "auto league" viewer

If any of these sound interesting, open an issue or a pull request — see **Contributing** below.

### License & contributing

MIT — use it, fork it, embed it, ship it, modify it. Pull requests are welcome for any of the roadmap items above or for bug fixes. Please open an issue first for large redesigns so we can align on scope.
