# Qlib Local Source Setup Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the source-installed qlib checkout usable for local development by installing `.[dev]`, preparing the default market data directory, running a minimal official demo, and confirming that the current LibreSSL warning does not block the core workflow.

**Architecture:** Keep all Python dependencies isolated in `qlib/.venv`, keep the source tree editable-installed from the current checkout, and use qlib’s default data path `~/.qlib/qlib_data/cn_data` so official examples run without patching paths. Validate the setup with qlib’s own entry points: the data download path from `scripts/get_data.py`, and the official minimal workflow in `examples/workflow_by_code.py`.

**Tech Stack:** Python 3.9 virtualenv, pip editable install, qlib source checkout, qlib CLI/data utilities, official example scripts, shell verification commands.

---

## File and Directory Map

**Existing files used as references:**
- `README.md` — source install and default data directory guidance.
- `docs/developer/code_standard_and_dev_guide.rst` — recommends `pip install -e ".[dev]"` for development.
- `examples/workflow_by_code.py` — minimal official workflow-by-code demo using `~/.qlib/qlib_data/cn_data`.
- `scripts/get_data.py` — official data download entry point.
- `examples/benchmarks/LightGBM/workflow_config_lightgbm_Alpha158.yaml` — fallback YAML-based benchmark workflow if needed.

**Directories that will be created or populated:**
- `~/.qlib/qlib_data/cn_data/` — qlib default daily-frequency CN market dataset.
- `~/.qlib/qlib_data/cn_data/.setup-marker.txt` — optional local marker written only if the workflow needs a human-readable note that this directory was prepared for this machine.

**Environment already in place and reused:**
- `/Users/zyb/Desktop/vibe coding/Financial Management/qlib/.venv/` — isolated virtual environment for this checkout.

---

### Task 1: Install qlib development dependencies into the isolated environment

**Files:**
- Reference: `docs/developer/code_standard_and_dev_guide.rst:58-63`
- Reuse: `/Users/zyb/Desktop/vibe coding/Financial Management/qlib/.venv/`

- [ ] **Step 1: Confirm the editable source install is active before changing dependencies**

Run:
```bash
cd '/Users/zyb/Desktop/vibe coding/Financial Management/qlib' && \
'.venv/bin/python' - <<'PY'
import qlib, pathlib
print(pathlib.Path(qlib.__file__).resolve())
print(getattr(qlib, '__version__', 'unknown'))
PY
```

Expected:
- The printed path is `/Users/zyb/Desktop/vibe coding/Financial Management/qlib/qlib/__init__.py`
- The command exits with code 0

- [ ] **Step 2: Install the development extras recommended by qlib**

Run:
```bash
cd '/Users/zyb/Desktop/vibe coding/Financial Management/qlib' && \
'.venv/bin/python' -m pip install -e ".[dev]"
```

Expected:
- pip finishes with exit code 0
- Output includes `Successfully installed` or `Requirement already satisfied` lines for dev dependencies such as `pytest`

- [ ] **Step 3: Verify the expected developer tools are available in this environment**

Run:
```bash
cd '/Users/zyb/Desktop/vibe coding/Financial Management/qlib' && \
'.venv/bin/python' - <<'PY'
import importlib.util
mods = ['pytest', 'qlib']
for name in mods:
    print(name, importlib.util.find_spec(name) is not None)
PY
```

Expected:
- Printed lines include `pytest True` and `qlib True`
- The command exits with code 0

- [ ] **Step 4: Record the exact installed qlib package metadata for later troubleshooting**

Run:
```bash
'/Users/zyb/Desktop/vibe coding/Financial Management/qlib/.venv/bin/python' -m pip show pyqlib
```

Expected:
- Output contains `Name: pyqlib`
- Output contains `Editable project location: /Users/zyb/Desktop/vibe coding/Financial Management/qlib`

---

### Task 2: Prepare the default qlib data directory

**Files:**
- Reference: `README.md:211-245`
- Reference: `scripts/get_data.py:1-8`
- Create/Populate: `~/.qlib/qlib_data/cn_data/`

- [ ] **Step 1: Create the default directory path used by qlib examples**

Run:
```bash
mkdir -p ~/.qlib/qlib_data/cn_data
```

Expected:
- The command exits with code 0
- The directory `~/.qlib/qlib_data/cn_data` exists afterwards

- [ ] **Step 2: Try the README-recommended community dataset tarball first**

Run:
```bash
cd ~/.qlib/qlib_data && \
curl -L --fail --output qlib_bin.tar.gz \
https://github.com/chenditc/investment_data/releases/latest/download/qlib_bin.tar.gz && \
tar -zxvf qlib_bin.tar.gz -C ~/.qlib/qlib_data/cn_data --strip-components=1 && \
rm -f qlib_bin.tar.gz
```

Expected:
- `curl` exits with code 0
- `tar` extracts files into `~/.qlib/qlib_data/cn_data`
- `qlib_bin.tar.gz` is deleted after extraction

- [ ] **Step 3: If the community tarball fails, fall back to qlib’s built-in data fetcher**

Run only if Step 2 fails:
```bash
cd '/Users/zyb/Desktop/vibe coding/Financial Management/qlib' && \
'.venv/bin/python' scripts/get_data.py qlib_data \
  --target_dir ~/.qlib/qlib_data/cn_data \
  --region cn
```

Expected:
- The command exits with code 0
- The directory fills with qlib dataset content instead of remaining empty

- [ ] **Step 4: Verify the data directory is populated enough for qlib initialization**

Run:
```bash
python3 - <<'PY'
from pathlib import Path
p = Path.home() / '.qlib' / 'qlib_data' / 'cn_data'
print('exists=', p.exists())
items = sorted(x.name for x in p.iterdir()) if p.exists() else []
print('count=', len(items))
print('sample=', items[:10])
PY
```

Expected:
- `exists= True`
- `count=` is greater than 0
- `sample=` shows dataset content rather than an empty list

---

### Task 3: Run the minimal official source-based demo

**Files:**
- Run: `examples/workflow_by_code.py`
- Reference: `examples/workflow_by_code.py:19-85`
- Fallback Reference: `examples/benchmarks/LightGBM/workflow_config_lightgbm_Alpha158.yaml`

- [ ] **Step 1: Run the official workflow-by-code demo from the source checkout**

Run:
```bash
cd '/Users/zyb/Desktop/vibe coding/Financial Management/qlib' && \
'.venv/bin/python' examples/workflow_by_code.py
```

Expected:
- The command exits with code 0
- Output shows qlib initialization and workflow progress
- The run creates an `mlruns/` directory or recorder artifacts in the current working directory

- [ ] **Step 2: If the workflow-by-code script fails because a dependency is still missing, inspect the exact failure before changing anything**

Run only if Step 1 fails:
```bash
cd '/Users/zyb/Desktop/vibe coding/Financial Management/qlib' && \
'.venv/bin/python' examples/workflow_by_code.py 2>&1 | tee /tmp/qlib-workflow-by-code.log
```

Expected:
- `/tmp/qlib-workflow-by-code.log` is created
- The failing module, command, or stack trace is visible for root-cause analysis

- [ ] **Step 3: Use the YAML-based LightGBM benchmark as the official fallback path if the code-based script is blocked by a specific issue unrelated to core installation**

Run only if the workflow-by-code route is blocked but the environment is otherwise intact:
```bash
cd '/Users/zyb/Desktop/vibe coding/Financial Management/qlib/examples/benchmarks/LightGBM' && \
'../../.venv/bin/qrun' workflow_config_lightgbm_Alpha158.yaml
```

Expected:
- The command exits with code 0
- qlib runs the benchmark workflow against `~/.qlib/qlib_data/cn_data`

- [ ] **Step 4: Verify that the demo produced artifacts rather than only printing startup text**

Run:
```bash
python3 - <<'PY'
from pathlib import Path
for path in [Path('/Users/zyb/Desktop/vibe coding/Financial Management/qlib/mlruns'), Path('/Users/zyb/Desktop/vibe coding/Financial Management/qlib/examples/benchmarks/LightGBM/mlruns')]:
    print(path, path.exists())
PY
```

Expected:
- At least one printed path ends with `True`

---

### Task 4: Verify the LibreSSL warning is non-blocking under the chosen A-strategy

**Files:**
- Observe runtime output from `qrun` and example scripts
- No source file changes in this task

- [ ] **Step 1: Capture the warning with a minimal command that still succeeds**

Run:
```bash
'/Users/zyb/Desktop/vibe coding/Financial Management/qlib/.venv/bin/qrun' --help
```

Expected:
- Output may include `NotOpenSSLWarning`
- Help text still prints successfully
- The command exits with code 0

- [ ] **Step 2: Confirm that the same Python environment can still perform qlib import and one network-related operation if needed by the data path already used**

Run:
```bash
cd '/Users/zyb/Desktop/vibe coding/Financial Management/qlib' && \
'.venv/bin/python' - <<'PY'
import qlib
import ssl
print('qlib_ok=True')
print('ssl_library=', ssl.OPENSSL_VERSION)
PY
```

Expected:
- Output includes `qlib_ok=True`
- Output prints the SSL library string
- The command exits with code 0

- [ ] **Step 3: Decide and document the impact based on evidence from Tasks 2 and 3**

Decision rule:
```text
If data preparation and demo execution both succeeded, classify the warning as non-blocking for current local use.
If either operation failed with an SSL/urllib3 stack trace, stop and propose a Python interpreter replacement plan instead of guessing.
```

Expected:
- The final report explicitly says either `non-blocking warning` or `blocking SSL issue requiring environment replacement`

---

### Task 5: Produce the final operator handoff commands

**Files:**
- No repository source changes required
- Report commands directly to the user after verification

- [ ] **Step 1: Verify the final reusable commands work in the prepared environment**

Run:
```bash
source '/Users/zyb/Desktop/vibe coding/Financial Management/qlib/.venv/bin/activate' && \
cd '/Users/zyb/Desktop/vibe coding/Financial Management/qlib' && \
python -c "import qlib; print(qlib.__file__); print(qlib.__version__)" && \
qrun --help
```

Expected:
- The import path points to the local checkout
- Version prints successfully
- `qrun --help` succeeds even if it emits the LibreSSL warning

- [ ] **Step 2: Hand the user the exact day-to-day commands**

Provide this command block:
```bash
# activate env
source '/Users/zyb/Desktop/vibe coding/Financial Management/qlib/.venv/bin/activate'

# enter repo
cd '/Users/zyb/Desktop/vibe coding/Financial Management/qlib'

# verify source install
python -c "import qlib; print(qlib.__file__); print(qlib.__version__)"

# run minimal official demo
python examples/workflow_by_code.py

# run qlib CLI help
qrun --help
```

Expected:
- The report includes the actual data directory used and whether `mlruns/` was created

---

## Self-Review

- **Spec coverage:** This plan covers all four approved items: install `.[dev]`, prepare the default qlib data directory, run the minimal official demo, and validate the LibreSSL warning under strategy A.
- **Placeholder scan:** No `TODO`, `TBD`, or vague “appropriate handling” language remains; each task includes exact commands and expected outcomes.
- **Type consistency:** All paths, command names, and target directories are consistent with the current checkout and qlib’s official examples.
