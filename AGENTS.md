# AGENTS.md

## Cursor Cloud specific instructions

### Overview

This is a NetBox + Arista AVD CI/CD pipeline demo repository with three main components:

1. **Webhook Server** (`webhook_server/sync_netbox_avd_cvaas.py`) — Flask/Waitress server that receives webhooks from NetBox and Gitea, triggering Ansible playbooks and Git operations. Runs on port 5000.
2. **NetBox Plugins** (`netbox_plugins/`) — Three Django-based plugins (`netbox-run-anta-plugin`, `netbox-vlan-creator-status-plugin`, `netbox-sync-manager-plugin`) designed to run inside a NetBox instance.
3. **Ansible Playbooks** (root `*.yml`) — Playbooks for inventory management, config generation (AVD), deployment, and ANTA testing.

### Running the Webhook Server

```bash
cd webhook_server
NETBOX_WEBHOOK_SECRET=<secret> GITEA_WEBHOOK_SECRET=<secret> python3 sync_netbox_avd_cvaas.py
```

The server has hardcoded paths (`/home/andreasm/...`) for the repo and env file. In the cloud environment, the server will start and accept HTTP requests, but background Ansible/Git operations will fail because those paths don't exist. This is expected — the webhook HTTP layer is fully testable.

### Linting

```bash
flake8 webhook_server/ scripts/ netbox_plugins/ --max-line-length=120
```

The codebase has ~114 pre-existing flake8 findings (unused imports, whitespace, line length). These are in the original code.

### Testing

- **Plugin tests** (`pytest netbox_plugins/`) require the full NetBox Django framework (`netbox` module) which is not installed in the cloud environment. Import errors during collection are expected.
- **Ansible syntax check**: `ansible-playbook --syntax-check <playbook>.yml` — works for all playbooks with `arista.avd` collection installed.

### Key Gotchas

- The webhook server's `REPO_PATH` and `ENV_FILE` constants are hardcoded to the original author's machine paths. The HTTP endpoints (`/status`, `/latest-report`, `/webhook`, `/gitea-webhook`) work regardless, but background processing threads will fail.
- Ansible playbooks that use AVD roles (`build.yml`, `dev-build.yml`, `deploy.yml`, etc.) require the `arista.avd` Ansible Galaxy collection.
- The three NetBox plugins each have their own `pyproject.toml` and can be installed in editable mode with `pip install -e .` from their respective directories.
- Python dependencies for the webhook server: `flask`, `waitress`, `requests`, `rich`, `python-dotenv`.
- Dev tools: `flake8`, `black`, `pytest`, `check-manifest`.
