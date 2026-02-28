# Helm Cheatsheet

> Quick-reference for managing Kubernetes applications with Helm charts.

## 🔥 Most Used

| Command | What it does | Example |
|---|---|---|
| `helm install <rel> <chart>` | Install a chart | `helm install myapp ./chart` |
| `helm upgrade <rel> <chart>` | Upgrade a release | `helm upgrade myapp ./chart -f values-prod.yaml` |
| `helm uninstall <rel>` | Remove a release | `helm uninstall myapp` |
| `helm list` | List releases | `helm list -n prod` |
| `helm repo add <name> <url>` | Add chart repo | `helm repo add bitnami https://charts.bitnami.com/bitnami` |
| `helm repo update` | Refresh repo index | `helm repo update` |
| `helm template <rel> <chart>` | Render templates locally | `helm template myapp ./chart -f values.yaml` |
| `helm status <rel>` | Release status | `helm status myapp` |

## Repo Management

| Command | What it does | Example |
|---|---|---|
| `helm repo add <name> <url>` | Add a chart repository | `helm repo add stable https://charts.helm.sh/stable` |
| `helm repo list` | List configured repos | `helm repo list` |
| `helm repo update` | Update repo indexes | `helm repo update` |
| `helm repo remove <name>` | Remove a repo | `helm repo remove stable` |
| `helm search repo <keyword>` | Search repos for charts | `helm search repo nginx` |
| `helm search repo <chart> --versions` | List all versions | `helm search repo bitnami/redis --versions` |
| `helm search hub <keyword>` | Search Artifact Hub | `helm search hub prometheus` |

## Install / Upgrade / Rollback

| Command | What it does | Example |
|---|---|---|
| `helm install <rel> <chart>` | Install chart as release | `helm install api bitnami/nginx` |
| `helm install <rel> <chart> -f vals.yaml` | Install with custom values | `helm install api ./chart -f values-prod.yaml` |
| `helm install <rel> <chart> --set k=v` | Install with inline overrides | `helm install api ./chart --set replicas=3` |
| `helm install <rel> <chart> -n <ns> --create-namespace` | Install into namespace | `helm install api ./chart -n prod --create-namespace` |
| `helm install <rel> <chart> --dry-run` | Simulate install (no changes) | `helm install api ./chart --dry-run` |
| `helm upgrade <rel> <chart>` | Upgrade release | `helm upgrade api ./chart` |
| `helm upgrade --install <rel> <chart>` | Install-or-upgrade (idempotent) | `helm upgrade --install api ./chart` |
| `helm rollback <rel> <rev>` | Rollback to revision | `helm rollback api 2` |
| `helm uninstall <rel>` | Remove release | `helm uninstall api -n prod` |
| `helm list` | List all releases | `helm list -A` (all namespaces) |
| `helm history <rel>` | Show release revisions | `helm history api` |

### Common Install/Upgrade Flags

| Flag | Purpose |
|---|---|
| `-f values.yaml` | Override default values file |
| `--set key=value` | Override single value inline |
| `--set-string key=value` | Force value as string |
| `-n <namespace>` | Target namespace |
| `--create-namespace` | Create namespace if missing |
| `--dry-run` | Render + validate without applying |
| `--wait` | Wait until all resources are ready |
| `--timeout 5m` | Max wait time (with `--wait`) |
| `--atomic` | Rollback on failure |
| `--force` | Force resource replacement |
| `--version <ver>` | Specify chart version |

## Values

| Command | What it does | Example |
|---|---|---|
| `helm show values <chart>` | Show default values | `helm show values bitnami/nginx` |
| `helm get values <rel>` | Show deployed values | `helm get values api` |
| `helm get values <rel> --all` | Show all (defaults + overrides) | `helm get values api --all` |
| `helm get values <rel> -o yaml` | Output as YAML | `helm get values api -o yaml > current.yaml` |
| `helm install -f a.yaml -f b.yaml` | Merge multiple values files | Last file wins for conflicts |
| `helm install --set-json 'k=["a","b"]'` | Set JSON values | `helm install api ./chart --set-json 'env=["prod"]'` |

### Values Precedence (lowest → highest)

```
chart defaults (values.yaml) → parent chart values → -f file1.yaml → -f file2.yaml → --set flags
```

## Templates

| Command | What it does | Example |
|---|---|---|
| `helm template <rel> <chart>` | Render all templates locally | `helm template api ./chart` |
| `helm template <rel> <chart> -f vals.yaml` | Render with values | `helm template api ./chart -f values-prod.yaml` |
| `helm template <rel> <chart> -s templates/deploy.yaml` | Render single template | `helm template api ./chart -s templates/deployment.yaml` |
| `helm get manifest <rel>` | Show deployed manifests | `helm get manifest api` |
| `helm lint <chart>` | Lint chart for errors | `helm lint ./chart` |
| `helm lint <chart> -f vals.yaml` | Lint with values | `helm lint ./chart -f values-prod.yaml` |

## Debug

| Command | What it does | Example |
|---|---|---|
| `helm install --dry-run --debug <rel> <chart>` | Full debug output without applying | `helm install --dry-run --debug api ./chart` |
| `helm template --debug <rel> <chart>` | Render with debug info | `helm template --debug api ./chart` |
| `helm get manifest <rel>` | See what was actually deployed | `helm get manifest api` |
| `helm get notes <rel>` | Post-install notes | `helm get notes api` |
| `helm get hooks <rel>` | Show release hooks | `helm get hooks api` |
| `helm status <rel>` | Release status + notes | `helm status api` |
| `helm history <rel>` | Revision history + status | `helm history api` |

### Debug a Failed Release

```bash
# 1. Check release status
helm status myrelease

# 2. Check history for failed revisions
helm history myrelease

# 3. Compare rendered templates with values
helm template myrelease ./chart -f values.yaml --debug

# 4. Dry-run to catch errors before applying
helm upgrade myrelease ./chart -f values.yaml --dry-run --debug

# 5. Rollback if needed
helm rollback myrelease 1
```

## Chart Development

| Command | What it does | Example |
|---|---|---|
| `helm create <name>` | Scaffold a new chart | `helm create mychart` |
| `helm package <chart>` | Package chart as .tgz | `helm package ./mychart` |
| `helm lint <chart>` | Validate chart | `helm lint ./mychart` |
| `helm dependency list <chart>` | List chart dependencies | `helm dependency list ./mychart` |
| `helm dependency update <chart>` | Download/update deps | `helm dependency update ./mychart` |
| `helm dependency build <chart>` | Rebuild charts/ from lock | `helm dependency build ./mychart` |
| `helm show chart <chart>` | Show Chart.yaml metadata | `helm show chart ./mychart` |
| `helm show all <chart>` | Show all chart info | `helm show all bitnami/nginx` |

### Chart Structure

```
mychart/
├── Chart.yaml          # Metadata (name, version, dependencies)
├── Chart.lock          # Locked dependency versions
├── values.yaml         # Default configuration values
├── templates/          # Kubernetes manifest templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── _helpers.tpl    # Reusable template snippets
│   └── NOTES.txt       # Post-install message
└── charts/             # Dependency charts (vendored)
```

### Dependency Management in Chart.yaml

```yaml
dependencies:
  - name: redis
    version: "17.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled    # Toggle with values
```

```bash
helm dependency update ./mychart   # Downloads into charts/
helm dependency build ./mychart    # Rebuilds from Chart.lock
```

---

*See [Kubernetes 101](../../tutorials/rust-k8s-operator/README.md#part-3-kubernetes-101--deploy-with-kubectl) for deploying with kubectl before graduating to Helm.*
