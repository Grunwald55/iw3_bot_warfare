# IW3 Bot Warfare — repository review and file inventory

Reviewed on 2026-11-12 at commit `901895a10dee921142cf4228c467f99013b5b6c4`. The source identifies itself as **version 2.3.0**; the README describes that version as unreleased.

## Contents

- [What the mod does](#what-the-mod-does)
- [Inventory scope and totals](#inventory-scope-and-totals)
- [Source scripts](#source-scripts--14-files)
- [Documentation](#documentation--3-files)
- [Repository, editor and CI configuration](#repository-editor-and-ci-configuration--6-files)
- [Artwork and tutorial images](#artwork-and-tutorial-images--9-files)
- [Bot names](#bot-names--1-file)
- [Map navigation data](#map-navigation-data--360-files)
- [Review notes and validation limits](#review-notes-and-validation-limits)

## What the mod does

**IW3 Bot Warfare adds AI-controlled multiplayer players to Call of Duty 4: Modern Warfare through CoD4x.** It is primarily written in GSC, the game's scripting language. The bots join teams, choose classes, navigate maps, fight, and participate in the standard multiplayer objectives, providing a combat-training experience on a local or dedicated server.

- **Combat:** Bots acquire enemies using visibility, field of view, smoke checks, and configurable reaction and memory times. They aim, fire, aim down sights, reload, switch weapons, melee, use grenades and launchers, deploy claymores/C4, and throw back grenades. Behavior includes quickscoping, jump/drop shots, and reload cancellation.
- **Movement:** Bots follow waypoint graphs using A* pathfinding, with a cost adjustment to spread teammates across routes. Movement includes sprinting, stance changes, ladders/mantling, breaking obstructing glass, and attempts to recover when stuck. CoD4x supplies the native movement and button actions.
- **Objectives:** Dedicated logic captures and defends Domination flags, captures/destroys Headquarters radios, handles Sabotage bombs and their carriers, and separates Search and Destroy attackers and defenders for planting, guarding, and defusing. General combat supports Deathmatch and Team Deathmatch.
- **Awareness and tactics:** Bots investigate footsteps, gunfire and UAV information, react to damage, alert nearby bot teammates, remember their killer's location, follow players, and camp. They can target helicopters and enemy equipment and call in UAV, airstrike, and helicopter rewards.
- **Individual identities and loadouts:** Each bot gets skill settings, behavioral tendencies, rank/prestige, and generated classes with weapons, attachments, perks, grenades, and camouflage. Persistent player fields retain bot configuration across round restarts. A text file supplies bot names.
- **Server administration:** Console variables (DVARs) control adding/removing bots, population targets, spectator counting, team balancing, difficulty, loadouts, and individual capabilities. Authorized hosts also receive an in-game menu: press primary and secondary grenade together to open it, use ADS/fire to navigate, and melee to select.
- **Chat:** An optional scripted chatter system reacts to kills, deaths, match transitions, objectives, combat actions, and player connections. `bots_main_chat` scales its frequency; `0` disables it.
- **Map authoring:** The built-in waypoint editor displays nodes and links, creates/removes/reloads points, links/unlinks them, supports automatic linking and special action points, checks for common graph problems, and writes CSV files plus diagnostic dumps.

### How the pieces fit together

The CoD4x callback bootstrap in [maps/mp/gametypes/_callbacksetup.gsx](../maps/mp/gametypes/_callbacksetup.gsx#L11) starts the engine adapter and the chat, menu, editor, and bot entry points. The adapter registers engine-specific operations in `level.bot_builtins`. The core manager loads navigation data, configures DVAR defaults, wraps damage/death callbacks, watches players and game events, and maintains bot population, teams, and difficulty.

Each bot then runs two cooperating layers: `_bot_internal.gsc` handles immediate sensing, aiming, movement and input, while `_bot_script.gsc` chooses classes, tactics, equipment use, rewards, and objectives. `_bot_utility.gsc` provides the shared API, navigation loader, data structures, and A* implementation. Chat consumes the bot event stream; the menu and editor provide human-facing controls.

### Defaults and scope

- The mod is enabled by default, but both `bots_manage_add` and `bots_manage_fill` default to `0`; installation alone does not request a populated match.
- `bots_skill=0` gives bots varied difficulty; fixed levels span `1`–`7`, with additional custom/randomized modes. Most combat and movement capabilities default to enabled.
- `bots_main_firstIsHost=1` gives the first human player administration access. The repository explicitly recommends setting it to `0` on public dedicated servers and configuring authorized GUIDs.
- The expected runtime is an installed CoD4x server/client with the base game's scripts, tables, weapons, and maps. The README's installation layout uses `mods/mp_bots` and a release archive containing `z_svr_bots.iwd`. This checkout contains source/data, with no tracked IWD archive or release packaging script.
- The 360 map CSVs provide navigation data; the corresponding custom map packages must be installed separately. Their presence establishes supplied navigation coverage, not verified gameplay quality on every map.

## Inventory scope and totals

This inventory covers **all 393 tracked repository files**, including hidden configuration files. The working tree was clean at inspection, with no additional untracked project files. Git's internal `.git/` database, hooks and logs are repository metadata and are excluded. This generated report was added after the inventory was completed and is not part of the original 393-file count.

| Category | Files | Role |
| --- | ---: | --- |
| GSC/GSX source | 14 | Bot behavior, engine integration, UI and editing |
| Waypoint CSV data | 360 | Map-specific navigation graphs |
| Bot name data | 1 | Configurable bot display names |
| Markdown documentation | 3 | Usage, installation and editor instructions |
| Image/artwork assets | 9 | Logo source and documentation screenshots |
| Repository/editor/CI configuration | 6 | Formatting, file handling and syntax validation |
| **Total** | **393** | |

### Source scripts — 14 files

| File | What it does |
| --- | --- |
| [maps/mp/bots/_bot.gsc](../maps/mp/bots/_bot.gsc) | Core manager and initialization (1,335 lines): establishes version/DVAR defaults; loads waypoints and create-a-class indexes; wraps player damage/death callbacks; initializes bots; manages population, teams and difficulty; tracks smoke, UAVs, helicopters, firing, chat and turret use; adapts Headquarters/Search and Destroy objects. |
| [maps/mp/bots/_bot_chat.gsc](../maps/mp/bots/_bot_chat.gsc) | Scripted bot chatter (2,301 lines): listens for bot/game events and selects random public/team chat and quick messages for deaths, kills, streaks, objectives, tactics, connections and match transitions. Frequency is controlled by bots_main_chat. |
| [maps/mp/bots/_bot_internal.gsc](../maps/mp/bots/_bot_internal.gsc) | Immediate bot control (2,871 lines): maintains runtime state, detects/selects targets, applies aim/reaction behavior, controls shooting and ADS, manages reload/sprint/stance/grenade responses, follows A* paths, climbs/mantles, handles obstacles and stuck recovery, and issues native input actions. |
| [maps/mp/bots/_bot_script.gsc](../maps/mp/bots/_bot_script.gsc) | Higher-level bot decisions (5,034 lines): assigns rank/prestige, difficulty and randomized classes; joins teams; reacts to damage/deaths; chooses camping/following/awareness actions; uses weapons, explosives and killstreaks; attacks helicopters/equipment; and implements Domination, Headquarters, Sabotage, and Search and Destroy tactics. |
| [maps/mp/bots/_bot_utility.gsc](../maps/mp/bots/_bot_utility.gsc) | Shared bot API and algorithms (3,242 lines): wraps engine built-ins; checks host authorization; exposes goal/aim/enemy/input helpers; loads bot names and CSV/Frontlines waypoints; indexes loadout data; provides geometry, arrays, lists, heaps and KD-tree helpers; implements congestion-aware A*; and supplies Search and Destroy planting fixes. |
| [maps/mp/bots/_menu.gsc](../maps/mp/bots/_menu.gsc) | Host-only in-game administration menu (1,369 lines): draws and cleans up HUD elements, handles grenade/ADS/fire/melee navigation, and changes bot counts, fill rules, teams, difficulty, loadout options and behavior toggles. |
| [maps/mp/bots/_wp_editor.gsc](../maps/mp/bots/_wp_editor.gsc) | In-game waypoint editor (740 lines): enters developer editing mode, draws node/link/type/angle overlays, handles add/delete/link/unlink/reload/autolink controls, validates common waypoint problems, and saves CSV plus log/console dumps. Editing mode changes bot/menu/population DVARs and may restart the map to enable developer mode. |
| [maps/mp/bots/waypoints/_custom_map.gsc](../maps/mp/bots/waypoints/_custom_map.gsc) | Extension placeholder: main(mapname) is empty, so it currently supplies no fallback navigation. Its separate helper prints an encoded author-credit message, called from a periodic name check in the bot code. |
| [maps/mp/gametypes/_callbacksetup.gsx](../maps/mp/gametypes/_callbacksetup.gsx) | CoD4x game callback bootstrap (145 lines): starts the adapter and the four bot, chat, menu and editor subsystems after the gametype starts, forwards connection/damage/death/last-stand callbacks, initializes default callbacks and damage flags, and provides unsupported-gametype abort handling. |
| [scripts/mp/bots.gsc](../scripts/mp/bots.gsc) | Four-line entry wrapper that starts maps/mp/bots/_bot.gsc::init on the level. |
| [scripts/mp/bots_adapter_cod4x.gsc](../scripts/mp/bots_adapter_cod4x.gsc) | CoD4x engine adapter (83 lines): registers console/file operations, bot detection, actions, stop/move-to and player-angle controls in level.bot_builtins. File operations are rooted under scriptdata/. Direct forward/right movement and melee-lunge parameter hooks are empty for this backend. |
| [scripts/mp/bots_chat.gsc](../scripts/mp/bots_chat.gsc) | Four-line entry wrapper that starts maps/mp/bots/_bot_chat.gsc::init. |
| [scripts/mp/bots_menu.gsc](../scripts/mp/bots_menu.gsc) | Four-line entry wrapper that starts maps/mp/bots/_menu.gsc::init. |
| [scripts/mp/bots_wp_editor.gsc](../scripts/mp/bots_wp_editor.gsc) | Four-line entry wrapper that starts maps/mp/bots/_wp_editor.gsc::init. |

### Documentation — 3 files

| File | What it does |
| --- | --- |
| [README.md](../README.md) | Main project guide: explains the CoD4x bot mod, installation, menu controls and DVARs; lists features, changelog and credits; and calls out the first-player administration default. |
| [bw-assets/wpedit.md](../bw-assets/wpedit.md) | Waypoint-authoring tutorial: explains A*, developer/editor setup, controls, node types, linking, automatic linking, CSV saving and output location, illustrated by the bundled screenshots. |
| [scriptdata/waypoints/README.md](../scriptdata/waypoints/README.md) | Waypoint collection overview and contributor credits, with a link to an external PeZBOT GSC-to-CSV conversion tool. |

### Repository, editor and CI configuration — 6 files

| File | What it does |
| --- | --- |
| [.astylerc](../.astylerc) | Artistic Style formatter rules: C/Allman-style braces, forced tabs of width 2, Windows line endings, operator/parenthesis/header spacing, explicit braces, and whitespace/block handling for the scripts. |
| [.editorconfig](../.editorconfig) | Shared editor settings: tab indentation of size 2, Latin-1 encoding, a final newline, and preservation of trailing whitespace, including Markdown. |
| [.gitattributes](../.gitattributes) | Tells GitHub Linguist to classify .gsx files as GSC. |
| [.github/workflows/main.yml](../.github/workflows/main.yml) | GitHub Actions workflow on push: checks out the repository on Windows, installs gsc-tool 1.4.0, and runs gsc-tool.exe -m parse -g iw5 -s pc . as a script parsing check. |
| [.gitignore](../.gitignore) | Excludes ZIP archives and generated server logs, statistics, demos, missing-asset lists, and server configuration/log artifacts from version control. |
| [.vscode/settings.json](../.vscode/settings.json) | VS Code workspace preferences: points Artistic Style to .astylerc, enables GSC/GSX formatting, selects the AStyle formatter, adjusts suggestions/completions, and associates .gsx with GSC. |

### Artwork and tutorial images — 9 files

| File | What it does |
| --- | --- |
| [bw-assets/bw-logo.png](../bw-assets/bw-logo.png) | Project logo displayed at the top of README.md (PNG, 888 × 443). |
| [bw-assets/bwlogo.pdn](../bw-assets/bwlogo.pdn) | Paint.NET source artwork for the project logo; editable design asset. |
| [bw-assets/console-map.png](../bw-assets/console-map.png) | Waypoint tutorial screenshot showing the console command used to start a map for editing (PNG, 540 × 191). |
| [bw-assets/console.png](../bw-assets/console.png) | Waypoint tutorial screenshot showing the debug/editor DVAR being set in the console (PNG, 534 × 477). |
| [bw-assets/editor-addwp.png](../bw-assets/editor-addwp.png) | Waypoint tutorial screenshot illustrating creation of the first waypoint (PNG, 1,014 × 757). |
| [bw-assets/editor-addwp2.png](../bw-assets/editor-addwp2.png) | Waypoint tutorial screenshot illustrating creation of a second waypoint (PNG, 1,017 × 760). |
| [bw-assets/editor-link.png](../bw-assets/editor-link.png) | Waypoint tutorial screenshot illustrating links between navigation points (PNG, 1,019 × 758). |
| [bw-assets/editor.png](../bw-assets/editor.png) | Waypoint tutorial overview of the editor with node numbers, types, links and overlays (PNG, 1,014 × 761). |
| [bw-assets/saved.png](../bw-assets/saved.png) | Waypoint tutorial screenshot showing the saved CSV output location (PNG, 665 × 174). |

### Bot names — 1 file

| File | What it does |
| --- | --- |
| [scriptdata/botnames.txt](../scriptdata/botnames.txt) | Runtime bot-name pool: 26 names, bota through botz. The loader cycles through these names using a cursor DVAR when adding bots. |

### Map navigation data — 360 files

Each file below supplies the bot navigation graph for the exact map identifier shown in its summary. The loader looks for `scriptdata/waypoints/<mapname>_wp.csv`. Each file starts with a declared waypoint count; subsequent rows store `x y z`, space-separated linked node indexes, a waypoint type, optional view angles, and unused trailing fields. Node indexes are zero-based. Links are counted as stored references, so a bidirectional connection usually contributes two references.

Together the CSVs contain **111,185 waypoint records** and **332,404 stored link tokens**. Counts include the malformed tokens noted below. All 360 files' declared counts match their actual waypoint record counts. Graph sizes range from 33 nodes (`cube`) to 808 (`mp_colonel`). Some map variants intentionally share identical graph data; each remains a separate runtime file.

The type column reports literal values found in the data, with counts. `undefined` is a literal legacy value. Type names in a CSV do not by themselves establish a distinct engine behavior for each value. The editor documents stance, climb, grenade, tube, and claymore points; camping points are represented by crouch nodes with one connection. The bundled data contains no `tube` nodes.

| File | What it does | Waypoints | Stored types and counts |
| --- | --- | ---: | --- |
| [scriptdata/waypoints/cube_wp.csv](../scriptdata/waypoints/cube_wp.csv) | Navigation graph for `cube`; supplies positions and traversable links for bot routing. | 33 | undefined: 33 |
| [scriptdata/waypoints/mp_4doffice_wp.csv](../scriptdata/waypoints/mp_4doffice_wp.csv) | Navigation graph for `mp_4doffice`; supplies positions and traversable links for bot routing. | 113 | climb: 12, stand: 101 |
| [scriptdata/waypoints/mp_4hanoi_wp.csv](../scriptdata/waypoints/mp_4hanoi_wp.csv) | Navigation graph for `mp_4hanoi`; supplies positions and traversable links for bot routing. | 143 | stand: 143 |
| [scriptdata/waypoints/mp_4nuketown_wp.csv](../scriptdata/waypoints/mp_4nuketown_wp.csv) | Navigation graph for `mp_4nuketown`; supplies positions and traversable links for bot routing. | 323 | stand: 323 |
| [scriptdata/waypoints/mp_4t4dawnville2_wp.csv](../scriptdata/waypoints/mp_4t4dawnville2_wp.csv) | Navigation graph for `mp_4t4dawnville2`; supplies positions and traversable links for bot routing. | 435 | stand: 435 |
| [scriptdata/waypoints/mp_4t4dawnville_wp.csv](../scriptdata/waypoints/mp_4t4dawnville_wp.csv) | Navigation graph for `mp_4t4dawnville`; supplies positions and traversable links for bot routing. | 309 | stand: 309 |
| [scriptdata/waypoints/mp_4t4scrap_r_wp.csv](../scriptdata/waypoints/mp_4t4scrap_r_wp.csv) | Navigation graph for `mp_4t4scrap_r`; supplies positions and traversable links for bot routing. | 391 | climb: 39, crouch: 19, stand: 333 |
| [scriptdata/waypoints/mp_4t4scrap_s_wp.csv](../scriptdata/waypoints/mp_4t4scrap_s_wp.csv) | Navigation graph for `mp_4t4scrap_s`; supplies positions and traversable links for bot routing. | 391 | climb: 39, crouch: 19, stand: 333 |
| [scriptdata/waypoints/mp_4t4scrap_wp.csv](../scriptdata/waypoints/mp_4t4scrap_wp.csv) | Navigation graph for `mp_4t4scrap`; supplies positions and traversable links for bot routing. | 391 | climb: 39, crouch: 19, stand: 333 |
| [scriptdata/waypoints/mp_78workshop_wp.csv](../scriptdata/waypoints/mp_78workshop_wp.csv) | Navigation graph for `mp_78workshop`; supplies positions and traversable links for bot routing. | 446 | climb: 153, stand: 293 |
| [scriptdata/waypoints/mp_82ab_bazaar_wp.csv](../scriptdata/waypoints/mp_82ab_bazaar_wp.csv) | Navigation graph for `mp_82ab_bazaar`; supplies positions and traversable links for bot routing. | 557 | crouch: 9, stand: 548 |
| [scriptdata/waypoints/mp_82ab_villa_wp.csv](../scriptdata/waypoints/mp_82ab_villa_wp.csv) | Navigation graph for `mp_82ab_villa`; supplies positions and traversable links for bot routing. | 437 | climb: 9, stand: 428 |
| [scriptdata/waypoints/mp_aerodrome_wp.csv](../scriptdata/waypoints/mp_aerodrome_wp.csv) | Navigation graph for `mp_aerodrome`; supplies positions and traversable links for bot routing. | 330 | climb: 2, stand: 328 |
| [scriptdata/waypoints/mp_agx_cz_wp.csv](../scriptdata/waypoints/mp_agx_cz_wp.csv) | Navigation graph for `mp_agx_cz`; supplies positions and traversable links for bot routing. | 686 | climb: 32, stand: 654 |
| [scriptdata/waypoints/mp_agx_nightmare_wp.csv](../scriptdata/waypoints/mp_agx_nightmare_wp.csv) | Navigation graph for `mp_agx_nightmare`; supplies positions and traversable links for bot routing. | 528 | climb: 30, stand: 498 |
| [scriptdata/waypoints/mp_airstrike_v1_wp.csv](../scriptdata/waypoints/mp_airstrike_v1_wp.csv) | Navigation graph for `mp_airstrike_v1`; supplies positions and traversable links for bot routing. | 115 | climb: 4, stand: 111 |
| [scriptdata/waypoints/mp_al_jabal_wp.csv](../scriptdata/waypoints/mp_al_jabal_wp.csv) | Navigation graph for `mp_al_jabal`; supplies positions and traversable links for bot routing. | 272 | climb: 2, crouch: 7, stand: 263 |
| [scriptdata/waypoints/mp_al_jierz_wp.csv](../scriptdata/waypoints/mp_al_jierz_wp.csv) | Navigation graph for `mp_al_jierz`; supplies positions and traversable links for bot routing. | 313 | climb: 14, crouch: 10, stand: 289 |
| [scriptdata/waypoints/mp_anchorage_wp.csv](../scriptdata/waypoints/mp_anchorage_wp.csv) | Navigation graph for `mp_anchorage`; supplies positions and traversable links for bot routing. | 172 | crouch: 2, stand: 170 |
| [scriptdata/waypoints/mp_anchorhead_wp.csv](../scriptdata/waypoints/mp_anchorhead_wp.csv) | Navigation graph for `mp_anchorhead`; supplies positions and traversable links for bot routing. | 178 | climb: 24, stand: 154 |
| [scriptdata/waypoints/mp_ancient_ultimate_wp.csv](../scriptdata/waypoints/mp_ancient_ultimate_wp.csv) | Navigation graph for `mp_ancient_ultimate`; supplies positions and traversable links for bot routing. | 384 | crouch: 8, stand: 376 |
| [scriptdata/waypoints/mp_aosta_valley_wp.csv](../scriptdata/waypoints/mp_aosta_valley_wp.csv) | Navigation graph for `mp_aosta_valley`; supplies positions and traversable links for bot routing. | 507 | climb: 80, crouch: 70, stand: 357 |
| [scriptdata/waypoints/mp_apesgorod_wp.csv](../scriptdata/waypoints/mp_apesgorod_wp.csv) | Navigation graph for `mp_apesgorod`; supplies positions and traversable links for bot routing. | 554 | climb: 28, crouch: 13, stand: 513 |
| [scriptdata/waypoints/mp_arbor_day_wp.csv](../scriptdata/waypoints/mp_arbor_day_wp.csv) | Navigation graph for `mp_arbor_day`; supplies positions and traversable links for bot routing. | 266 | climb: 2, crouch: 11, stand: 253 |
| [scriptdata/waypoints/mp_ardennes_night_wp.csv](../scriptdata/waypoints/mp_ardennes_night_wp.csv) | Navigation graph for `mp_ardennes_night`; supplies positions and traversable links for bot routing. | 438 | climb: 2, stand: 436 |
| [scriptdata/waypoints/mp_ardennes_wp.csv](../scriptdata/waypoints/mp_ardennes_wp.csv) | Navigation graph for `mp_ardennes`; supplies positions and traversable links for bot routing. | 444 | climb: 2, stand: 442 |
| [scriptdata/waypoints/mp_area52_wp.csv](../scriptdata/waypoints/mp_area52_wp.csv) | Navigation graph for `mp_area52`; supplies positions and traversable links for bot routing. | 328 | climb: 9, crouch: 13, stand: 306 |
| [scriptdata/waypoints/mp_argel_wp.csv](../scriptdata/waypoints/mp_argel_wp.csv) | Navigation graph for `mp_argel`; supplies positions and traversable links for bot routing. | 345 | undefined: 345 |
| [scriptdata/waypoints/mp_arkona_wp.csv](../scriptdata/waypoints/mp_arkona_wp.csv) | Navigation graph for `mp_arkona`; supplies positions and traversable links for bot routing. | 256 | stand: 256 |
| [scriptdata/waypoints/mp_armshotel_wp.csv](../scriptdata/waypoints/mp_armshotel_wp.csv) | Navigation graph for `mp_armshotel`; supplies positions and traversable links for bot routing. | 298 | stand: 298 |
| [scriptdata/waypoints/mp_arrange_wp.csv](../scriptdata/waypoints/mp_arrange_wp.csv) | Navigation graph for `mp_arrange`; supplies positions and traversable links for bot routing. | 273 | climb: 6, stand: 267 |
| [scriptdata/waypoints/mp_asylum_wp.csv](../scriptdata/waypoints/mp_asylum_wp.csv) | Navigation graph for `mp_asylum`; supplies positions and traversable links for bot routing. | 235 | stand: 235 |
| [scriptdata/waypoints/mp_athena_wp.csv](../scriptdata/waypoints/mp_athena_wp.csv) | Navigation graph for `mp_athena`; supplies positions and traversable links for bot routing. | 316 | climb: 12, crouch: 58, stand: 246 |
| [scriptdata/waypoints/mp_atp_wp.csv](../scriptdata/waypoints/mp_atp_wp.csv) | Navigation graph for `mp_atp`; supplies positions and traversable links for bot routing. | 214 | stand: 214 |
| [scriptdata/waypoints/mp_ava_asylum_wp.csv](../scriptdata/waypoints/mp_ava_asylum_wp.csv) | Navigation graph for `mp_ava_asylum`; supplies positions and traversable links for bot routing. | 547 | climb: 12, stand: 535 |
| [scriptdata/waypoints/mp_ava_crossroads_wp.csv](../scriptdata/waypoints/mp_ava_crossroads_wp.csv) | Navigation graph for `mp_ava_crossroads`; supplies positions and traversable links for bot routing. | 298 | crouch: 5, stand: 293 |
| [scriptdata/waypoints/mp_ava_tumbleweed_wp.csv](../scriptdata/waypoints/mp_ava_tumbleweed_wp.csv) | Navigation graph for `mp_ava_tumbleweed`; supplies positions and traversable links for bot routing. | 567 | climb: 10, stand: 557 |
| [scriptdata/waypoints/mp_aztec_wp.csv](../scriptdata/waypoints/mp_aztec_wp.csv) | Navigation graph for `mp_aztec`; supplies positions and traversable links for bot routing. | 155 | stand: 155 |
| [scriptdata/waypoints/mp_bacalao_wp.csv](../scriptdata/waypoints/mp_bacalao_wp.csv) | Navigation graph for `mp_bacalao`; supplies positions and traversable links for bot routing. | 205 | stand: 205 |
| [scriptdata/waypoints/mp_backlot_2_wp.csv](../scriptdata/waypoints/mp_backlot_2_wp.csv) | Navigation graph for `mp_backlot_2`; supplies positions and traversable links for bot routing. | 462 | climb: 20, crouch: 6, stand: 436 |
| [scriptdata/waypoints/mp_backlot_lt_wp.csv](../scriptdata/waypoints/mp_backlot_lt_wp.csv) | Navigation graph for `mp_backlot_lt`; supplies positions and traversable links for bot routing. | 216 | climb: 2, stand: 214 |
| [scriptdata/waypoints/mp_backlot_night_wp.csv](../scriptdata/waypoints/mp_backlot_night_wp.csv) | Navigation graph for `mp_backlot_night`; supplies positions and traversable links for bot routing. | 216 | climb: 2, stand: 214 |
| [scriptdata/waypoints/mp_backlot_snow_wp.csv](../scriptdata/waypoints/mp_backlot_snow_wp.csv) | Navigation graph for `mp_backlot_snow`; supplies positions and traversable links for bot routing. | 216 | climb: 2, stand: 214 |
| [scriptdata/waypoints/mp_backlot_wp.csv](../scriptdata/waypoints/mp_backlot_wp.csv) | Navigation graph for `mp_backlot`; supplies positions and traversable links for bot routing. | 178 | climb: 4, stand: 174 |
| [scriptdata/waypoints/mp_bagheera_wp.csv](../scriptdata/waypoints/mp_bagheera_wp.csv) | Navigation graph for `mp_bagheera`; supplies positions and traversable links for bot routing. | 495 | stand: 495 |
| [scriptdata/waypoints/mp_bakaara_wp.csv](../scriptdata/waypoints/mp_bakaara_wp.csv) | Navigation graph for `mp_bakaara`; supplies positions and traversable links for bot routing. | 171 | stand: 171 |
| [scriptdata/waypoints/mp_bastogne_wp.csv](../scriptdata/waypoints/mp_bastogne_wp.csv) | Navigation graph for `mp_bastogne`; supplies positions and traversable links for bot routing. | 233 | climb: 2, stand: 231 |
| [scriptdata/waypoints/mp_bestine_wp.csv](../scriptdata/waypoints/mp_bestine_wp.csv) | Navigation graph for `mp_bestine`; supplies positions and traversable links for bot routing. | 122 | climb: 6, stand: 116 |
| [scriptdata/waypoints/mp_betatnl_wp.csv](../scriptdata/waypoints/mp_betatnl_wp.csv) | Navigation graph for `mp_betatnl`; supplies positions and traversable links for bot routing. | 181 | climb: 8, stand: 173 |
| [scriptdata/waypoints/mp_bjwifi_aim_wp.csv](../scriptdata/waypoints/mp_bjwifi_aim_wp.csv) | Navigation graph for `mp_bjwifi_aim`; supplies positions and traversable links for bot routing. | 189 | climb: 33, stand: 156 |
| [scriptdata/waypoints/mp_bjwifi_port_wp.csv](../scriptdata/waypoints/mp_bjwifi_port_wp.csv) | Navigation graph for `mp_bjwifi_port`; supplies positions and traversable links for bot routing. | 500 | climb: 42, crouch: 13, stand: 445 |
| [scriptdata/waypoints/mp_bloc_wp.csv](../scriptdata/waypoints/mp_bloc_wp.csv) | Navigation graph for `mp_bloc`; supplies positions and traversable links for bot routing. | 260 | stand: 260 |
| [scriptdata/waypoints/mp_blutstadt_wp.csv](../scriptdata/waypoints/mp_blutstadt_wp.csv) | Navigation graph for `mp_blutstadt`; supplies positions and traversable links for bot routing. | 500 | stand: 500 |
| [scriptdata/waypoints/mp_bo2_town_wp.csv](../scriptdata/waypoints/mp_bo2_town_wp.csv) | Navigation graph for `mp_bo2_town`; supplies positions and traversable links for bot routing. | 100 | climb: 2, stand: 98 |
| [scriptdata/waypoints/mp_bo2cargo_wp.csv](../scriptdata/waypoints/mp_bo2cargo_wp.csv) | Navigation graph for `mp_bo2cargo`; supplies positions and traversable links for bot routing. | 455 | climb: 45, stand: 410 |
| [scriptdata/waypoints/mp_bo2cargom_wp.csv](../scriptdata/waypoints/mp_bo2cargom_wp.csv) | Navigation graph for `mp_bo2cargom`; supplies positions and traversable links for bot routing. | 568 | climb: 36, stand: 532 |
| [scriptdata/waypoints/mp_bo2carrier_wp.csv](../scriptdata/waypoints/mp_bo2carrier_wp.csv) | Navigation graph for `mp_bo2carrier`; supplies positions and traversable links for bot routing. | 299 | climb: 4, stand: 295 |
| [scriptdata/waypoints/mp_bo2cove_wp.csv](../scriptdata/waypoints/mp_bo2cove_wp.csv) | Navigation graph for `mp_bo2cove`; supplies positions and traversable links for bot routing. | 214 | climb: 4, stand: 210 |
| [scriptdata/waypoints/mp_bo2drone_wp.csv](../scriptdata/waypoints/mp_bo2drone_wp.csv) | Navigation graph for `mp_bo2drone`; supplies positions and traversable links for bot routing. | 382 | climb: 7, stand: 375 |
| [scriptdata/waypoints/mp_bo2frost_wp.csv](../scriptdata/waypoints/mp_bo2frost_wp.csv) | Navigation graph for `mp_bo2frost`; supplies positions and traversable links for bot routing. | 243 | stand: 243 |
| [scriptdata/waypoints/mp_bo2grind_wp.csv](../scriptdata/waypoints/mp_bo2grind_wp.csv) | Navigation graph for `mp_bo2grind`; supplies positions and traversable links for bot routing. | 221 | climb: 2, stand: 219 |
| [scriptdata/waypoints/mp_bo2hydro2_wp.csv](../scriptdata/waypoints/mp_bo2hydro2_wp.csv) | Navigation graph for `mp_bo2hydro2`; supplies positions and traversable links for bot routing. | 321 | climb: 14, stand: 307 |
| [scriptdata/waypoints/mp_bo2hydro_wp.csv](../scriptdata/waypoints/mp_bo2hydro_wp.csv) | Navigation graph for `mp_bo2hydro`; supplies positions and traversable links for bot routing. | 259 | climb: 16, stand: 243 |
| [scriptdata/waypoints/mp_bo2meltdown2_wp.csv](../scriptdata/waypoints/mp_bo2meltdown2_wp.csv) | Navigation graph for `mp_bo2meltdown2`; supplies positions and traversable links for bot routing. | 315 | climb: 15, stand: 300 |
| [scriptdata/waypoints/mp_bo2meltdown_wp.csv](../scriptdata/waypoints/mp_bo2meltdown_wp.csv) | Navigation graph for `mp_bo2meltdown`; supplies positions and traversable links for bot routing. | 315 | climb: 15, stand: 300 |
| [scriptdata/waypoints/mp_bo2mir_wp.csv](../scriptdata/waypoints/mp_bo2mir_wp.csv) | Navigation graph for `mp_bo2mir`; supplies positions and traversable links for bot routing. | 427 | climb: 12, stand: 415 |
| [scriptdata/waypoints/mp_bo2nuketown2_wp.csv](../scriptdata/waypoints/mp_bo2nuketown2_wp.csv) | Navigation graph for `mp_bo2nuketown2`; supplies positions and traversable links for bot routing. | 198 | climb: 7, stand: 191 |
| [scriptdata/waypoints/mp_bo2nuketown_wp.csv](../scriptdata/waypoints/mp_bo2nuketown_wp.csv) | Navigation graph for `mp_bo2nuketown`; supplies positions and traversable links for bot routing. | 198 | climb: 7, stand: 191 |
| [scriptdata/waypoints/mp_bo2overflow_wp.csv](../scriptdata/waypoints/mp_bo2overflow_wp.csv) | Navigation graph for `mp_bo2overflow`; supplies positions and traversable links for bot routing. | 188 | climb: 2, stand: 186 |
| [scriptdata/waypoints/mp_bo2paintball_wp.csv](../scriptdata/waypoints/mp_bo2paintball_wp.csv) | Navigation graph for `mp_bo2paintball`; supplies positions and traversable links for bot routing. | 227 | climb: 2, stand: 225 |
| [scriptdata/waypoints/mp_bo2plaza_wp.csv](../scriptdata/waypoints/mp_bo2plaza_wp.csv) | Navigation graph for `mp_bo2plaza`; supplies positions and traversable links for bot routing. | 161 | climb: 4, stand: 157 |
| [scriptdata/waypoints/mp_bo2raid3_wp.csv](../scriptdata/waypoints/mp_bo2raid3_wp.csv) | Navigation graph for `mp_bo2raid3`; supplies positions and traversable links for bot routing. | 314 | climb: 1, stand: 313 |
| [scriptdata/waypoints/mp_bo2raid_wp.csv](../scriptdata/waypoints/mp_bo2raid_wp.csv) | Navigation graph for `mp_bo2raid`; supplies positions and traversable links for bot routing. | 260 | stand: 260 |
| [scriptdata/waypoints/mp_bo2slums_wp.csv](../scriptdata/waypoints/mp_bo2slums_wp.csv) | Navigation graph for `mp_bo2slums`; supplies positions and traversable links for bot routing. | 246 | stand: 246 |
| [scriptdata/waypoints/mp_bo2standoff_ex_wp.csv](../scriptdata/waypoints/mp_bo2standoff_ex_wp.csv) | Navigation graph for `mp_bo2standoff_ex`; supplies positions and traversable links for bot routing. | 464 | climb: 17, stand: 447 |
| [scriptdata/waypoints/mp_bo2standoff_wp.csv](../scriptdata/waypoints/mp_bo2standoff_wp.csv) | Navigation graph for `mp_bo2standoff`; supplies positions and traversable links for bot routing. | 363 | climb: 15, stand: 348 |
| [scriptdata/waypoints/mp_bo2turbine_wp.csv](../scriptdata/waypoints/mp_bo2turbine_wp.csv) | Navigation graph for `mp_bo2turbine`; supplies positions and traversable links for bot routing. | 417 | stand: 417 |
| [scriptdata/waypoints/mp_bo2yemen_wp.csv](../scriptdata/waypoints/mp_bo2yemen_wp.csv) | Navigation graph for `mp_bo2yemen`; supplies positions and traversable links for bot routing. | 364 | climb: 11, stand: 353 |
| [scriptdata/waypoints/mp_bog_wp.csv](../scriptdata/waypoints/mp_bog_wp.csv) | Navigation graph for `mp_bog`; supplies positions and traversable links for bot routing. | 138 | stand: 138 |
| [scriptdata/waypoints/mp_bookstore_wp.csv](../scriptdata/waypoints/mp_bookstore_wp.csv) | Navigation graph for `mp_bookstore`; supplies positions and traversable links for bot routing. | 156 | stand: 156 |
| [scriptdata/waypoints/mp_bordertown_final_wp.csv](../scriptdata/waypoints/mp_bordertown_final_wp.csv) | Navigation graph for `mp_bordertown_final`; supplies positions and traversable links for bot routing. | 400 | climb: 12, crouch: 2, stand: 386 |
| [scriptdata/waypoints/mp_borisbeta_wp.csv](../scriptdata/waypoints/mp_borisbeta_wp.csv) | Navigation graph for `mp_borisbeta`; supplies positions and traversable links for bot routing. | 383 | stand: 383 |
| [scriptdata/waypoints/mp_box_hjacked_wp.csv](../scriptdata/waypoints/mp_box_hjacked_wp.csv) | Navigation graph for `mp_box_hjacked`; supplies positions and traversable links for bot routing. | 185 | climb: 12, stand: 173 |
| [scriptdata/waypoints/mp_brdo_wp.csv](../scriptdata/waypoints/mp_brdo_wp.csv) | Navigation graph for `mp_brdo`; supplies positions and traversable links for bot routing. | 486 | stand: 486 |
| [scriptdata/waypoints/mp_breakout_wp.csv](../scriptdata/waypoints/mp_breakout_wp.csv) | Navigation graph for `mp_breakout`; supplies positions and traversable links for bot routing. | 385 | crouch: 2, stand: 383 |
| [scriptdata/waypoints/mp_brecourt_manor_wp.csv](../scriptdata/waypoints/mp_brecourt_manor_wp.csv) | Navigation graph for `mp_brecourt_manor`; supplies positions and traversable links for bot routing. | 671 | climb: 4, stand: 667 |
| [scriptdata/waypoints/mp_brecourt_v2_wp.csv](../scriptdata/waypoints/mp_brecourt_v2_wp.csv) | Navigation graph for `mp_brecourt_v2`; supplies positions and traversable links for bot routing. | 488 | jump: 15, mantle: 15, stand: 458 |
| [scriptdata/waypoints/mp_bridge2011_wp.csv](../scriptdata/waypoints/mp_bridge2011_wp.csv) | Navigation graph for `mp_bridge2011`; supplies positions and traversable links for bot routing. | 494 | climb: 4, stand: 490 |
| [scriptdata/waypoints/mp_bridge_wp.csv](../scriptdata/waypoints/mp_bridge_wp.csv) | Navigation graph for `mp_bridge`; supplies positions and traversable links for bot routing. | 386 | climb: 2, crouch: 7, stand: 377 |
| [scriptdata/waypoints/mp_broadcast2_wp.csv](../scriptdata/waypoints/mp_broadcast2_wp.csv) | Navigation graph for `mp_broadcast2`; supplies positions and traversable links for bot routing. | 359 | climb: 2, stand: 357 |
| [scriptdata/waypoints/mp_broadcast_wp.csv](../scriptdata/waypoints/mp_broadcast_wp.csv) | Navigation graph for `mp_broadcast`; supplies positions and traversable links for bot routing. | 287 | climb: 2, stand: 285 |
| [scriptdata/waypoints/mp_burg_b1_wp.csv](../scriptdata/waypoints/mp_burg_b1_wp.csv) | Navigation graph for `mp_burg_b1`; supplies positions and traversable links for bot routing. | 549 | climb: 18, prone: 2, stand: 529 |
| [scriptdata/waypoints/mp_burgundy_bulls_wp.csv](../scriptdata/waypoints/mp_burgundy_bulls_wp.csv) | Navigation graph for `mp_burgundy_bulls`; supplies positions and traversable links for bot routing. | 466 | climb: 2, stand: 464 |
| [scriptdata/waypoints/mp_caen_wp.csv](../scriptdata/waypoints/mp_caen_wp.csv) | Navigation graph for `mp_caen`; supplies positions and traversable links for bot routing. | 370 | undefined: 370 |
| [scriptdata/waypoints/mp_canal_bf3_wp.csv](../scriptdata/waypoints/mp_canal_bf3_wp.csv) | Navigation graph for `mp_canal_bf3`; supplies positions and traversable links for bot routing. | 621 | climb: 2, stand: 619 |
| [scriptdata/waypoints/mp_carentan44_wp.csv](../scriptdata/waypoints/mp_carentan44_wp.csv) | Navigation graph for `mp_carentan44`; supplies positions and traversable links for bot routing. | 352 | stand: 352 |
| [scriptdata/waypoints/mp_carentan_2_wp.csv](../scriptdata/waypoints/mp_carentan_2_wp.csv) | Navigation graph for `mp_carentan_2`; supplies positions and traversable links for bot routing. | 336 | stand: 336 |
| [scriptdata/waypoints/mp_carentan_wp.csv](../scriptdata/waypoints/mp_carentan_wp.csv) | Navigation graph for `mp_carentan`; supplies positions and traversable links for bot routing. | 285 | stand: 285 |
| [scriptdata/waypoints/mp_cargoship_wp.csv](../scriptdata/waypoints/mp_cargoship_wp.csv) | Navigation graph for `mp_cargoship`; supplies positions and traversable links for bot routing. | 234 | stand: 234 |
| [scriptdata/waypoints/mp_castle_v1_wp.csv](../scriptdata/waypoints/mp_castle_v1_wp.csv) | Navigation graph for `mp_castle_v1`; supplies positions and traversable links for bot routing. | 255 | climb: 2, stand: 253 |
| [scriptdata/waypoints/mp_cgc_bog_wp.csv](../scriptdata/waypoints/mp_cgc_bog_wp.csv) | Navigation graph for `mp_cgc_bog`; supplies positions and traversable links for bot routing. | 125 | stand: 125 |
| [scriptdata/waypoints/mp_chaos_lm_wp.csv](../scriptdata/waypoints/mp_chaos_lm_wp.csv) | Navigation graph for `mp_chaos_lm`; supplies positions and traversable links for bot routing. | 427 | climb: 10, stand: 417 |
| [scriptdata/waypoints/mp_chateau_wp.csv](../scriptdata/waypoints/mp_chateau_wp.csv) | Navigation graph for `mp_chateau`; supplies positions and traversable links for bot routing. | 196 | stand: 196 |
| [scriptdata/waypoints/mp_chernobyl_wp.csv](../scriptdata/waypoints/mp_chernobyl_wp.csv) | Navigation graph for `mp_chernobyl`; supplies positions and traversable links for bot routing. | 656 | climb: 27, stand: 629 |
| [scriptdata/waypoints/mp_citi_hotel_wp.csv](../scriptdata/waypoints/mp_citi_hotel_wp.csv) | Navigation graph for `mp_citi_hotel`; supplies positions and traversable links for bot routing. | 425 | climb: 2, stand: 423 |
| [scriptdata/waypoints/mp_citystreets_wp.csv](../scriptdata/waypoints/mp_citystreets_wp.csv) | Navigation graph for `mp_citystreets`; supplies positions and traversable links for bot routing. | 206 | stand: 206 |
| [scriptdata/waypoints/mp_cloud_city_wp.csv](../scriptdata/waypoints/mp_cloud_city_wp.csv) | Navigation graph for `mp_cloud_city`; supplies positions and traversable links for bot routing. | 230 | undefined: 230 |
| [scriptdata/waypoints/mp_cmm_wp.csv](../scriptdata/waypoints/mp_cmm_wp.csv) | Navigation graph for `mp_cmm`; supplies positions and traversable links for bot routing. | 148 | climb: 2, crouch: 3, stand: 143 |
| [scriptdata/waypoints/mp_cod4_assault_wp.csv](../scriptdata/waypoints/mp_cod4_assault_wp.csv) | Navigation graph for `mp_cod4_assault`; supplies positions and traversable links for bot routing. | 144 | climb: 10, stand: 134 |
| [scriptdata/waypoints/mp_colonel_wp.csv](../scriptdata/waypoints/mp_colonel_wp.csv) | Navigation graph for `mp_colonel`; supplies positions and traversable links for bot routing. | 808 | stand: 808 |
| [scriptdata/waypoints/mp_compact_wp.csv](../scriptdata/waypoints/mp_compact_wp.csv) | Navigation graph for `mp_compact`; supplies positions and traversable links for bot routing. | 167 | climb: 10, stand: 157 |
| [scriptdata/waypoints/mp_complex_wp.csv](../scriptdata/waypoints/mp_complex_wp.csv) | Navigation graph for `mp_complex`; supplies positions and traversable links for bot routing. | 421 | climb: 18, crouch: 14, stand: 389 |
| [scriptdata/waypoints/mp_construction_wp.csv](../scriptdata/waypoints/mp_construction_wp.csv) | Navigation graph for `mp_construction`; supplies positions and traversable links for bot routing. | 92 | stand: 92 |
| [scriptdata/waypoints/mp_convoy_wp.csv](../scriptdata/waypoints/mp_convoy_wp.csv) | Navigation graph for `mp_convoy`; supplies positions and traversable links for bot routing. | 238 | climb: 2, stand: 236 |
| [scriptdata/waypoints/mp_countdown_wp.csv](../scriptdata/waypoints/mp_countdown_wp.csv) | Navigation graph for `mp_countdown`; supplies positions and traversable links for bot routing. | 197 | stand: 197 |
| [scriptdata/waypoints/mp_crash2_wp.csv](../scriptdata/waypoints/mp_crash2_wp.csv) | Navigation graph for `mp_crash2`; supplies positions and traversable links for bot routing. | 294 | climb: 8, stand: 286 |
| [scriptdata/waypoints/mp_crash_snow_wp.csv](../scriptdata/waypoints/mp_crash_snow_wp.csv) | Navigation graph for `mp_crash_snow`; supplies positions and traversable links for bot routing. | 227 | climb: 8, stand: 219 |
| [scriptdata/waypoints/mp_crash_wp.csv](../scriptdata/waypoints/mp_crash_wp.csv) | Navigation graph for `mp_crash`; supplies positions and traversable links for bot routing. | 224 | climb: 8, stand: 216 |
| [scriptdata/waypoints/mp_creek_wp.csv](../scriptdata/waypoints/mp_creek_wp.csv) | Navigation graph for `mp_creek`; supplies positions and traversable links for bot routing. | 307 | climb: 2, stand: 305 |
| [scriptdata/waypoints/mp_crossfire2_wp.csv](../scriptdata/waypoints/mp_crossfire2_wp.csv) | Navigation graph for `mp_crossfire2`; supplies positions and traversable links for bot routing. | 376 | climb: 8, stand: 368 |
| [scriptdata/waypoints/mp_crossfire_wp.csv](../scriptdata/waypoints/mp_crossfire_wp.csv) | Navigation graph for `mp_crossfire`; supplies positions and traversable links for bot routing. | 303 | climb: 4, stand: 299 |
| [scriptdata/waypoints/mp_ctan_wp.csv](../scriptdata/waypoints/mp_ctan_wp.csv) | Navigation graph for `mp_ctan`; supplies positions and traversable links for bot routing. | 233 | stand: 233 |
| [scriptdata/waypoints/mp_d2c_wp.csv](../scriptdata/waypoints/mp_d2c_wp.csv) | Navigation graph for `mp_d2c`; supplies positions and traversable links for bot routing. | 69 | undefined: 69 |
| [scriptdata/waypoints/mp_dahar_wp.csv](../scriptdata/waypoints/mp_dahar_wp.csv) | Navigation graph for `mp_dahar`; supplies positions and traversable links for bot routing. | 330 | crouch: 5, stand: 325 |
| [scriptdata/waypoints/mp_damnalley_wp.csv](../scriptdata/waypoints/mp_damnalley_wp.csv) | Navigation graph for `mp_damnalley`; supplies positions and traversable links for bot routing. | 181 | undefined: 181 |
| [scriptdata/waypoints/mp_dawnville_wp.csv](../scriptdata/waypoints/mp_dawnville_wp.csv) | Navigation graph for `mp_dawnville`; supplies positions and traversable links for bot routing. | 309 | stand: 309 |
| [scriptdata/waypoints/mp_de_aztec_wp.csv](../scriptdata/waypoints/mp_de_aztec_wp.csv) | Navigation graph for `mp_de_aztec`; supplies positions and traversable links for bot routing. | 121 | stand: 121 |
| [scriptdata/waypoints/mp_decoytrenches_wp.csv](../scriptdata/waypoints/mp_decoytrenches_wp.csv) | Navigation graph for `mp_decoytrenches`; supplies positions and traversable links for bot routing. | 307 | climb: 8, stand: 299 |
| [scriptdata/waypoints/mp_desertglory_wp.csv](../scriptdata/waypoints/mp_desertglory_wp.csv) | Navigation graph for `mp_desertglory`; supplies positions and traversable links for bot routing. | 702 | stand: 702 |
| [scriptdata/waypoints/mp_desfarm_night_wp.csv](../scriptdata/waypoints/mp_desfarm_night_wp.csv) | Navigation graph for `mp_desfarm_night`; supplies positions and traversable links for bot routing. | 216 | climb: 6, stand: 210 |
| [scriptdata/waypoints/mp_desfarm_wp.csv](../scriptdata/waypoints/mp_desfarm_wp.csv) | Navigation graph for `mp_desfarm`; supplies positions and traversable links for bot routing. | 181 | climb: 12, stand: 169 |
| [scriptdata/waypoints/mp_dhdjunkyard_wp.csv](../scriptdata/waypoints/mp_dhdjunkyard_wp.csv) | Navigation graph for `mp_dhdjunkyard`; supplies positions and traversable links for bot routing. | 426 | climb: 6, crouch: 17, stand: 403 |
| [scriptdata/waypoints/mp_disputed_wp.csv](../scriptdata/waypoints/mp_disputed_wp.csv) | Navigation graph for `mp_disputed`; supplies positions and traversable links for bot routing. | 279 | undefined: 279 |
| [scriptdata/waypoints/mp_docks_n_wp.csv](../scriptdata/waypoints/mp_docks_n_wp.csv) | Navigation graph for `mp_docks_n`; supplies positions and traversable links for bot routing. | 350 | stand: 350 |
| [scriptdata/waypoints/mp_docks_wp.csv](../scriptdata/waypoints/mp_docks_wp.csv) | Navigation graph for `mp_docks`; supplies positions and traversable links for bot routing. | 350 | stand: 350 |
| [scriptdata/waypoints/mp_dome_wp.csv](../scriptdata/waypoints/mp_dome_wp.csv) | Navigation graph for `mp_dome`; supplies positions and traversable links for bot routing. | 231 | climb: 6, stand: 225 |
| [scriptdata/waypoints/mp_doowood_wp.csv](../scriptdata/waypoints/mp_doowood_wp.csv) | Navigation graph for `mp_doowood`; supplies positions and traversable links for bot routing. | 215 | climb: 12, stand: 203 |
| [scriptdata/waypoints/mp_downtown_wp.csv](../scriptdata/waypoints/mp_downtown_wp.csv) | Navigation graph for `mp_downtown`; supplies positions and traversable links for bot routing. | 238 | climb: 2, stand: 236 |
| [scriptdata/waypoints/mp_dust2_classic_wp.csv](../scriptdata/waypoints/mp_dust2_classic_wp.csv) | Navigation graph for `mp_dust2_classic`; supplies positions and traversable links for bot routing. | 114 | stand: 114 |
| [scriptdata/waypoints/mp_dust2_wp.csv](../scriptdata/waypoints/mp_dust2_wp.csv) | Navigation graph for `mp_dust2`; supplies positions and traversable links for bot routing. | 97 | climb: 2, stand: 95 |
| [scriptdata/waypoints/mp_dustcod4_wp.csv](../scriptdata/waypoints/mp_dustcod4_wp.csv) | Navigation graph for `mp_dustcod4`; supplies positions and traversable links for bot routing. | 127 | climb: 4, stand: 123 |
| [scriptdata/waypoints/mp_dv_wp.csv](../scriptdata/waypoints/mp_dv_wp.csv) | Navigation graph for `mp_dv`; supplies positions and traversable links for bot routing. | 322 | climb: 4, stand: 318 |
| [scriptdata/waypoints/mp_e_streetstation_wp.csv](../scriptdata/waypoints/mp_e_streetstation_wp.csv) | Navigation graph for `mp_e_streetstation`; supplies positions and traversable links for bot routing. | 601 | climb: 27, stand: 574 |
| [scriptdata/waypoints/mp_fabrika_wp.csv](../scriptdata/waypoints/mp_fabrika_wp.csv) | Navigation graph for `mp_fabrika`; supplies positions and traversable links for bot routing. | 290 | climb: 6, crouch: 8, stand: 276 |
| [scriptdata/waypoints/mp_fallen_n_wp.csv](../scriptdata/waypoints/mp_fallen_n_wp.csv) | Navigation graph for `mp_fallen_n`; supplies positions and traversable links for bot routing. | 293 | stand: 293 |
| [scriptdata/waypoints/mp_fallen_wp.csv](../scriptdata/waypoints/mp_fallen_wp.csv) | Navigation graph for `mp_fallen`; supplies positions and traversable links for bot routing. | 293 | stand: 293 |
| [scriptdata/waypoints/mp_fallout_wp.csv](../scriptdata/waypoints/mp_fallout_wp.csv) | Navigation graph for `mp_fallout`; supplies positions and traversable links for bot routing. Contains malformed link token(s); see review notes. | 492 | climb: 8, crouch: 22, stand: 462 |
| [scriptdata/waypoints/mp_farm_wp.csv](../scriptdata/waypoints/mp_farm_wp.csv) | Navigation graph for `mp_farm`; supplies positions and traversable links for bot routing. | 241 | climb: 2, stand: 239 |
| [scriptdata/waypoints/mp_fav_wp.csv](../scriptdata/waypoints/mp_fav_wp.csv) | Navigation graph for `mp_fav`; supplies positions and traversable links for bot routing. | 235 | climb: 10, stand: 225 |
| [scriptdata/waypoints/mp_firingrange_v2_wp.csv](../scriptdata/waypoints/mp_firingrange_v2_wp.csv) | Navigation graph for `mp_firingrange_v2`; supplies positions and traversable links for bot routing. | 122 | climb: 6, stand: 116 |
| [scriptdata/waypoints/mp_forest2_wp.csv](../scriptdata/waypoints/mp_forest2_wp.csv) | Navigation graph for `mp_forest2`; supplies positions and traversable links for bot routing. | 254 | climb: 4, stand: 250 |
| [scriptdata/waypoints/mp_forlorn_wp.csv](../scriptdata/waypoints/mp_forlorn_wp.csv) | Navigation graph for `mp_forlorn`; supplies positions and traversable links for bot routing. | 189 | undefined: 189 |
| [scriptdata/waypoints/mp_fr_hv3_wp.csv](../scriptdata/waypoints/mp_fr_hv3_wp.csv) | Navigation graph for `mp_fr_hv3`; supplies positions and traversable links for bot routing. | 632 | stand: 632 |
| [scriptdata/waypoints/mp_fr_street_wp.csv](../scriptdata/waypoints/mp_fr_street_wp.csv) | Navigation graph for `mp_fr_street`; supplies positions and traversable links for bot routing. | 448 | climb: 4, stand: 444 |
| [scriptdata/waypoints/mp_freeb_wp.csv](../scriptdata/waypoints/mp_freeb_wp.csv) | Navigation graph for `mp_freeb`; supplies positions and traversable links for bot routing. | 582 | climb: 8, crouch: 15, stand: 559 |
| [scriptdata/waypoints/mp_fregata_wp.csv](../scriptdata/waypoints/mp_fregata_wp.csv) | Navigation graph for `mp_fregata`; supplies positions and traversable links for bot routing. | 183 | crouch: 2, stand: 181 |
| [scriptdata/waypoints/mp_fritzberg_wp.csv](../scriptdata/waypoints/mp_fritzberg_wp.csv) | Navigation graph for `mp_fritzberg`; supplies positions and traversable links for bot routing. | 390 | climb: 2, stand: 388 |
| [scriptdata/waypoints/mp_funland_wp.csv](../scriptdata/waypoints/mp_funland_wp.csv) | Navigation graph for `mp_funland`; supplies positions and traversable links for bot routing. | 219 | stand: 219 |
| [scriptdata/waypoints/mp_garena_wp.csv](../scriptdata/waypoints/mp_garena_wp.csv) | Navigation graph for `mp_garena`; supplies positions and traversable links for bot routing. | 301 | climb: 8, stand: 293 |
| [scriptdata/waypoints/mp_gob_aim_snow_wp.csv](../scriptdata/waypoints/mp_gob_aim_snow_wp.csv) | Navigation graph for `mp_gob_aim_snow`; supplies positions and traversable links for bot routing. | 95 | climb: 4, stand: 91 |
| [scriptdata/waypoints/mp_gob_aim_v2_wp.csv](../scriptdata/waypoints/mp_gob_aim_v2_wp.csv) | Navigation graph for `mp_gob_aim_v2`; supplies positions and traversable links for bot routing. | 95 | climb: 4, stand: 91 |
| [scriptdata/waypoints/mp_hangareturn_wp.csv](../scriptdata/waypoints/mp_hangareturn_wp.csv) | Navigation graph for `mp_hangareturn`; supplies positions and traversable links for bot routing. | 212 | climb: 8, stand: 204 |
| [scriptdata/waypoints/mp_hardhat_wp.csv](../scriptdata/waypoints/mp_hardhat_wp.csv) | Navigation graph for `mp_hardhat`; supplies positions and traversable links for bot routing. | 285 | stand: 285 |
| [scriptdata/waypoints/mp_haus_wp.csv](../scriptdata/waypoints/mp_haus_wp.csv) | Navigation graph for `mp_haus`; supplies positions and traversable links for bot routing. | 278 | climb: 13, crouch: 15, stand: 250 |
| [scriptdata/waypoints/mp_hhk_ballroom_wp.csv](../scriptdata/waypoints/mp_hhk_ballroom_wp.csv) | Navigation graph for `mp_hhk_ballroom`; supplies positions and traversable links for bot routing. | 215 | stand: 215 |
| [scriptdata/waypoints/mp_highrise_wp.csv](../scriptdata/waypoints/mp_highrise_wp.csv) | Navigation graph for `mp_highrise`; supplies positions and traversable links for bot routing. | 237 | stand: 237 |
| [scriptdata/waypoints/mp_hims_wp.csv](../scriptdata/waypoints/mp_hims_wp.csv) | Navigation graph for `mp_hims`; supplies positions and traversable links for bot routing. | 336 | undefined: 336 |
| [scriptdata/waypoints/mp_history_ground_wp.csv](../scriptdata/waypoints/mp_history_ground_wp.csv) | Navigation graph for `mp_history_ground`; supplies positions and traversable links for bot routing. | 362 | crouch: 2, stand: 360 |
| [scriptdata/waypoints/mp_hk2_wp.csv](../scriptdata/waypoints/mp_hk2_wp.csv) | Navigation graph for `mp_hk2`; supplies positions and traversable links for bot routing. | 214 | stand: 214 |
| [scriptdata/waypoints/mp_hollow_wp.csv](../scriptdata/waypoints/mp_hollow_wp.csv) | Navigation graph for `mp_hollow`; supplies positions and traversable links for bot routing. | 314 | climb: 2, stand: 312 |
| [scriptdata/waypoints/mp_houses_wp.csv](../scriptdata/waypoints/mp_houses_wp.csv) | Navigation graph for `mp_houses`; supplies positions and traversable links for bot routing. | 149 | stand: 149 |
| [scriptdata/waypoints/mp_hurtgen_s_wp.csv](../scriptdata/waypoints/mp_hurtgen_s_wp.csv) | Navigation graph for `mp_hurtgen_s`; supplies positions and traversable links for bot routing. | 368 | climb: 2, stand: 366 |
| [scriptdata/waypoints/mp_i2_wp.csv](../scriptdata/waypoints/mp_i2_wp.csv) | Navigation graph for `mp_i2`; supplies positions and traversable links for bot routing. | 475 | climb: 46, stand: 429 |
| [scriptdata/waypoints/mp_ic_cod_wp.csv](../scriptdata/waypoints/mp_ic_cod_wp.csv) | Navigation graph for `mp_ic_cod`; supplies positions and traversable links for bot routing. | 283 | stand: 283 |
| [scriptdata/waypoints/mp_iclaim_wp.csv](../scriptdata/waypoints/mp_iclaim_wp.csv) | Navigation graph for `mp_iclaim`; supplies positions and traversable links for bot routing. | 164 | climb: 8, stand: 156 |
| [scriptdata/waypoints/mp_iclan_night_wp.csv](../scriptdata/waypoints/mp_iclan_night_wp.csv) | Navigation graph for `mp_iclan_night`; supplies positions and traversable links for bot routing. | 496 | climb: 4, stand: 492 |
| [scriptdata/waypoints/mp_iclan_wp.csv](../scriptdata/waypoints/mp_iclan_wp.csv) | Navigation graph for `mp_iclan`; supplies positions and traversable links for bot routing. | 496 | climb: 4, stand: 492 |
| [scriptdata/waypoints/mp_incursion_wp.csv](../scriptdata/waypoints/mp_incursion_wp.csv) | Navigation graph for `mp_incursion`; supplies positions and traversable links for bot routing. | 419 | climb: 7, stand: 412 |
| [scriptdata/waypoints/mp_inferno_wp.csv](../scriptdata/waypoints/mp_inferno_wp.csv) | Navigation graph for `mp_inferno`; supplies positions and traversable links for bot routing. | 228 | undefined: 228 |
| [scriptdata/waypoints/mp_infi_v16_wp.csv](../scriptdata/waypoints/mp_infi_v16_wp.csv) | Navigation graph for `mp_infi_v16`; supplies positions and traversable links for bot routing. | 197 | climb: 23, stand: 174 |
| [scriptdata/waypoints/mp_inv_wp.csv](../scriptdata/waypoints/mp_inv_wp.csv) | Navigation graph for `mp_inv`; supplies positions and traversable links for bot routing. | 245 | climb: 14, stand: 231 |
| [scriptdata/waypoints/mp_iranian_embassy2_wp.csv](../scriptdata/waypoints/mp_iranian_embassy2_wp.csv) | Navigation graph for `mp_iranian_embassy2`; supplies positions and traversable links for bot routing. | 366 | climb: 2, crouch: 1, stand: 363 |
| [scriptdata/waypoints/mp_japmap_wp.csv](../scriptdata/waypoints/mp_japmap_wp.csv) | Navigation graph for `mp_japmap`; supplies positions and traversable links for bot routing. | 570 | stand: 570 |
| [scriptdata/waypoints/mp_jarak_wp.csv](../scriptdata/waypoints/mp_jarak_wp.csv) | Navigation graph for `mp_jarak`; supplies positions and traversable links for bot routing. | 265 | climb: 6, stand: 259 |
| [scriptdata/waypoints/mp_jundland_wp.csv](../scriptdata/waypoints/mp_jundland_wp.csv) | Navigation graph for `mp_jundland`; supplies positions and traversable links for bot routing. | 238 | climb: 2, stand: 236 |
| [scriptdata/waypoints/mp_karachi_wp.csv](../scriptdata/waypoints/mp_karachi_wp.csv) | Navigation graph for `mp_karachi`; supplies positions and traversable links for bot routing. | 255 | climb: 10, stand: 245 |
| [scriptdata/waypoints/mp_karrak_wp.csv](../scriptdata/waypoints/mp_karrak_wp.csv) | Navigation graph for `mp_karrak`; supplies positions and traversable links for bot routing. | 387 | undefined: 387 |
| [scriptdata/waypoints/mp_kgboffice_wp.csv](../scriptdata/waypoints/mp_kgboffice_wp.csv) | Navigation graph for `mp_kgboffice`; supplies positions and traversable links for bot routing. | 350 | stand: 350 |
| [scriptdata/waypoints/mp_kh1_wp.csv](../scriptdata/waypoints/mp_kh1_wp.csv) | Navigation graph for `mp_kh1`; supplies positions and traversable links for bot routing. | 427 | undefined: 427 |
| [scriptdata/waypoints/mp_killhouse2_wp.csv](../scriptdata/waypoints/mp_killhouse2_wp.csv) | Navigation graph for `mp_killhouse2`; supplies positions and traversable links for bot routing. | 208 | undefined: 208 |
| [scriptdata/waypoints/mp_killhouse_wp.csv](../scriptdata/waypoints/mp_killhouse_wp.csv) | Navigation graph for `mp_killhouse`; supplies positions and traversable links for bot routing. | 112 | stand: 112 |
| [scriptdata/waypoints/mp_kk_arena2_wp.csv](../scriptdata/waypoints/mp_kk_arena2_wp.csv) | Navigation graph for `mp_kk_arena2`; supplies positions and traversable links for bot routing. | 387 | climb: 16, stand: 371 |
| [scriptdata/waypoints/mp_kk_crypt_wp.csv](../scriptdata/waypoints/mp_kk_crypt_wp.csv) | Navigation graph for `mp_kk_crypt`; supplies positions and traversable links for bot routing. | 249 | climb: 2, stand: 247 |
| [scriptdata/waypoints/mp_kodash_wp.csv](../scriptdata/waypoints/mp_kodash_wp.csv) | Navigation graph for `mp_kodash`; supplies positions and traversable links for bot routing. | 335 | climb: 4, stand: 331 |
| [scriptdata/waypoints/mp_kowloon_wp.csv](../scriptdata/waypoints/mp_kowloon_wp.csv) | Navigation graph for `mp_kowloon`; supplies positions and traversable links for bot routing. | 207 | climb: 6, crouch: 25, stand: 176 |
| [scriptdata/waypoints/mp_kstoujane_wp.csv](../scriptdata/waypoints/mp_kstoujane_wp.csv) | Navigation graph for `mp_kstoujane`; supplies positions and traversable links for bot routing. | 285 | stand: 285 |
| [scriptdata/waypoints/mp_kufah_wp.csv](../scriptdata/waypoints/mp_kufah_wp.csv) | Navigation graph for `mp_kufah`; supplies positions and traversable links for bot routing. | 224 | climb: 2, crouch: 2, prone: 1, stand: 219 |
| [scriptdata/waypoints/mp_lagoa_rain_wp.csv](../scriptdata/waypoints/mp_lagoa_rain_wp.csv) | Navigation graph for `mp_lagoa_rain`; supplies positions and traversable links for bot routing. | 588 | climb: 46, crouch: 2, stand: 540 |
| [scriptdata/waypoints/mp_lagoa_snow_wp.csv](../scriptdata/waypoints/mp_lagoa_snow_wp.csv) | Navigation graph for `mp_lagoa_snow`; supplies positions and traversable links for bot routing. | 588 | climb: 46, crouch: 2, stand: 540 |
| [scriptdata/waypoints/mp_lake_wp.csv](../scriptdata/waypoints/mp_lake_wp.csv) | Navigation graph for `mp_lake`; supplies positions and traversable links for bot routing. | 155 | climb: 9, stand: 146 |
| [scriptdata/waypoints/mp_lapatrouille_wp.csv](../scriptdata/waypoints/mp_lapatrouille_wp.csv) | Navigation graph for `mp_lapatrouille`; supplies positions and traversable links for bot routing. | 277 | climb: 2, stand: 275 |
| [scriptdata/waypoints/mp_lastlight_wp.csv](../scriptdata/waypoints/mp_lastlight_wp.csv) | Navigation graph for `mp_lastlight`; supplies positions and traversable links for bot routing. | 381 | climb: 10, stand: 371 |
| [scriptdata/waypoints/mp_legoland_wp.csv](../scriptdata/waypoints/mp_legoland_wp.csv) | Navigation graph for `mp_legoland`; supplies positions and traversable links for bot routing. | 119 | stand: 119 |
| [scriptdata/waypoints/mp_lolv2_wp.csv](../scriptdata/waypoints/mp_lolv2_wp.csv) | Navigation graph for `mp_lolv2`; supplies positions and traversable links for bot routing. | 443 | undefined: 443 |
| [scriptdata/waypoints/mp_lpost_wp.csv](../scriptdata/waypoints/mp_lpost_wp.csv) | Navigation graph for `mp_lpost`; supplies positions and traversable links for bot routing. | 235 | undefined: 235 |
| [scriptdata/waypoints/mp_lumberyard_wp.csv](../scriptdata/waypoints/mp_lumberyard_wp.csv) | Navigation graph for `mp_lumberyard`; supplies positions and traversable links for bot routing. | 360 | climb: 10, stand: 350 |
| [scriptdata/waypoints/mp_luna2_wp.csv](../scriptdata/waypoints/mp_luna2_wp.csv) | Navigation graph for `mp_luna2`; supplies positions and traversable links for bot routing. | 370 | climb: 22, stand: 348 |
| [scriptdata/waypoints/mp_lut_gholein_wp.csv](../scriptdata/waypoints/mp_lut_gholein_wp.csv) | Navigation graph for `mp_lut_gholein`; supplies positions and traversable links for bot routing. | 389 | climb: 4, stand: 385 |
| [scriptdata/waypoints/mp_mansion_wp.csv](../scriptdata/waypoints/mp_mansion_wp.csv) | Navigation graph for `mp_mansion`; supplies positions and traversable links for bot routing. | 88 | climb: 6, stand: 82 |
| [scriptdata/waypoints/mp_matroska_wp.csv](../scriptdata/waypoints/mp_matroska_wp.csv) | Navigation graph for `mp_matroska`; supplies positions and traversable links for bot routing. | 218 | stand: 218 |
| [scriptdata/waypoints/mp_me_wp.csv](../scriptdata/waypoints/mp_me_wp.csv) | Navigation graph for `mp_me`; supplies positions and traversable links for bot routing. | 146 | stand: 146 |
| [scriptdata/waypoints/mp_menight_wp.csv](../scriptdata/waypoints/mp_menight_wp.csv) | Navigation graph for `mp_menight`; supplies positions and traversable links for bot routing. | 146 | stand: 146 |
| [scriptdata/waypoints/mp_microsim_wp.csv](../scriptdata/waypoints/mp_microsim_wp.csv) | Navigation graph for `mp_microsim`; supplies positions and traversable links for bot routing. | 163 | stand: 163 |
| [scriptdata/waypoints/mp_mission_wp.csv](../scriptdata/waypoints/mp_mission_wp.csv) | Navigation graph for `mp_mission`; supplies positions and traversable links for bot routing. | 314 | climb: 8, stand: 306 |
| [scriptdata/waypoints/mp_modern_rust_wp.csv](../scriptdata/waypoints/mp_modern_rust_wp.csv) | Navigation graph for `mp_modern_rust`; supplies positions and traversable links for bot routing. | 90 | climb: 2, stand: 88 |
| [scriptdata/waypoints/mp_modernbacklot_wp.csv](../scriptdata/waypoints/mp_modernbacklot_wp.csv) | Navigation graph for `mp_modernbacklot`; supplies positions and traversable links for bot routing. | 323 | stand: 323 |
| [scriptdata/waypoints/mp_moh_sfrance_b1_wp.csv](../scriptdata/waypoints/mp_moh_sfrance_b1_wp.csv) | Navigation graph for `mp_moh_sfrance_b1`; supplies positions and traversable links for bot routing. | 205 | stand: 205 |
| [scriptdata/waypoints/mp_mom_wp.csv](../scriptdata/waypoints/mp_mom_wp.csv) | Navigation graph for `mp_mom`; supplies positions and traversable links for bot routing. | 201 | stand: 201 |
| [scriptdata/waypoints/mp_mount_wp.csv](../scriptdata/waypoints/mp_mount_wp.csv) | Navigation graph for `mp_mount`; supplies positions and traversable links for bot routing. | 291 | stand: 291 |
| [scriptdata/waypoints/mp_mtl_the_rock_wp.csv](../scriptdata/waypoints/mp_mtl_the_rock_wp.csv) | Navigation graph for `mp_mtl_the_rock`; supplies positions and traversable links for bot routing. Contains malformed link token(s); see review notes. | 578 | climb: 18, crouch: 4, stand: 556 |
| [scriptdata/waypoints/mp_mw2_quarry_night_wp.csv](../scriptdata/waypoints/mp_mw2_quarry_night_wp.csv) | Navigation graph for `mp_mw2_quarry_night`; supplies positions and traversable links for bot routing. | 272 | climb: 10, stand: 262 |
| [scriptdata/waypoints/mp_mw2_quarry_wp.csv](../scriptdata/waypoints/mp_mw2_quarry_wp.csv) | Navigation graph for `mp_mw2_quarry`; supplies positions and traversable links for bot routing. | 272 | climb: 10, stand: 262 |
| [scriptdata/waypoints/mp_mw2_rust_night_wp.csv](../scriptdata/waypoints/mp_mw2_rust_night_wp.csv) | Navigation graph for `mp_mw2_rust_night`; supplies positions and traversable links for bot routing. | 80 | stand: 80 |
| [scriptdata/waypoints/mp_mw2_rust_wp.csv](../scriptdata/waypoints/mp_mw2_rust_wp.csv) | Navigation graph for `mp_mw2_rust`; supplies positions and traversable links for bot routing. | 80 | stand: 80 |
| [scriptdata/waypoints/mp_mw2_term_wp.csv](../scriptdata/waypoints/mp_mw2_term_wp.csv) | Navigation graph for `mp_mw2_term`; supplies positions and traversable links for bot routing. | 222 | climb: 4, stand: 218 |
| [scriptdata/waypoints/mp_mw2_underpass_night_wp.csv](../scriptdata/waypoints/mp_mw2_underpass_night_wp.csv) | Navigation graph for `mp_mw2_underpass_night`; supplies positions and traversable links for bot routing. | 158 | stand: 158 |
| [scriptdata/waypoints/mp_mw2_underpass_wp.csv](../scriptdata/waypoints/mp_mw2_underpass_wp.csv) | Navigation graph for `mp_mw2_underpass`; supplies positions and traversable links for bot routing. | 158 | stand: 158 |
| [scriptdata/waypoints/mp_mw3_alpha_wp.csv](../scriptdata/waypoints/mp_mw3_alpha_wp.csv) | Navigation graph for `mp_mw3_alpha`; supplies positions and traversable links for bot routing. | 263 | stand: 263 |
| [scriptdata/waypoints/mp_mw3_hardhat_night_wp.csv](../scriptdata/waypoints/mp_mw3_hardhat_night_wp.csv) | Navigation graph for `mp_mw3_hardhat_night`; supplies positions and traversable links for bot routing. | 215 | climb: 2, stand: 213 |
| [scriptdata/waypoints/mp_mw3_hardhat_wp.csv](../scriptdata/waypoints/mp_mw3_hardhat_wp.csv) | Navigation graph for `mp_mw3_hardhat`; supplies positions and traversable links for bot routing. | 215 | climb: 2, stand: 213 |
| [scriptdata/waypoints/mp_mw3_lookout_wp.csv](../scriptdata/waypoints/mp_mw3_lookout_wp.csv) | Navigation graph for `mp_mw3_lookout`; supplies positions and traversable links for bot routing. | 38 | stand: 38 |
| [scriptdata/waypoints/mp_mw3_vortex_wp.csv](../scriptdata/waypoints/mp_mw3_vortex_wp.csv) | Navigation graph for `mp_mw3_vortex`; supplies positions and traversable links for bot routing. | 53 | stand: 53 |
| [scriptdata/waypoints/mp_mw3piazza_wp.csv](../scriptdata/waypoints/mp_mw3piazza_wp.csv) | Navigation graph for `mp_mw3piazza`; supplies positions and traversable links for bot routing. | 288 | stand: 288 |
| [scriptdata/waypoints/mp_mw3resistance_wp.csv](../scriptdata/waypoints/mp_mw3resistance_wp.csv) | Navigation graph for `mp_mw3resistance`; supplies positions and traversable links for bot routing. | 308 | climb: 4, stand: 304 |
| [scriptdata/waypoints/mp_naout_n_wp.csv](../scriptdata/waypoints/mp_naout_n_wp.csv) | Navigation graph for `mp_naout_n`; supplies positions and traversable links for bot routing. | 402 | climb: 2, crouch: 2, stand: 398 |
| [scriptdata/waypoints/mp_naout_wp.csv](../scriptdata/waypoints/mp_naout_wp.csv) | Navigation graph for `mp_naout`; supplies positions and traversable links for bot routing. | 402 | climb: 2, crouch: 2, stand: 398 |
| [scriptdata/waypoints/mp_ncc_1701_wp.csv](../scriptdata/waypoints/mp_ncc_1701_wp.csv) | Navigation graph for `mp_ncc_1701`; supplies positions and traversable links for bot routing. | 174 | climb: 4, stand: 170 |
| [scriptdata/waypoints/mp_ngbc_day_wp.csv](../scriptdata/waypoints/mp_ngbc_day_wp.csv) | Navigation graph for `mp_ngbc_day`; supplies positions and traversable links for bot routing. | 792 | stand: 792 |
| [scriptdata/waypoints/mp_ngbc_shipment_wp.csv](../scriptdata/waypoints/mp_ngbc_shipment_wp.csv) | Navigation graph for `mp_ngbc_shipment`; supplies positions and traversable links for bot routing. | 221 | stand: 221 |
| [scriptdata/waypoints/mp_ngbc_shipmentw_wp.csv](../scriptdata/waypoints/mp_ngbc_shipmentw_wp.csv) | Navigation graph for `mp_ngbc_shipmentw`; supplies positions and traversable links for bot routing. | 221 | stand: 221 |
| [scriptdata/waypoints/mp_ngbc_wp.csv](../scriptdata/waypoints/mp_ngbc_wp.csv) | Navigation graph for `mp_ngbc`; supplies positions and traversable links for bot routing. | 792 | stand: 792 |
| [scriptdata/waypoints/mp_noobville_wp.csv](../scriptdata/waypoints/mp_noobville_wp.csv) | Navigation graph for `mp_noobville`; supplies positions and traversable links for bot routing. | 459 | stand: 459 |
| [scriptdata/waypoints/mp_ntown_wp.csv](../scriptdata/waypoints/mp_ntown_wp.csv) | Navigation graph for `mp_ntown`; supplies positions and traversable links for bot routing. | 222 | stand: 222 |
| [scriptdata/waypoints/mp_nuketown_wp.csv](../scriptdata/waypoints/mp_nuketown_wp.csv) | Navigation graph for `mp_nuketown`; supplies positions and traversable links for bot routing. | 148 | stand: 148 |
| [scriptdata/waypoints/mp_office_wp.csv](../scriptdata/waypoints/mp_office_wp.csv) | Navigation graph for `mp_office`; supplies positions and traversable links for bot routing. | 127 | climb: 2, stand: 125 |
| [scriptdata/waypoints/mp_offices_v3_wp.csv](../scriptdata/waypoints/mp_offices_v3_wp.csv) | Navigation graph for `mp_offices_v3`; supplies positions and traversable links for bot routing. | 212 | climb: 2, stand: 210 |
| [scriptdata/waypoints/mp_oldschool_wp.csv](../scriptdata/waypoints/mp_oldschool_wp.csv) | Navigation graph for `mp_oldschool`; supplies positions and traversable links for bot routing. | 335 | stand: 335 |
| [scriptdata/waypoints/mp_ordnance_wp.csv](../scriptdata/waypoints/mp_ordnance_wp.csv) | Navigation graph for `mp_ordnance`; supplies positions and traversable links for bot routing. | 155 | climb: 2, stand: 153 |
| [scriptdata/waypoints/mp_osg_backlot_n_wp.csv](../scriptdata/waypoints/mp_osg_backlot_n_wp.csv) | Navigation graph for `mp_osg_backlot_n`; supplies positions and traversable links for bot routing. | 247 | climb: 2, stand: 245 |
| [scriptdata/waypoints/mp_osg_backlot_s_wp.csv](../scriptdata/waypoints/mp_osg_backlot_s_wp.csv) | Navigation graph for `mp_osg_backlot_s`; supplies positions and traversable links for bot routing. | 247 | climb: 2, stand: 245 |
| [scriptdata/waypoints/mp_osg_backlot_wp.csv](../scriptdata/waypoints/mp_osg_backlot_wp.csv) | Navigation graph for `mp_osg_backlot`; supplies positions and traversable links for bot routing. | 247 | climb: 2, stand: 245 |
| [scriptdata/waypoints/mp_osg_cargo_wp.csv](../scriptdata/waypoints/mp_osg_cargo_wp.csv) | Navigation graph for `mp_osg_cargo`; supplies positions and traversable links for bot routing. | 429 | climb: 44, stand: 385 |
| [scriptdata/waypoints/mp_osg_carrier_p_wp.csv](../scriptdata/waypoints/mp_osg_carrier_p_wp.csv) | Navigation graph for `mp_osg_carrier_p`; supplies positions and traversable links for bot routing. | 299 | climb: 4, stand: 295 |
| [scriptdata/waypoints/mp_osg_drone_p_wp.csv](../scriptdata/waypoints/mp_osg_drone_p_wp.csv) | Navigation graph for `mp_osg_drone_p`; supplies positions and traversable links for bot routing. | 310 | climb: 2, stand: 308 |
| [scriptdata/waypoints/mp_osg_hazzard_wp.csv](../scriptdata/waypoints/mp_osg_hazzard_wp.csv) | Navigation graph for `mp_osg_hazzard`; supplies positions and traversable links for bot routing. | 266 | stand: 266 |
| [scriptdata/waypoints/mp_osg_hijacked_wp.csv](../scriptdata/waypoints/mp_osg_hijacked_wp.csv) | Navigation graph for `mp_osg_hijacked`; supplies positions and traversable links for bot routing. | 236 | climb: 4, stand: 232 |
| [scriptdata/waypoints/mp_osg_mirage_n_wp.csv](../scriptdata/waypoints/mp_osg_mirage_n_wp.csv) | Navigation graph for `mp_osg_mirage_n`; supplies positions and traversable links for bot routing. | 369 | climb: 16, stand: 353 |
| [scriptdata/waypoints/mp_osg_mirage_wp.csv](../scriptdata/waypoints/mp_osg_mirage_wp.csv) | Navigation graph for `mp_osg_mirage`; supplies positions and traversable links for bot routing. | 376 | climb: 16, stand: 360 |
| [scriptdata/waypoints/mp_osg_raid_d_wp.csv](../scriptdata/waypoints/mp_osg_raid_d_wp.csv) | Navigation graph for `mp_osg_raid_d`; supplies positions and traversable links for bot routing. | 374 | climb: 9, stand: 365 |
| [scriptdata/waypoints/mp_osg_raid_wp.csv](../scriptdata/waypoints/mp_osg_raid_wp.csv) | Navigation graph for `mp_osg_raid`; supplies positions and traversable links for bot routing. | 314 | climb: 1, stand: 313 |
| [scriptdata/waypoints/mp_osg_roadrage_wp.csv](../scriptdata/waypoints/mp_osg_roadrage_wp.csv) | Navigation graph for `mp_osg_roadrage`; supplies positions and traversable links for bot routing. | 208 | stand: 208 |
| [scriptdata/waypoints/mp_osg_standoff_wp.csv](../scriptdata/waypoints/mp_osg_standoff_wp.csv) | Navigation graph for `mp_osg_standoff`; supplies positions and traversable links for bot routing. | 363 | climb: 15, stand: 348 |
| [scriptdata/waypoints/mp_osg_studio_2_wp.csv](../scriptdata/waypoints/mp_osg_studio_2_wp.csv) | Navigation graph for `mp_osg_studio_2`; supplies positions and traversable links for bot routing. | 333 | climb: 7, stand: 326 |
| [scriptdata/waypoints/mp_osg_tropical_rust_wp.csv](../scriptdata/waypoints/mp_osg_tropical_rust_wp.csv) | Navigation graph for `mp_osg_tropical_rust`; supplies positions and traversable links for bot routing. | 164 | climb: 20, stand: 144 |
| [scriptdata/waypoints/mp_osg_yemen_n_wp.csv](../scriptdata/waypoints/mp_osg_yemen_n_wp.csv) | Navigation graph for `mp_osg_yemen_n`; supplies positions and traversable links for bot routing. | 365 | climb: 11, stand: 354 |
| [scriptdata/waypoints/mp_osg_yemen_wp.csv](../scriptdata/waypoints/mp_osg_yemen_wp.csv) | Navigation graph for `mp_osg_yemen`; supplies positions and traversable links for bot routing. | 364 | climb: 11, stand: 353 |
| [scriptdata/waypoints/mp_oukhta_wp.csv](../scriptdata/waypoints/mp_oukhta_wp.csv) | Navigation graph for `mp_oukhta`; supplies positions and traversable links for bot routing. | 260 | stand: 260 |
| [scriptdata/waypoints/mp_outback_wp.csv](../scriptdata/waypoints/mp_outback_wp.csv) | Navigation graph for `mp_outback`; supplies positions and traversable links for bot routing. | 708 | stand: 708 |
| [scriptdata/waypoints/mp_outset_wp.csv](../scriptdata/waypoints/mp_outset_wp.csv) | Navigation graph for `mp_outset`; supplies positions and traversable links for bot routing. | 193 | stand: 193 |
| [scriptdata/waypoints/mp_outskirts_wp.csv](../scriptdata/waypoints/mp_outskirts_wp.csv) | Navigation graph for `mp_outskirts`; supplies positions and traversable links for bot routing. | 385 | climb: 2, stand: 383 |
| [scriptdata/waypoints/mp_overgrown_wp.csv](../scriptdata/waypoints/mp_overgrown_wp.csv) | Navigation graph for `mp_overgrown`; supplies positions and traversable links for bot routing. | 282 | climb: 4, stand: 278 |
| [scriptdata/waypoints/mp_overpass_wp.csv](../scriptdata/waypoints/mp_overpass_wp.csv) | Navigation graph for `mp_overpass`; supplies positions and traversable links for bot routing. | 423 | stand: 423 |
| [scriptdata/waypoints/mp_pic_wp.csv](../scriptdata/waypoints/mp_pic_wp.csv) | Navigation graph for `mp_pic`; supplies positions and traversable links for bot routing. | 329 | climb: 20, stand: 309 |
| [scriptdata/waypoints/mp_pillar_wp.csv](../scriptdata/waypoints/mp_pillar_wp.csv) | Navigation graph for `mp_pillar`; supplies positions and traversable links for bot routing. | 233 | undefined: 233 |
| [scriptdata/waypoints/mp_pipeline_wp.csv](../scriptdata/waypoints/mp_pipeline_wp.csv) | Navigation graph for `mp_pipeline`; supplies positions and traversable links for bot routing. | 317 | climb: 10, stand: 307 |
| [scriptdata/waypoints/mp_poolday_reunion_wp.csv](../scriptdata/waypoints/mp_poolday_reunion_wp.csv) | Navigation graph for `mp_poolday_reunion`; supplies positions and traversable links for bot routing. | 106 | stand: 106 |
| [scriptdata/waypoints/mp_poolday_wp.csv](../scriptdata/waypoints/mp_poolday_wp.csv) | Navigation graph for `mp_poolday`; supplies positions and traversable links for bot routing. | 363 | climb: 29, stand: 334 |
| [scriptdata/waypoints/mp_poolparty_wp.csv](../scriptdata/waypoints/mp_poolparty_wp.csv) | Navigation graph for `mp_poolparty`; supplies positions and traversable links for bot routing. | 197 | climb: 22, stand: 175 |
| [scriptdata/waypoints/mp_pow_rld_wp.csv](../scriptdata/waypoints/mp_pow_rld_wp.csv) | Navigation graph for `mp_pow_rld`; supplies positions and traversable links for bot routing. | 224 | climb: 8, stand: 216 |
| [scriptdata/waypoints/mp_printery_wp.csv](../scriptdata/waypoints/mp_printery_wp.csv) | Navigation graph for `mp_printery`; supplies positions and traversable links for bot routing. | 503 | climb: 11, stand: 492 |
| [scriptdata/waypoints/mp_pripyat_wp.csv](../scriptdata/waypoints/mp_pripyat_wp.csv) | Navigation graph for `mp_pripyat`; supplies positions and traversable links for bot routing. | 424 | climb: 22, stand: 402 |
| [scriptdata/waypoints/mp_prod_test_wp.csv](../scriptdata/waypoints/mp_prod_test_wp.csv) | Navigation graph for `mp_prod_test`; supplies positions and traversable links for bot routing. | 245 | climb: 11, crouch: 4, stand: 230 |
| [scriptdata/waypoints/mp_prunis_wp.csv](../scriptdata/waypoints/mp_prunis_wp.csv) | Navigation graph for `mp_prunis`; supplies positions and traversable links for bot routing. | 115 | undefined: 115 |
| [scriptdata/waypoints/mp_psycho_day_wp.csv](../scriptdata/waypoints/mp_psycho_day_wp.csv) | Navigation graph for `mp_psycho_day`; supplies positions and traversable links for bot routing. | 308 | climb: 2, stand: 306 |
| [scriptdata/waypoints/mp_psycho_wp.csv](../scriptdata/waypoints/mp_psycho_wp.csv) | Navigation graph for `mp_psycho`; supplies positions and traversable links for bot routing. | 308 | climb: 2, stand: 306 |
| [scriptdata/waypoints/mp_pval_wp.csv](../scriptdata/waypoints/mp_pval_wp.csv) | Navigation graph for `mp_pval`; supplies positions and traversable links for bot routing. | 625 | climb: 31, stand: 594 |
| [scriptdata/waypoints/mp_rasalem_wp.csv](../scriptdata/waypoints/mp_rasalem_wp.csv) | Navigation graph for `mp_rasalem`; supplies positions and traversable links for bot routing. | 320 | stand: 320 |
| [scriptdata/waypoints/mp_rats_sansa_room_wp.csv](../scriptdata/waypoints/mp_rats_sansa_room_wp.csv) | Navigation graph for `mp_rats_sansa_room`; supplies positions and traversable links for bot routing. | 329 | climb: 87, crouch: 4, stand: 238 |
| [scriptdata/waypoints/mp_rd_snow_wp.csv](../scriptdata/waypoints/mp_rd_snow_wp.csv) | Navigation graph for `mp_rd_snow`; supplies positions and traversable links for bot routing. | 558 | stand: 558 |
| [scriptdata/waypoints/mp_rd_xmas_wp.csv](../scriptdata/waypoints/mp_rd_xmas_wp.csv) | Navigation graph for `mp_rd_xmas`; supplies positions and traversable links for bot routing. | 558 | stand: 558 |
| [scriptdata/waypoints/mp_rebellion_wp.csv](../scriptdata/waypoints/mp_rebellion_wp.csv) | Navigation graph for `mp_rebellion`; supplies positions and traversable links for bot routing. | 386 | climb: 26, stand: 360 |
| [scriptdata/waypoints/mp_remagen_wp.csv](../scriptdata/waypoints/mp_remagen_wp.csv) | Navigation graph for `mp_remagen`; supplies positions and traversable links for bot routing. | 235 | stand: 235 |
| [scriptdata/waypoints/mp_remontna_wp.csv](../scriptdata/waypoints/mp_remontna_wp.csv) | Navigation graph for `mp_remontna`; supplies positions and traversable links for bot routing. | 281 | stand: 281 |
| [scriptdata/waypoints/mp_renan_bridge4_wp.csv](../scriptdata/waypoints/mp_renan_bridge4_wp.csv) | Navigation graph for `mp_renan_bridge4`; supplies positions and traversable links for bot routing. | 314 | climb: 12, stand: 302 |
| [scriptdata/waypoints/mp_rescue2014_wp.csv](../scriptdata/waypoints/mp_rescue2014_wp.csv) | Navigation graph for `mp_rescue2014`; supplies positions and traversable links for bot routing. | 433 | climb: 2, stand: 431 |
| [scriptdata/waypoints/mp_ricochet_wp.csv](../scriptdata/waypoints/mp_ricochet_wp.csv) | Navigation graph for `mp_ricochet`; supplies positions and traversable links for bot routing. | 326 | climb: 4, stand: 322 |
| [scriptdata/waypoints/mp_roof_n_wp.csv](../scriptdata/waypoints/mp_roof_n_wp.csv) | Navigation graph for `mp_roof_n`; supplies positions and traversable links for bot routing. | 361 | climb: 18, crouch: 21, stand: 322 |
| [scriptdata/waypoints/mp_roof_wp.csv](../scriptdata/waypoints/mp_roof_wp.csv) | Navigation graph for `mp_roof`; supplies positions and traversable links for bot routing. Contains malformed link token(s); see review notes. | 361 | climb: 18, crouch: 21, stand: 322 |
| [scriptdata/waypoints/mp_roundup_day_wp.csv](../scriptdata/waypoints/mp_roundup_day_wp.csv) | Navigation graph for `mp_roundup_day`; supplies positions and traversable links for bot routing. | 262 | undefined: 262 |
| [scriptdata/waypoints/mp_roundup_wp.csv](../scriptdata/waypoints/mp_roundup_wp.csv) | Navigation graph for `mp_roundup`; supplies positions and traversable links for bot routing. | 327 | climb: 2, stand: 325 |
| [scriptdata/waypoints/mp_ruins_wp.csv](../scriptdata/waypoints/mp_ruins_wp.csv) | Navigation graph for `mp_ruins`; supplies positions and traversable links for bot routing. | 275 | undefined: 275 |
| [scriptdata/waypoints/mp_rundown_wp.csv](../scriptdata/waypoints/mp_rundown_wp.csv) | Navigation graph for `mp_rundown`; supplies positions and traversable links for bot routing. | 520 | stand: 520 |
| [scriptdata/waypoints/mp_sbase_wp.csv](../scriptdata/waypoints/mp_sbase_wp.csv) | Navigation graph for `mp_sbase`; supplies positions and traversable links for bot routing. | 265 | climb: 6, stand: 259 |
| [scriptdata/waypoints/mp_school_wp.csv](../scriptdata/waypoints/mp_school_wp.csv) | Navigation graph for `mp_school`; supplies positions and traversable links for bot routing. | 317 | climb: 2, stand: 315 |
| [scriptdata/waypoints/mp_sconsegrad_wp.csv](../scriptdata/waypoints/mp_sconsegrad_wp.csv) | Navigation graph for `mp_sconsegrad`; supplies positions and traversable links for bot routing. | 753 | stand: 753 |
| [scriptdata/waypoints/mp_seatown_snow_wp.csv](../scriptdata/waypoints/mp_seatown_snow_wp.csv) | Navigation graph for `mp_seatown_snow`; supplies positions and traversable links for bot routing. | 284 | climb: 4, stand: 280 |
| [scriptdata/waypoints/mp_seatown_wp.csv](../scriptdata/waypoints/mp_seatown_wp.csv) | Navigation graph for `mp_seatown`; supplies positions and traversable links for bot routing. | 284 | climb: 4, stand: 280 |
| [scriptdata/waypoints/mp_selo_wp.csv](../scriptdata/waypoints/mp_selo_wp.csv) | Navigation graph for `mp_selo`; supplies positions and traversable links for bot routing. | 461 | climb: 8, crouch: 1, stand: 452 |
| [scriptdata/waypoints/mp_seti_wp.csv](../scriptdata/waypoints/mp_seti_wp.csv) | Navigation graph for `mp_seti`; supplies positions and traversable links for bot routing. | 613 | climb: 15, stand: 598 |
| [scriptdata/waypoints/mp_shipment44_wp.csv](../scriptdata/waypoints/mp_shipment44_wp.csv) | Navigation graph for `mp_shipment44`; supplies positions and traversable links for bot routing. | 151 | stand: 151 |
| [scriptdata/waypoints/mp_shipment_island_wp.csv](../scriptdata/waypoints/mp_shipment_island_wp.csv) | Navigation graph for `mp_shipment_island`; supplies positions and traversable links for bot routing. | 84 | climb: 8, stand: 76 |
| [scriptdata/waypoints/mp_shipment_wp.csv](../scriptdata/waypoints/mp_shipment_wp.csv) | Navigation graph for `mp_shipment`; supplies positions and traversable links for bot routing. | 75 | claymore: 19, crouch: 10, grenade: 9, stand: 37 |
| [scriptdata/waypoints/mp_showdown_wp.csv](../scriptdata/waypoints/mp_showdown_wp.csv) | Navigation graph for `mp_showdown`; supplies positions and traversable links for bot routing. | 79 | stand: 79 |
| [scriptdata/waypoints/mp_skidrow_wp.csv](../scriptdata/waypoints/mp_skidrow_wp.csv) | Navigation graph for `mp_skidrow`; supplies positions and traversable links for bot routing. | 172 | climb: 6, stand: 166 |
| [scriptdata/waypoints/mp_skyline_wp.csv](../scriptdata/waypoints/mp_skyline_wp.csv) | Navigation graph for `mp_skyline`; supplies positions and traversable links for bot routing. | 145 | climb: 8, stand: 137 |
| [scriptdata/waypoints/mp_slums_wp.csv](../scriptdata/waypoints/mp_slums_wp.csv) | Navigation graph for `mp_slums`; supplies positions and traversable links for bot routing. | 383 | stand: 383 |
| [scriptdata/waypoints/mp_stable_wp.csv](../scriptdata/waypoints/mp_stable_wp.csv) | Navigation graph for `mp_stable`; supplies positions and traversable links for bot routing. | 380 | climb: 18, crouch: 14, stand: 348 |
| [scriptdata/waypoints/mp_steamlab_wp.csv](../scriptdata/waypoints/mp_steamlab_wp.csv) | Navigation graph for `mp_steamlab`; supplies positions and traversable links for bot routing. | 346 | climb: 52, crouch: 21, stand: 273 |
| [scriptdata/waypoints/mp_storm_b_wp.csv](../scriptdata/waypoints/mp_storm_b_wp.csv) | Navigation graph for `mp_storm_b`; supplies positions and traversable links for bot routing. | 328 | stand: 328 |
| [scriptdata/waypoints/mp_stream_wp.csv](../scriptdata/waypoints/mp_stream_wp.csv) | Navigation graph for `mp_stream`; supplies positions and traversable links for bot routing. | 280 | stand: 280 |
| [scriptdata/waypoints/mp_strike_2_wp.csv](../scriptdata/waypoints/mp_strike_2_wp.csv) | Navigation graph for `mp_strike_2`; supplies positions and traversable links for bot routing. | 400 | stand: 400 |
| [scriptdata/waypoints/mp_strike_at_karkand_v2_wp.csv](../scriptdata/waypoints/mp_strike_at_karkand_v2_wp.csv) | Navigation graph for `mp_strike_at_karkand_v2`; supplies positions and traversable links for bot routing. | 138 | climb: 6, stand: 132 |
| [scriptdata/waypoints/mp_strike_wp.csv](../scriptdata/waypoints/mp_strike_wp.csv) | Navigation graph for `mp_strike`; supplies positions and traversable links for bot routing. | 325 | stand: 325 |
| [scriptdata/waypoints/mp_struggle_dawn_wp.csv](../scriptdata/waypoints/mp_struggle_dawn_wp.csv) | Navigation graph for `mp_struggle_dawn`; supplies positions and traversable links for bot routing. | 249 | stand: 249 |
| [scriptdata/waypoints/mp_struggle_night_wp.csv](../scriptdata/waypoints/mp_struggle_night_wp.csv) | Navigation graph for `mp_struggle_night`; supplies positions and traversable links for bot routing. | 249 | stand: 249 |
| [scriptdata/waypoints/mp_struggle_wp.csv](../scriptdata/waypoints/mp_struggle_wp.csv) | Navigation graph for `mp_struggle`; supplies positions and traversable links for bot routing. | 249 | stand: 249 |
| [scriptdata/waypoints/mp_summit_wp.csv](../scriptdata/waypoints/mp_summit_wp.csv) | Navigation graph for `mp_summit`; supplies positions and traversable links for bot routing. | 213 | stand: 213 |
| [scriptdata/waypoints/mp_tatooine_wp.csv](../scriptdata/waypoints/mp_tatooine_wp.csv) | Navigation graph for `mp_tatooine`; supplies positions and traversable links for bot routing. | 250 | climb: 8, stand: 242 |
| [scriptdata/waypoints/mp_tchernobyl_wp.csv](../scriptdata/waypoints/mp_tchernobyl_wp.csv) | Navigation graph for `mp_tchernobyl`; supplies positions and traversable links for bot routing. | 562 | stand: 562 |
| [scriptdata/waypoints/mp_theater_wp.csv](../scriptdata/waypoints/mp_theater_wp.csv) | Navigation graph for `mp_theater`; supplies positions and traversable links for bot routing. | 271 | stand: 271 |
| [scriptdata/waypoints/mp_tibongal_wp.csv](../scriptdata/waypoints/mp_tibongal_wp.csv) | Navigation graph for `mp_tibongal`; supplies positions and traversable links for bot routing. | 397 | climb: 15, crouch: 12, stand: 370 |
| [scriptdata/waypoints/mp_titanic_wp.csv](../scriptdata/waypoints/mp_titanic_wp.csv) | Navigation graph for `mp_titanic`; supplies positions and traversable links for bot routing. | 360 | climb: 8, stand: 352 |
| [scriptdata/waypoints/mp_toujane_snow_wp.csv](../scriptdata/waypoints/mp_toujane_snow_wp.csv) | Navigation graph for `mp_toujane_snow`; supplies positions and traversable links for bot routing. | 279 | climb: 10, stand: 269 |
| [scriptdata/waypoints/mp_toujane_v2_wp.csv](../scriptdata/waypoints/mp_toujane_v2_wp.csv) | Navigation graph for `mp_toujane_v2`; supplies positions and traversable links for bot routing. | 173 | climb: 12, stand: 161 |
| [scriptdata/waypoints/mp_toujane_wp.csv](../scriptdata/waypoints/mp_toujane_wp.csv) | Navigation graph for `mp_toujane`; supplies positions and traversable links for bot routing. | 279 | climb: 10, stand: 269 |
| [scriptdata/waypoints/mp_toybox4_wp.csv](../scriptdata/waypoints/mp_toybox4_wp.csv) | Navigation graph for `mp_toybox4`; supplies positions and traversable links for bot routing. | 631 | climb: 26, stand: 605 |
| [scriptdata/waypoints/mp_trainstation_wp.csv](../scriptdata/waypoints/mp_trainstation_wp.csv) | Navigation graph for `mp_trainstation`; supplies positions and traversable links for bot routing. | 364 | stand: 364 |
| [scriptdata/waypoints/mp_tundra_depot_wp.csv](../scriptdata/waypoints/mp_tundra_depot_wp.csv) | Navigation graph for `mp_tundra_depot`; supplies positions and traversable links for bot routing. | 401 | climb: 6, stand: 395 |
| [scriptdata/waypoints/mp_twin_2_wp.csv](../scriptdata/waypoints/mp_twin_2_wp.csv) | Navigation graph for `mp_twin_2`; supplies positions and traversable links for bot routing. | 445 | climb: 8, stand: 437 |
| [scriptdata/waypoints/mp_uber_wp.csv](../scriptdata/waypoints/mp_uber_wp.csv) | Navigation graph for `mp_uber`; supplies positions and traversable links for bot routing. | 137 | climb: 2, stand: 135 |
| [scriptdata/waypoints/mp_uplink_wp.csv](../scriptdata/waypoints/mp_uplink_wp.csv) | Navigation graph for `mp_uplink`; supplies positions and traversable links for bot routing. | 213 | stand: 213 |
| [scriptdata/waypoints/mp_uprise_wp.csv](../scriptdata/waypoints/mp_uprise_wp.csv) | Navigation graph for `mp_uprise`; supplies positions and traversable links for bot routing. | 164 | climb: 6, stand: 158 |
| [scriptdata/waypoints/mp_vacant_wp.csv](../scriptdata/waypoints/mp_vacant_wp.csv) | Navigation graph for `mp_vacant`; supplies positions and traversable links for bot routing. | 147 | stand: 147 |
| [scriptdata/waypoints/mp_vil_blops_wp.csv](../scriptdata/waypoints/mp_vil_blops_wp.csv) | Navigation graph for `mp_vil_blops`; supplies positions and traversable links for bot routing. | 190 | stand: 190 |
| [scriptdata/waypoints/mp_village_night_wp.csv](../scriptdata/waypoints/mp_village_night_wp.csv) | Navigation graph for `mp_village_night`; supplies positions and traversable links for bot routing. | 402 | climb: 4, crouch: 3, stand: 395 |
| [scriptdata/waypoints/mp_village_s_wp.csv](../scriptdata/waypoints/mp_village_s_wp.csv) | Navigation graph for `mp_village_s`; supplies positions and traversable links for bot routing. | 402 | climb: 4, crouch: 3, stand: 395 |
| [scriptdata/waypoints/mp_village_wp.csv](../scriptdata/waypoints/mp_village_wp.csv) | Navigation graph for `mp_village`; supplies positions and traversable links for bot routing. | 367 | climb: 2, crouch: 3, stand: 362 |
| [scriptdata/waypoints/mp_vukovar_n_wp.csv](../scriptdata/waypoints/mp_vukovar_n_wp.csv) | Navigation graph for `mp_vukovar_n`; supplies positions and traversable links for bot routing. | 351 | climb: 2, stand: 349 |
| [scriptdata/waypoints/mp_vukovar_wp.csv](../scriptdata/waypoints/mp_vukovar_wp.csv) | Navigation graph for `mp_vukovar`; supplies positions and traversable links for bot routing. | 584 | climb: 10, crouch: 14, stand: 560 |
| [scriptdata/waypoints/mp_warehouse_13_wp.csv](../scriptdata/waypoints/mp_warehouse_13_wp.csv) | Navigation graph for `mp_warehouse_13`; supplies positions and traversable links for bot routing. | 566 | climb: 39, crouch: 2, stand: 525 |
| [scriptdata/waypoints/mp_warehouse_wp.csv](../scriptdata/waypoints/mp_warehouse_wp.csv) | Navigation graph for `mp_warehouse`; supplies positions and traversable links for bot routing. | 76 | stand: 76 |
| [scriptdata/waypoints/mp_wasted_wp.csv](../scriptdata/waypoints/mp_wasted_wp.csv) | Navigation graph for `mp_wasted`; supplies positions and traversable links for bot routing. | 188 | climb: 6, crouch: 11, prone: 5, stand: 166 |
| [scriptdata/waypoints/mp_waw_castle_night_wp.csv](../scriptdata/waypoints/mp_waw_castle_night_wp.csv) | Navigation graph for `mp_waw_castle_night`; supplies positions and traversable links for bot routing. | 255 | climb: 2, stand: 253 |
| [scriptdata/waypoints/mp_waw_castle_wp.csv](../scriptdata/waypoints/mp_waw_castle_wp.csv) | Navigation graph for `mp_waw_castle`; supplies positions and traversable links for bot routing. | 255 | climb: 2, stand: 253 |
| [scriptdata/waypoints/mp_westoff_wp.csv](../scriptdata/waypoints/mp_westoff_wp.csv) | Navigation graph for `mp_westoff`; supplies positions and traversable links for bot routing. | 305 | climb: 2, stand: 303 |
| [scriptdata/waypoints/mp_winter_bakaara_wp.csv](../scriptdata/waypoints/mp_winter_bakaara_wp.csv) | Navigation graph for `mp_winter_bakaara`; supplies positions and traversable links for bot routing. | 171 | stand: 171 |
| [scriptdata/waypoints/mp_winter_rasalem_wp.csv](../scriptdata/waypoints/mp_winter_rasalem_wp.csv) | Navigation graph for `mp_winter_rasalem`; supplies positions and traversable links for bot routing. | 319 | stand: 319 |
| [scriptdata/waypoints/mp_woodrr_wp.csv](../scriptdata/waypoints/mp_woodrr_wp.csv) | Navigation graph for `mp_woodrr`; supplies positions and traversable links for bot routing. | 198 | climb: 2, crouch: 2, stand: 194 |
| [scriptdata/waypoints/mp_zaseda_wp.csv](../scriptdata/waypoints/mp_zaseda_wp.csv) | Navigation graph for `mp_zaseda`; supplies positions and traversable links for bot routing. | 367 | climb: 15, stand: 352 |
| [scriptdata/waypoints/mp_zavod_wp.csv](../scriptdata/waypoints/mp_zavod_wp.csv) | Navigation graph for `mp_zavod`; supplies positions and traversable links for bot routing. | 697 | climb: 20, stand: 677 |

## Review notes and validation limits

1. **Four malformed link tokens occur in three CSVs.** The CSV parser expects integer child indexes, but these records contain decimal values in the link column:

   | File | Physical line | Waypoint index | Malformed link token |
   | --- | ---: | ---: | --- |
   | [scriptdata/waypoints/mp_fallout_wp.csv](../scriptdata/waypoints/mp_fallout_wp.csv#L491) | 491 | 489 | `-177.924` |
   | [scriptdata/waypoints/mp_mtl_the_rock_wp.csv](../scriptdata/waypoints/mp_mtl_the_rock_wp.csv#L576) | 576 | 574 | `75.7727` |
   | [scriptdata/waypoints/mp_roof_wp.csv](../scriptdata/waypoints/mp_roof_wp.csv#L318) | 318 | 316 | `175.995` |
   | [scriptdata/waypoints/mp_roof_wp.csv](../scriptdata/waypoints/mp_roof_wp.csv#L350) | 350 | 348 | `9.59657` |

   These are static data anomalies. The runtime converts link tokens with `int(...)`; actual conversion behavior and gameplay impact were not tested. No corrections were made. All checked positions had three numeric coordinates, and all link tokens that parsed as integers were within their files' node-index ranges.

2. **Navigation has fallback hooks despite the README's CSV-only changelog wording.** `_bot_utility.gsc::load_waypoints` first reads CSV, then calls the empty custom-map hook, then tries Frontlines `flwp_*` DVAR data. The supplied usable graph collection is the 360 CSV files.

3. **The stock-mod compatibility claim has integration limits.** The README says it edits no stock `.gsc` files and claims broad compatibility. This checkout supplies a stock-path `_callbacksetup.gsx` bootstrap, wraps shared damage/death callbacks, changes loadout tables and some gametype object handlers, and relies on stock game APIs. Compatibility with arbitrary other mods was not established by this review.

4. **Debug mode activates an editor with gameplay side effects.** A nonzero `bots_main_debug` at editor initialization enables developer editing behavior, disables bot/menu operation through DVAR changes, and can restart the map. This matters when interpreting the README's debug-event descriptions.

5. **Validation was static.** Documentation, all source modules and repository configuration were inspected, every CSV was parsed for inventory counts and basic coordinate/link shape, and artwork was identified by file format and its documentation references. The Paint.NET artwork was not opened in an editor. The existing CI workflow parses scripts with an IW5 target; it does not demonstrate CoD4x gameplay execution. No game server was started, no runtime/CI test was run, and graph connectivity, reciprocity and in-map reachability were not exhaustively validated.

The review did not change game code or data. This Markdown report was subsequently added to the repository as `docs/repository-review.md`.
