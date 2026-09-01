# EIC Deployment Area Overview

This document describes the directory structure used under `/eic/opt/` for managing the deployment, testing, and operational releases of software. This layout supports multiple release areas (development, commissioning, operational), fallback mechanisms, and separate areas for scripts, binaries, and configurations.

It is intended to provide a clean, standardized, and intuitive foundation for managing executable applications, including C++ binaries, Java applications, Python applications bundled using [Shiv](https://shiv.readthedocs.io/) and Shell scripts.

---

## 📁 Directory Structure
::

    /eic/opt/
    ├── releases/
    │   ├── development/
    │   │   ├── bin/
    │   │   ├── scripts/
    │   │   ├── config/
    │   │   └── fallback/
    │   ├── commissioning/
    │   │   ├── bin/
    │   │   ├── scripts/
    │   │   └── config/
    │   ├── operational/
    │   │   ├── bin/
    │   │   ├── scripts/
    │   │   ├── config/
    │   │   └── fallback/
    │   └── shared_env/
    ├── packages/
    ├── tools/
    │   ├── promote_release.sh
    │   └── validate_symlinks.py
    ├── docs/
    │   ├── README.md
    │   ├── release_checklist.md
    │   ├── promotion_policy.md
    │   └── rollback_procedure.md

---

## 🔁 Release Promotion Workflow

1. **Development**: Software is staged in `development/bin`, tested by developers.
2. **Commissioning**: Once stable, contents are promoted to `commissioning/bin` for broader user testing.
3. **Operational**: Approved versions are promoted to `operational/bin` for production use.
4. **Fallback**: Previous versions of `development` and `operational` versions are archived in their respective `fallback/` directories.

Symlinks (e.g., `MyApp -> MyApp-1.2.3.shiv`) may be used in `bin/` to abstract version-specific files.

---

## 📚 Documentation & Source of Truth

Full documentation is hosted on ReadTheDocs:

🔗 https://eic.readthedocs.io/en/latest/eic-development-guidelines/release_procedure.html

This directory (`/eic/opt/docs/`) includes essential local documentation for operators and system maintainers, especially useful in offline or emergency scenarios.

---

## 🛠 Notes

- Only `development` and `operational` environments maintain a `fallback/` directory.
- `scripts/` should only contain **executable scripts** (e.g., `.sh`, `.py` with shebangs).
- Non-executable support logic (e.g., libraries or templates) should live in `config/`.

---

For questions or contributions to this structure, please refer to `docs/contacts.md` 
