# Cardboard Management Backend

This repository is the Bench-level backend workspace for Cardboard Management.

## Components

- `apps/frappe` — Frappe Framework, version 15
- `apps/erpnext` — ERPNext, version 15
- `apps/cardboard_management` — Cardboard Management application, branch `MVP`
- `sites`, `config`, `Procfile` — Bench runtime configuration copied from the development workspace

Each application remains an independent Git repository and is also registered as a Git submodule so its history and upstream relationship are preserved.

## Clone on a new machine

```bash
git clone --recurse-submodules https://github.com/muhhammedeid/cardboard-backend.git
cd cardboard-backend
```

For an existing clone:

```bash
git submodule update --init --recursive
```

The `cardboard_management` submodule is private and requires GitHub access for the account performing the clone.

## Important

The repository does not contain production credentials. Configure site secrets locally in `sites/*/site_config.json` on the new machine. Do not commit passwords, API keys, or database dumps.

The Windows-side copy at `D:\Mohamed\cardboard-Backend` contains the complete transferred workspace, including the independent Git metadata directories for the three applications.
