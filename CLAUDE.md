# Copperwise

World of Warcraft retail addon that tracks gold income and expense per character, plus a
web viewer for the saved data. Owned by NeonRat85, MIT licensed.

Copperwise began as a fork of **GoldLedger** by tum24 and was rebranded at 1.0.0. The
first commit in this repository is the unmodified GoldLedger 2.4.4. Credit to tum24 must
stay in `LICENSE`, the README's Credits section and the `## X-Credits` line in the .toc.

- GitHub: https://github.com/NeonRat85/Copperwise
- CurseForge: project ID 1701196 (`## X-Curse-Project-ID`)
- Viewer: https://neonrat85.github.io/Copperwise/ — GitHub Pages serves `docs/index.html`
- Commits use `NeonRat85 <22443399+NeonRat85@users.noreply.github.com>`; there is no
  global git identity on this machine, so set it per clone.

## Working copy vs installed copy

Develop in a clone, not in `World of Warcraft/_retail_/Interface/AddOns/Copperwise`. The
CurseForge app overwrote that folder with the packaged release once and removed its `.git`
along with `tests/`, `docs/` and `.github/`. Either keep the AddOns copy as a plain install
and copy builds into it, or re-clone there and never let the CurseForge app manage it.

## Layout

| Path | What it is |
| --- | --- |
| `Core.lua` | Module system, event bus, slash commands, feature registry |
| `Data.lua` | SavedVariables (`CopperwiseDB`), entries, daily/monthly totals, goal, GoldLedger import |
| `Tracker.lua` | `PLAYER_MONEY` handling, source detection, bank transfers, vendor sale naming |
| `Locale.lua` | Every UI string (English) |
| `UI/`, `Themes/` | Main window, settings, helpers, colours |
| `Modules/<Feature>/` | Self-contained features; deleting a folder removes that feature |
| `Libs/` | LibStub, CallbackHandler-1.0, LibDataBroker-1.1, LibDBIcon-1.0 — unmodified, own licences |
| `tests/` | Harnesses run locally and in CI |
| `docs/index.html` | The viewer page (not shipped in the addon zip) |
| `.github/workflows/` | `ci.yml` (every push) and `release.yml` (version tags) |

## Rules that matter

- **English only.** The Russian translation and the language picker were deliberately
  removed (the Ukraine war). Do not reintroduce `ruRU` tables or a language selector. For
  anything locale-dependent, use Blizzard's own localised globals — AH sale mail is matched
  with `AUCTION_SOLD_MAIL_SUBJECT`, never hard-coded words.
- **Never assign a Blizzard global**, not even to its current value. `SlashCmdList =
  SlashCmdList or {}` tainted the global, which made every slash command run tainted and
  got `/pvp` blocked with ADDON_ACTION_FORBIDDEN blamed on the addon. `tests/globals.lua`
  enforces this; only Copperwise's own names may be assigned.
- **Midnight (12.x) addon restrictions** are wider than combat. Before touching anything
  protected, check every `Enum.AddOnRestrictionType` through
  `C_RestrictedActions.IsAddOnRestrictionActive`, not just `InCombatLockdown()`.
- **Minimap button is LibDBIcon.** Do not hand-roll one; a hand-built button lost its
  layer under the minimap and swallowed hovers. State lives in `settings.minimap`.
- **Warband bank** deposits and withdrawals are `type = "transfer"` entries: shown in
  transaction lists, excluded from income/expense totals, charts and session stats. The
  balance counts towards the savings goal. Guild bank gold stays income/expense.
- **Vendor sale item names** come from matching the gold change to a new merchant buyback
  entry by exact price. Gold and buyback updates arrive in either order, several items can
  share one money change (Sell Junk), and unmatched sales expire after 5 seconds.
- **GoldLedger import** runs once, only while Copperwise has no data of its own.
- Line endings are LF via `.gitattributes`; CRLF warnings when committing are expected.

## Tests

LuaJIT implements Lua 5.1, like the WoW client, and is on PATH as `luajit` (a `lua` /
`luac` shim exists too). The harnesses load the real addon files against a stubbed WoW API.

```bash
luajit tests/harness.lua .   # bank transfers, vendor sales, GoldLedger import, aggregation
luajit tests/minimap.lua .   # LibDBIcon registration, tooltip, visibility, settings migration
luajit tests/globals.lua .   # no global assignments the addon doesn't own
```

CI also syntax-checks every Lua file and verifies every path listed in `Copperwise.toc`
exists. When fixing a bug, check the new test fails without the fix.

API reference: https://github.com/Gethe/wow-ui-source (beta branch) for Blizzard's current
Lua and the generated API documentation.

## Releasing

1. Bump `## Version` in `Copperwise.toc` and add a `CHANGELOG.md` section.
2. Commit, then tag and push:

```bash
git tag -a v1.0.2 -m "Copperwise 1.0.2" && git push origin main v1.0.2
```

`release.yml` runs the tests, checks the tag matches the .toc version, packages with
BigWigsMods/packager and uploads to CurseForge (repository secret `CF_API_KEY`) and GitHub
Releases. `docs/`, `tests/` and `.github/` are excluded from the archive via `.pkgmeta` and
`export-ignore`.
