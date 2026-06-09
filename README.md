# nomad_oasis_BC_helm

Helm chart repository for deploying [NOMAD Oasis](https://nomad-lab.eu/) and its
dependencies (Elasticsearch, JupyterHub, Temporal, PostgreSQL, Keycloak) on
Kubernetes.

Charts are published to **GitHub Pages** via GitHub Actions:
<https://bigchemistry-infrastructure.github.io/nomad_oasis_BC_helm>

## Usage

```bash
helm repo add nomad-bc https://bigchemistry-infrastructure.github.io/nomad_oasis_BC_helm
helm repo update
helm install nomad-oasis nomad-bc/default -f my-values.yaml
```

## Repository layout

```
charts/
  default/                 # The NOMAD umbrella chart
    Chart.yaml             # Chart + subchart dependency definitions
    values.yaml            # Default values
    templates/             # NOMAD resources (app, worker, proxy, mongodb, ...)
    charts/                # Vendored subchart dependencies (*.tgz)
    custom-values/         # Example value overlays (aws, kind, minikube, tls, ...)
helpers/                   # Local dev helper scripts (kind/minikube setup, status)
.github/workflows/
  release-charts.yml       # CI: package + publish charts to GitHub Pages
```

## How publishing works

The workflow at `.github/workflows/release-charts.yml` runs on every push to
`main` (and can be triggered manually). It:

1. Installs Helm.
2. Restores a cache of downloaded subcharts (`~/.cache/helm` and
   `charts/**/charts/*.tgz`) to avoid re-fetching them on every run.
3. Adds and **updates** the upstream subchart repositories (bitnami, elastic,
   temporal, jupyterhub, codecentric).
4. Runs `helm dependency update` so the **related subcharts are cached/vendored**
   into each chart before packaging.
5. Lints and packages every chart under `charts/`.
6. Downloads the currently published charts so older versions remain
   installable, then rebuilds a merged `index.yaml`.
7. Deploys the result to GitHub Pages.

## One-time setup

In the GitHub repository **Settings -> Pages**, set
**Build and deployment -> Source** to **"GitHub Actions"**. No personal access
token is required; the workflow uses the built-in `GITHUB_TOKEN` with `pages:
write` permission.

## Releasing a new version

1. Make your chart changes under `charts/default/`.
2. Bump `version:` (and `appVersion:` if applicable) in
   `charts/default/Chart.yaml`.
3. Commit and push to `main`. The workflow packages the new version and adds it
   to the published index alongside previous releases.
