# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Primary Project: Ashtavakra Sutras Desktop Widget

The main codebase lives in `mywork/` — a cross-platform desktop widget (Python + CustomTkinter) that displays verses from the Ashtavakra Gita.

## Setup & Running

```bash
cd mywork
pip install -r requirements.txt

# Run (Windows)
run.bat
# Run (Linux/macOS)
./run.sh
# Run directly
python -m ashtavakra_widget.main
```

## Build Standalone Executable (Windows)

```bash
pip install pyinstaller
pyinstaller build.spec
# Output: dist/AshtavakraWidget.exe
```

## Data Scripts

```bash
python scripts/build_sutras.py      # Rebuild data/sutras.json from source
python scripts/validate_sutras.py   # Validate verse data integrity
```

## Architecture

The app is wired in `main.py`, which owns the lifecycle and connects four components via callbacks:

- **`app.py` — `AshtavakraWidget(ctk.CTk)`**: Frameless window (`overrideredirect=True`), drag-to-move via title bar. Calls `save_state()` on every verse change and window move. Close/minimize both hide to tray rather than quit.
- **`tray.py` — `TrayManager`**: System tray icon (pystray). Communicates back to `app` via the `on_show`, `on_toggle_always_on_top`, `on_quit` callbacks passed in from `main.py`.
- **`state.py` — `AppState`**: Dataclass persisted as JSON. Platform config paths: Windows `%APPDATA%\AshtavakraWidget\state.json`, macOS `~/Library/Application Support/AshtavakraWidget/state.json`, Linux `~/.config/ashtavakra-widget/state.json`.
- **`sutra.py`**: `Sutra` frozen dataclass + `load_sutras()` from `data/sutras.json` (298 verses). `daily_sutra_index()` is deterministic by date (`int(YYYYMMDD) % len(sutras)`). When frozen (PyInstaller), data is loaded from `sys._MEIPASS`.

Styles/layout constants and font helpers are isolated in `styles.py`.

## Ollama Modelfiles

Several `Modelfile-*` files in the home directory define custom Ollama models:

| File | Base | Role |
|------|------|------|
| `Modelfile` | qwen2.5:7b | General (GPU-accelerated) |
| `Modelfile-analyst` | phi3.5 | Data analysis, low temperature |
| `Modelfile-business` | phi3.5 | Formal business writing |
| `Modelfile-creative` | phi3.5 | Creative/presentation writing |
| `Modelfile-memory` | phi3.5 | Context retention via mirostat |

Create a model with: `ollama create <name> -f Modelfile-<name>`
