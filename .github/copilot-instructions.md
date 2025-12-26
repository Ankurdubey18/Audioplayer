<!--
Purpose: Guidance for AI coding agents working on this repository.
Keep suggestions targeted to the actual code patterns in `Audioplayer-main`.
-->
# Copilot instructions — Audioplayer

This small desktop app is a single-file Tkinter GUI audio player. Key files:

- `Audioplayer-main/main.py` — single-entry UI + playback logic.
- `Audioplayer-main/music/` — place MP3 files here; code looks for `*.mp3`.
- `Audioplayer-main/README.md` — project intent and features.

Quick summary
- UI: built with `tkinter` and runs `canvas.mainloop()` (blocking).
- Playback: uses `pygame.mixer` (`mixer.init()` and `mixer.music.load/play/pause/stop`).
- Visualizer: uses `librosa` to load audio and `matplotlib` (`FigureCanvasTkAgg`) to render embedded plots.
- Assets: optional images loaded via `_try_load_image()` (falls back to text buttons).

How to run (observed from repo)

1. Activate the project virtualenv if present (`.venv` exists). On Windows PowerShell:

   - `.\.venv\Scripts\Activate.ps1`

2. Install runtime deps (project imports):

   - `pip install pygame numpy matplotlib librosa`

3. Run the GUI:

   - `python Audioplayer-main/main.py`

Important code patterns and gotchas for edits

- Path handling: `rootpath = os.path.join(os.path.dirname(__file__), "music")`, but the code concatenates paths with `rootpath + "\\" + filename` (Windows-style). If you change path code, update all concatenations or prefer `os.path.join()` consistently.
- List selection: The UI relies on `listBox.get("anchor")` and `listBox.curselection()` for the currently selected song — preserve these calls when modifying playback flow.
- Visualizer: `update_visualizer()` calls `librosa.load()` each update (on the main thread). Avoid blocking the Tk mainloop when altering this — prefer preloading or offloading heavy work to a worker thread if you refactor.
- Images: `_try_load_image()` returns `None` on failure; code already safely displays text if images missing. Keep this fallback when changing button creation.

Integration points

- `pygame.mixer` — central playback API; do not replace with another engine unless updating all playback commands.
- `librosa` — used for waveform extraction; changes to sample handling must maintain the `y, sr` shape expected by the plotting code.
- `matplotlib.backends.backend_tkagg.FigureCanvasTkAgg` — the visualizer is embedded into the Tk window; keep redraw calls (`canvas_visualizer.draw()`) inside the GUI thread.

Editing recommendations (concrete examples)

- If you need cross-platform path fixes, replace occurrences like `rootpath + "\\" + next_song_name` with `os.path.join(rootpath, next_song_name)` everywhere.
- To make `update_visualizer()` non-blocking: preload audio arrays once (e.g., on file load) and compute slices in the GUI thread instead of calling `librosa.load()` repeatedly.
- When renaming functions used as command handlers (e.g., `select`, `play_next`, `play_prev`), update all `command=` bindings on the buttons in `main.py`.

No CI/tests detected

There are no test files or CI configuration in the repo. The app is a user-facing GUI; prefer small refactors (extract playback logic to helpers) to enable unit testing.

If adding files

- Keep UI wiring in `main.py` thin. Put business logic (file scanning, playback control, audio processing) in helpers or a small module to improve testability.

When merging

- If an existing `.github/copilot-instructions.md` exists, preserve any explicit author guidelines; otherwise prefer the concise format above and call out any conflicts in a short PR description.

Questions for the maintainer

- Do you want cross-platform path fixes applied automatically? (Windows-specific backslashes are currently used.)
- Should I extract playback/visualizer logic into modules for testability now, or leave as a focused, minimal change?
