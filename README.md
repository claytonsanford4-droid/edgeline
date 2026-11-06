# Edgeline (Claude Code edition)

Your Edgeline site, moved out of claude.ai so Claude Code can update it on a schedule.

## 1. Put it online (GitHub Pages, free)
1. Create a new **private or public** GitHub repo (e.g. `edgeline`) and upload/push this whole folder.
2. In the repo: **Settings → Pages → Build and deployment → Deploy from a branch → `main` / root → Save.**
3. After a minute your site is live at `https://<your-username>.github.io/edgeline/`. Bookmark it on your phone.
   (Private repos need a paid GitHub plan for Pages; otherwise make the repo public — the data is just your picks.)

To preview locally instead: `cd edgeline && python3 -m http.server` then open http://localhost:8000.
(Opening `index.html` directly as a file won't load the data.)

## 2. Daily automatic updates
In a terminal inside the repo, start Claude Code: `claude`
- Test once: `/daily-update`
- Schedule it: `/schedule` → daily, around 2 PM Eastern, prompt: `Run /daily-update`, and pick this repo.
  (Or at claude.ai/code/routines → New routine.) Routines run in the cloud, so your computer can be off.
Each run grades yesterday, loads today's slate and prices, runs the engine and review, and pushes —
the site picks up the new data within ~5 minutes (it re-checks the data file on its own).

## 3. Logging your own bets
- Easiest: in Claude Code, `/log-bet Penguins ML -110 1u`.
- Or add them on the site. Those save in that browser; to sync them, Settings → **Download my changes**,
  drop the file in the repo folder and run `/import-changes` in Claude Code.

## Notes
- All prices are restricted to Ontario-licensed sportsbooks (see CLAUDE.md).
- Team badges and player sprites are original pixel art, not official logos or photos.
- The claude.ai version of Edgeline still exists but will stop updating once you switch.
