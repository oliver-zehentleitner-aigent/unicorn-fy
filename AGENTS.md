# AGENTS.md — UnicornFy

## Planning & Backlog

Open development tasks and decisions are tracked in **[TASKS.md](TASKS.md)**.

---

## Project Overview

Python SDK (MIT License) to convert raw WebSocket stream data from Binance exchange endpoints into well-formed Python dictionaries ("unicorn-fied" data).

**Current Version:** 0.16.1.dev  
**Python Compatibility:** 3.8 – 3.13  
**Author:** Oliver Zehentleitner  
**PyPI:** `unicorn-fy`  
**Part of:** [UNICORN Binance Suite](https://github.com/oliver-zehentleitner/unicorn-binance-suite)

---

## Directory Structure

```
unicorn_fy/
    unicorn_fy.py          # Main class UnicornFy (~1 file, all conversion logic)
    __init__.py            # Exports UnicornFy

unittest_unicorn_fy.py     # Unit tests (run in CI)
dev/                       # Local dev/integration tests — NOT run in CI
examples/                  # Usage examples
docs/                      # Pre-built HTML documentation (Sphinx)
dev/sphinx/                # Sphinx source for rebuilding docs
```

---

## Supported Exchanges

| Exchange Method | Exchange |
|---|---|
| `binance_com_websocket()` | Binance.com + Testnet |
| `binance_com_margin_websocket()` | Binance.com Margin |
| `binance_com_isolated_margin_websocket()` | Binance.com Isolated Margin |
| `binance_com_futures_websocket()` | Binance.com Futures |
| `binance_com_coin_futures_websocket()` | Binance.com Coin-M Futures |
| `binance_us_websocket()` | Binance.US |
| `trbinance_com_websocket()` | trBinance.com |
| `binance_org_websocket()` | Binance.org (DEX, pass-through only) |

---

## Dependencies

Managed in `requirements.txt`, `setup.py`, and `pyproject.toml` — **all three must be kept in sync manually**:

- `ujson` — fast JSON parsing (primary JSON lib in source)
- `requests` — HTTP for version checks
- `Cython` — C extension compilation (release builds only)

---

## Running Tests

```bash
# Unit tests with coverage (this is what CI runs)
coverage run --source unicorn_fy unittest_unicorn_fy.py

# Unit tests without coverage
python unittest_unicorn_fy.py
```

Tests in `dev/` are local integration tests — they are **not run in CI**.

**Coverage config:** `.coveragerc`

---

## Build & Packaging

Development and testing use **plain Python** — no Cython compilation needed during development.

Cython compilation only happens for **release builds**:

```bash
python setup.py bdist_wheel
```

**Version bump** — done **manually** before each release. Update the version string in all three locations:
1. `setup.py`
2. `pyproject.toml`
3. `unicorn_fy/unicorn_fy.py` (`__version__`)

**CI/CD:** GitHub Actions in `.github/workflows/`
- `unit-tests.yml` — Python 3.8–3.13 on Ubuntu, Codecov upload
- `build_wheels.yml` — Manual trigger, builds wheels for Linux/macOS/Windows, PyPI release
- `codeql-analysis.yml` — Security scanning
- `build_conda.yml` — Conda package build

---

## Code Conventions

- **File header:** Always include the full MIT license block with author/copyright (2019-2025)
- **Encoding:** UTF-8, UNIX line endings
- **Logging:** `logging.getLogger("unicorn_fy")`
- **JSON:** `ujson` is used as `import ujson as json` — do not switch to stdlib `json`
- **Cython:** Core module compiles to C extension for releases — no `#cython:` directives needed in source
- **Versioning:** Keep version in sync across `setup.py`, `pyproject.toml`, and `unicorn_fy/unicorn_fy.py` manually

---

## Key Class

| Class | File | Purpose |
|---|---|---|
| `UnicornFy` | `unicorn_fy/unicorn_fy.py` | All conversion logic; static methods per exchange |

---

## Usage Pattern (Quick Reference)

```python
import unicorn_fy

ufy = unicorn_fy.UnicornFy()

# Convert raw Binance.com stream data
result = ufy.binance_com_websocket(raw_json_string)

# Result contains normalized keys + 'unicorn_fied' metadata field
# {'stream_type': '...', 'event_type': '...', ..., 'unicorn_fied': ['binance', '0.16.1']}
```

---

## Integration with UNICORN Binance WebSocket API

UnicornFy is used automatically by `unicorn-binance-websocket-api` when:
- `output_default="UnicornFy"` is set on `BinanceWebSocketApiManager()`
- `output="UnicornFy"` is set on individual `create_stream()` calls
