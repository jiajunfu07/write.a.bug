# write.a.bug

This repository hosts my personal documentation site (Sphinx). It contains tutorials and notes, including ROS 2 guides under `docs/source/ROS2`.

Quick start — build the documentation locally

1. Create and activate a Python virtual environment (recommended):

```bash
python3 -m venv .venv
source .venv/bin/activate
```

2. Install documentation requirements and build HTML:

```bash
pip install -r docs/requirements.txt
cd docs
make html
```

3. Serve the generated site locally:

```bash
python3 -m http.server --directory build/html 8000
# then open http://localhost:8000
```

Notes
- Chinese translations for the ROS2 tutorials have been added as separate files next to the originals with the `.zh.md` suffix. See `docs/source/ROS2/*.zh.md`.
- The Sphinx source is in `docs/source` and the built site is `docs/build/html`.

If you'd like, I can also commit these changes or open a pull request — tell me how you want to proceed.