# Hero Forge Build Plan

Nov 5, 2026 · @Sompop Charusaengrit

Hero Forge is built in 5 phases: tools and data first, then the battle engine, the game screens, the daily AI forge, and finally polish and the class showcase.

## Tools and accounts

Six tools do the work; GitHub is the single source of truth for code and hero data.

| Tool | Role in Hero Forge | Setup needed |
| --- | --- | --- |
| Gemini app | Runs the two skills and the daily forge task | Google AI Pro (for scheduled actions); create both skills |
| Gemini connectors | Google Workspace (Drive) to save specs and art; GitHub to read the repo | Connect in Connected Apps; check whether GitHub is listed |
| Google Drive | Hero Forge folder for concept, specs, art, prompts, showcase | Folder already created under My Drive > Projects |
| GitHub | Private repo `hero-forge`: game code + `heroes.json` | New repo, `main` branch, tags per phase |
| Claude | Battle engine, game logic, data format, code review | This project (Hero Forge) |
| Google AI Studio | Game screens and UI, quick visual iterations | Connect the app to the GitHub repo |

## Where things live

Drive holds the creative work in progress; GitHub holds what the game actually runs. A hero counts as "in the game" only once it is in `heroes.json` on GitHub.

**Google Drive: My Drive > Projects > Hero Forge**

- `01 Concept`: one-page game concept, roles, battle rules
- `02 Hero Specs`: one doc per hero (`H001 Name.md` style)
- `03 Art`: one portrait per hero, same ID as the spec (`H001.png`)
- `04 Build Prompts`: prompts sent to Claude and AI Studio, saved by phase
- `05 Showcase`: screenshots and notes for class

**GitHub: private repo `hero-forge`**

```
hero-forge/
  README.md            what the game is, how to run it
  docs/                concept + this build plan (copied from Drive)
  data/
    heroes.json        every hero, one entry each
    stages.json        campaign stages and enemy teams
  assets/heroes/       portraits, H001.png ...
  src/
    engine/            battle engine (Claude)
    ui/                screens (AI Studio)
    save/              browser save/load
```

Rules: one hero = one ID used everywhere (spec, art, data); tag a GitHub release at the end of each phase (`v0.1-engine`, `v0.2-screens` ...) so you can always roll back, as you did with travel-agent-ai.

## Pipeline

&#91;embedded content: hero pipeline · 8 steps\]

Gemini handles the top row on its own each evening. The bottom row is your quick check and the build that puts the hero into the game. Claude and AI Studio are only needed when code changes; a new hero alone is just data.

## Hero data format

Every hero is one entry in `heroes.json`, so adding a hero never needs new code. The Designer skill outputs exactly this shape.

```json
{
  "id": "H001",
  "name": "Kaelen Stoneward",
  "role": "tank",
  "element": "earth",
  "rarity": "rare",
  "row": "front",
  "stats": { "hp": 1200, "attack": 60, "defense": 90, "speed": 70 },
  "basic": { "name": "Shield Bash", "type": "damage", "power": 1.0, "target": "front" },
  "ultimate": { "name": "Granite Wall", "type": "shield", "power": 2.5, "target": "all-allies", "energy": 100 },
  "lore": "One sentence of backstory.",
  "art": "assets/heroes/H001.png",
  "artNote": "Stone-grey armor, round shield, calm face, blue accent"
}
```

The example hero is a placeholder showing the shape, not a final design.

**Field rules:**

- `role`: tank, healer, damage, support
- `row`: front or back
- `type`: damage, heal, shield, buff, debuff
- `target`: front, back, lowest-hp, all-enemies, self, all-allies
- Stat budget per rarity keeps heroes balanced: total stat points fixed per rarity (set in Phase 1)

## Skills

Two Gemini skills keep every forged hero balanced and on-style. Paste each block as the skill's instructions.

**Skill 1: Hero Forge Designer**

```
You design heroes for Hero Forge, a team auto-battler.

Always:
- Read the existing heroes first and never repeat a name, look or skill idea.
- Create fully original heroes. Never use names, skills or looks from existing games, anime, films or books.
- Fill a gap: prefer the role and element with the fewest heroes.
- Stay inside the stat budget for the chosen rarity.
- Give exactly 1 basic attack and 1 ultimate, using only the allowed types and targets.

Output:
1. The hero as JSON in the Hero Forge data format.
2. Three short lines: why this hero fits the roster, its best teammates, its counter.
```

**Skill 2: Hero Forge Art Director**

```
You create the portrait for each new Hero Forge hero.

Style (never change):
- Bold cel-shaded 2.5D portrait, shoulders up, facing slightly left
- Plain soft-gradient background in the role color:
  tank = blue, healer = green, damage = red, support = purple
- Clean outlines, readable at small card size, no text in the image

Rules:
- Original characters only; no resemblance to existing characters.
- Match the hero's artNote, element and role.
- Square image.

Output: the image, then a one-line art note (pose, colors, weapon).
```

## Daily forge task

One scheduled action in Gemini produces one new hero a day. Set it for a time outside your work hours, for example 7 PM.

```
Every day at 7 PM:
Using the Hero Forge Designer skill, read the hero specs in my Drive folder
"Hero Forge / 02 Hero Specs" and forge one new hero that fills
the biggest gap in the roster. Save the spec as a new doc in that folder,
named with the next ID (for example "H013 Name").
Then, using the Hero Forge Art Director skill, create the hero's portrait.
Finish with a short summary: ID, name, role, and why it was chosen.
```

Human steps each day (about 5 minutes):

1. Read the new spec; reject or tweak it if it is weak.
2. Save the portrait into `03 Art` with the same ID (Gemini may not save images to Drive itself).
3. Add the JSON to `data/heroes.json` and the image to `assets/heroes/` on GitHub.

## Phases

&#91;embedded content: build roadmap · 5 phases\]

Version 1 is Phases 1–3 this weekend, with both skills and the daily task set up too, so the forge starts running Monday, Nov 9. Each phase ends with a GitHub release tag, so a broken step never costs the work before it.

## Phase 1 build prompts

Phase 1 ends with a battle that runs in the browser console from `heroes.json`, before any screens exist. Start with Claude for the engine, then hand the repo to AI Studio.

**Prompt 1 (Claude): starter roster and rules**

```
For Hero Forge, a team auto-battler set in a brand new original world:
0. First propose 3 original world themes (name, setting, tone, 4 elements).
   Wait for me to pick one before continuing.
1. data/heroes.json with 12 original starter heroes (3 per role:
   tank, healer, damage, support) in the Hero Forge data format.
2. A stat budget per rarity (common, rare, epic) so heroes stay balanced.
3. data/stages.json with Act 1, Chapter 1: 10 stages of enemy teams
   built from the same heroes, getting harder each stage.
```

**Prompt 2 (Claude): battle engine**

```
Write src/engine/battle.js for Hero Forge as plain JavaScript with no UI.
Rules: two teams of up to 5; front row is targeted first; turn order by
speed; each hero uses its basic attack and gains energy; at 100 energy it
uses its ultimate; damage = attack x power x (100 / (100 + defense)).
Run until one team is defeated or 30 rounds pass (draw).
Return a battle log (one line per action) plus the winner.
Include a small test that runs stage 1 with a sample team and prints the log.
```

**Prompt 3 (AI Studio): first screen**

```
This repo is Hero Forge, a team auto-battler. Do not change src/engine.
Build a simple web page that loads data/heroes.json, lets me pick 5 heroes
into front and back rows, runs a battle against stage 1 using
src/engine/battle.js, and shows the battle log line by line with HP bars.
Use code-drawn hero cards (name, role color, stats); no images yet.
```

## Decisions

All five open questions are settled; the only risk left is the tight weekend for version 1.

- **Dates:** version 1 (Phases 1–3, plus both skills and the daily task set up) done by Sunday, Nov 8. Final project due Dec 10, 2026.
- **GitHub connector:** available in the Gemini app; connect it, and the daily task can read `data/heroes.json` directly instead of only the Drive specs.
- **Scheduled actions:** expected to work on your Gemini Pro plan; confirm by creating the daily task on Saturday.
- **Account:** your main Google account (it has Gemini Pro).
- **Theme:** brand new world. Prompt 1 asks Claude to propose 3 original world themes first; pick one before the 12 starter heroes are made.

If the weekend runs short, Phase 3 screens can slip to Monday; the engine and the forge setup matter more for the class showcase.
