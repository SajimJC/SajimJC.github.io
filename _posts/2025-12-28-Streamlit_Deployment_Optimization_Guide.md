---
title: "Streamlit Prototype Deployment & Architecture Guide"
date: "2025-12-28"
permalink: "/posts/2025/12/Streamlit_Deployment_Guidance/"
tags:
  - Streamlit
  - AI-Visualization
---

# Streamlit Deployment & Optimization Guide (Version 20251228)

This guide explains how to configure, organize, and optimize Streamlit apps for fast prototyping. It focuses on directory layout, static assets, layered architecture, safe configuration, and deployment notes.

## 1. Directory Structure & Multipage Basics

### 1.1 Recommended Layout
Organize your project to enable native multipage navigation and clean modularization:

```text
Project/
├── .streamlit/             # Project-level configs
│   ├── config.toml         # Base config (port, theme, etc.)
│   └── secrets.toml        # Secrets & credentials (never commit)
└── visualization/          # App directory
    ├── app.py              # Main entry
    ├── components/         # Reusable UI fragments (sidebar, navigation, filters)
    └── pages/              # Multipage directory (auto sidebar)
```

Start from the project root: `streamlit run visualization/app.py`

### 1.2 Is `pages/` required?
**Yes, if you want Streamlit's native multipage navigation.**

- Mechanism: Streamlit scans the `pages/` folder next to your main `app.py`. Every `.py` file inside becomes a subpage and appears in the sidebar automatically.
- Same-level files (`a.py`, `b.py`) placed next to `app.py` but not inside `pages/` are not auto-registered as pages.
- Alternative: If you do not want `pages/`, implement manual navigation (e.g., `st.radio` + conditional imports) inside `app.py`.

---

## 2. Static Assets Management

### 2.1 Is `static/` mandatory?
**Not necessarily—it depends on how you serve assets.**

1) Load assets in code paths (common)
- You can store images anywhere and load via relative paths.
- Example: `st.image("./assets/logo.png")`

2) Serve assets via URL (advanced)
- If you want users to access `http://your-ip:8501/static/image.png` directly or reference images in Markdown/HTML (`<img src='/static/image.png'>`), enable static serving.

Enable in `.streamlit/config.toml`:

```toml
[server]
enableStaticServing = true
```

By default, Streamlit expects a `static` folder at the project root as the static root.

---

## 3. Layered Architecture for Pages

Write clear, maintainable Streamlit apps by separating responsibilities.

### 3.1 Layers

**Data Layer**
- Responsibilities: load/cache/preprocess/persist data (decoupled from UI)
- Pseudofunctions: `get_app_config()`, `load_metadata(cfg)`, `cache_dataset(fn)` (prefer `st.cache_data`)

**Business Logic Layer**
- Responsibilities: core computation and rules
- Pseudofunctions: `run_search(query, providers)`, `merge_deduplicate(records)`, `apply_filters(records, filters)`

**View/UI Layer**
- Responsibilities: render UI components
- Pseudofunctions: `render_results_table(records)`, `render_editor_panel(record)`

**Components Layer**
- Responsibilities: reusable page fragments (sidebar, navigation, filters)
- Pseudofunctions: `display_search_sidebar()`, `navigation_menu()`, `filters_panel()`
- Location: `visualization/components/`

**Control/Orchestration Layer**
- Responsibilities: the main coordinator that chains data → logic → view
- Pseudofunctions: `main()`

### 3.2 Example Skeleton

```python
def main():
    cfg = get_app_config()
    dataset = load_metadata(cfg)
    display_search_sidebar()
    results = run_search(query=..., providers=...)
    results = apply_filters(results, filters=...)
    render_results_table(results)
```

---

## 4. Components: Recommendations & Index

See https://streamlit.io/components for more components.
Note: Pay attention to version compatibility between external components and your Streamlit version.

### 4.1 Common Components

**Auth & Encryption**
- `streamlit-authenticator`: login & sessions (with `bcrypt`)
- `bcrypt`: password hashing
- Install: `pip install streamlit-authenticator bcrypt`

**Documents & Files**
- `streamlit-pdf-viewer`: PDF preview with page navigation
- `streamlit-aggrid`: interactive tables and editing
- Install: `pip install streamlit-pdf-viewer streamlit-aggrid`

**Chemistry & Specialized**
- `st-ketcher`: chemical structures view & edit
- Install: `pip install st-ketcher`

**Navigation & Interaction**
- `streamlit-option-menu`: simple top/side navigation
- `streamlit-ace`: embedded code editor
- Install: `pip install streamlit-option-menu streamlit-ace`

### 4.2 Usage Suggestion

- Wrap external components inside `visualization/components/` as reusable, testable fragments.
- Use unified naming like `render_xxx()` or `display_xxx()` and call them within pages.

```python
# components/pdf_viewer.py
def render_pdf(path):
    ...

# pages/1_Analysis.py
from visualization.components.pdf_viewer import render_pdf
render_pdf("docs/report.pdf")
```

---

## 5. Configuration Management & Launch Options

### 5.1 Config Loading Priority
Streamlit loads config in this order (high → low):
1) Command-line args (e.g., `streamlit run app.py --server.port 8080`)
2) Environment variables (e.g., `STREAMLIT_SERVER_PORT=8080`)
3) Project config file (`./.streamlit/config.toml`)
4) Global config (`~/.streamlit/config.toml`)

### 5.2 Can you set the port in `main.py`?
**No.** Streamlit binds the port before your script runs, so you cannot set the port inside `st.set_page_config`.

### 5.3 Solution: Python Launcher Script

If you prefer a Python entry that centralizes port, config path, and environment tuning, write a small launcher in the project root.

```python
import os
import sys
from streamlit.web import cli as stcli

def resolve_config_path():
    return os.path.join(os.getcwd(), ".streamlit", "config.toml")

def main():
    # 1) Environment variables (override config.toml)
    os.environ["STREAMLIT_SERVER_PORT"] = "8505"
    os.environ["STREAMLIT_THEME_BASE"] = "dark"

    # 2) Prepare launch args
    # Equivalent to: streamlit run visualization/app.py --server.maxUploadSize=500
    sys.argv = [
        "streamlit",
        "run",
        "visualization/app.py",
        "--server.maxUploadSize=500",
        "--global.developmentMode=false",
    ]

    # 3) Launch Streamlit
    print("🚀 Launching Streamlit via Python launcher...")
    sys.exit(stcli.main())

if __name__ == "__main__":
    main()
```

Run with: `python launcher.py`

---

## 6. Security & Networking

### 6.1 Firewall Rules
After deployment, ensure the server firewall allows the Streamlit port (default `8501`).

**Windows Server (PowerShell)**
```powershell
New-NetFirewallRule -DisplayName "Streamlit App" -Direction Inbound -LocalPort 8501 -Protocol TCP -Action Allow
```

**Linux (Ubuntu/UFW)**
```bash
sudo ufw allow 8501/tcp
sudo ufw reload
```

### 6.2 Password Hashing & Verification
Do not store plaintext passwords in `secrets.toml`. Store hashes instead.

Step 1 — Generate a hash
```python
import streamlit_authenticator as stauth

hashed_passwords = stauth.Hasher(['my_secure_password']).generate()
print(hashed_passwords)
# Example output: ['$2b$12$xxxxxxxxxxxxxxxxxxxxxx']
```

Step 2 — Store in `.streamlit/secrets.toml`
```toml
[auth]
password_hash = "$2b$12$xxxxxxxxxxxxxxxxxxxxxx"
```

Step 3 — Verify in your app
```python
import streamlit as st
import hashlib

def check_password():
    def password_entered():
        input_hash = hashlib.sha256(st.session_state["password"].encode()).hexdigest()
        if input_hash == st.secrets["auth"]["password_hash"]:
            st.session_state["password_correct"] = True
            del st.session_state["password"]
        else:
            st.session_state["password_correct"] = False

    if "password_correct" not in st.session_state:
        st.text_input("Password", type="password", on_change=password_entered, key="password")
        return False
    elif not st.session_state["password_correct"]:
        st.text_input("Password", type="password", on_change=password_entered, key="password")
        st.error("Password incorrect")
        return False
    else:
        return True
```

---

## 7. Official References

- Configuration: https://docs.streamlit.io/develop/api-reference/configuration/config.toml
- Multipage Apps: https://docs.streamlit.io/develop/concepts/multipage-apps
- Static File Serving: https://docs.streamlit.io/develop/concepts/configuration/serving-static-files
- Secrets Management: https://docs.streamlit.io/develop/concepts/connections/secrets-management
- Deployment Guide: https://docs.streamlit.io/deploy/streamlit-community-cloud/deploy-your-app

