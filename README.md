# sts2ai-runs

4,960 complete runs of **Slay the Spire 2** (Ironclad, Ascension 0) played by an autonomous agent, recorded action by action.

| | |
|---|---|
| Completed runs | 4,960 |
| Wins | **4,251 (85.7%)** |
| Losses | 709 |
| Runs with a recorded trace | 4,950 of the 4,960 (10 traces were lost) |
| Character / difficulty | Ironclad, Ascension 0, all content unlocked |
| Recorded | October 2026 |

Write-up: [Milestone 1: Solving Slay the Spire 2 at Ascension 0](https://shawnxu0913.github.io/2026/10/07/solving-sts2-ascension-0.html)

The seeds were drawn at random before any run was played. The set includes every completed run, losses as well as wins, not a selection of the best ones.

## Download

The data is attached to the [v1.0 release](https://github.com/shawnxu0913/sts2ai-runs/releases/tag/v1.0) as a single tarball, `trace95_bundle.tar`.

```
manifest.jsonl                    one line per run
traces/<SEED>/agent_actions.json.gz   every action in the run, with the game state after it
traces/<SEED>/run_info.json           seed and character
```

## manifest.jsonl

One JSON object per run:

```json
{"seed": "00NSCBRFJD", "verdict": "WIN", "final_floor": 48, "wall_s": 1653, "has_trace": true}
```

| field | meaning |
|---|---|
| `seed` | the game seed; you can play it yourself from the game's custom-seed option |
| `verdict` | `WIN` or `LOSS`; `INCOMPLETE` for 19 runs stopped by a time limit before finishing (their partial traces are included; they are not counted in the win rate) |
| `final_floor` | the last floor reached: 48, or 50 on the longer map route, for a win |
| `wall_s` | time the agent took to play the run, in seconds |
| `has_trace` | whether `traces/<SEED>/` holds the run's actions |

## agent_actions.json.gz

A gzipped JSON array: one record per decision, in order. A record holds:

- `action`: what the agent did: `play_card`, `end_turn`, `use_potion`, `choose_map_node`, `choose_event_option`, `choose_rest_option`, `buy_card`, `buy_relic`, `buy_potion`, `purge_card`, `open_chest`, `choose_treasure_relic`, `collect_rewards_and_proceed` or `select_deck_card`;
- its parameters, where it has any: `card_index` and `target_index` for a card, `potion_index` and `target_index` for a potion, and `option_index` for a map node, event option, shop item or card choice;
- `_mod_state_after`: the full game state after the action. `run` covers HP, gold, floor, deck, relics and potions. `combat` covers the hand, piles, energy, powers and each enemy's HP, block, powers and intent;
- `_engine_combat_log`: the combat events the action caused (damage dealt and blocked, powers applied, enemy moves), as text lines.

Records with only `action` and `option_index` are choices made inside a card-selection prompt (for example, which card to exhaust) and belong to the action before them.

```python
import gzip, json

actions = json.load(gzip.open("traces/00NSCBRFJD/agent_actions.json.gz"))
for a in actions:
    s = a.get("_mod_state_after", {}).get("run", {})
    print(a["action"], s.get("floor"), s.get("current_hp"))
```

## How the agent works

The agent searches each fight inside an exact simulation of the game and plans the run with learned models. Because the simulation reproduces the game's random number generator, the agent effectively knows what it will draw and what random effects will produce, information a human player only has by save-scumming. See the write-up for details and the planned version that plays with only the information a player can see.

---

Slay the Spire 2 is a game by Mega Crit. This project is not affiliated with Mega Crit.
