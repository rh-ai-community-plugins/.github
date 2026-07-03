# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

This is the `.github` org-level profile repo for the `rh-ai-community-plugins` GitHub organization. It contains the org profile README (displayed on the org's GitHub page) and any org-wide GitHub config.

The organization hosts community plugins for the Red Hat OpenShift AI (RHOAI) dashboard. Plugins extend the RHOAI dashboard UI via Module Federation without touching the core platform.

## Organization Repos

| Repo | Purpose |
|------|---------|
| `charter` | Governance, plugin spec, submission process, plugin catalog (`plugins.yaml`) |
| `hello-plugin-world` | Reference/template plugin — minimal Hello World dashboard page |
| `hermes-agent-deployer` | Deploy and manage Hermes Agent instances from the RHOAI dashboard |
| `kueue-visualizer` | Visualize Kueue workload scheduling (queue topology, capacity, workloads) |
| `.github` | This repo — org profile and org-wide config |

## Key Concepts

- **Community plugins are unsupported by Red Hat** — maintained by community, no SLAs
- **Plugin catalog** lives in `charter/plugins.yaml` — new plugins submit PRs there
- **Plugin lifecycle**: Experimental -> Beta -> Stable-Candidate -> Deprecated -> Archived
- **Deployment models**: `per-project` (user provisions per namespace), `cluster-shared` (admin installs once), or `both`
- **Module Federation** is how plugins integrate with the RHOAI dashboard — each plugin serves a `remoteEntry.js` that the dashboard loads at runtime

## Plugin Technical Stack

All plugins in this org share a common pattern:

- **UI**: React 18, PatternFly 6, TypeScript, Webpack 5, Module Federation
- **Dashboard integration**: `@openshift/dynamic-plugin-sdk` — exposes `./extensions` (nav/route registration) and `./Icon` (sidebar icon)
- **Container**: UBI9 base, non-root UID 1001, port 8080, nginx serving the webpack bundle
- **Deployment**: Helm chart in `chart/` directory
- **Registry**: `quay.io/rh-ai-community-plugins/<plugin-name>`
- **RHOAI compatibility**: 3.4.0+ (declared in `plugin.yaml`)

## Plugin Repo Structure (Standard)

```
plugin-name/
├── plugin.yaml          # RHOAI plugin manifest (required)
├── chart/               # Helm chart (required)
├── src/
│   ├── rhoai/
│   │   ├── extensions.ts    # Dashboard nav/route/area registration
│   │   └── *NavIcon.tsx     # Sidebar icon component
│   └── app/
│       └── components/      # PatternFly 6 UI components
├── config/
│   ├── webpack.common.js
│   ├── webpack.dev.js
│   ├── webpack.prod.js
│   └── moduleFederation.js
├── Containerfile
└── package.json
```

## Common Commands (per plugin repo)

```bash
npm install
npm run start:dev       # dev server with HMR
npm run build           # production build -> dist/
npm test                # jest unit tests
npm run lint            # eslint src/
podman build -t <image> .
helm template <name> chart/ | oc apply --dry-run=client -f -
```

## Security Rules

- Non-root containers (UID 1001+), no privileged mode, no privilege escalation
- Non-privileged ports only (8080, 8443)
- UBI9 base images preferred
- All RBAC declared in `plugin.yaml` — no undeclared permissions
- Plugins cannot access RHAIE internal databases or APIs
- `helm uninstall` must leave zero orphaned resources

## Charter Docs Reference

Detailed specs live in `charter/docs/`:
- `plugin-spec.md` — full `plugin.yaml` schema and repo structure
- `helm-requirements.md` — Helm chart rules
- `security-guidelines.md` — container and RBAC security policy
- `examples/example-plugin.yaml` — starter template
