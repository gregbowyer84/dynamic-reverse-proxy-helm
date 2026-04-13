# Dynamic Reverse Proxy Helm Chart

This chart supports values-based overrides for resource name and namespace.
The chart now lives at the repository root.

## Key values

- `nameOverride`: short app name used in generated names
- `fullnameOverride`: full resource name (takes precedence over `nameOverride`)
- `namespaceOverride`: namespace injected into rendered manifests
- `proxy.resolver`: DNS resolver service for runtime upstream lookups
- `proxy.scheme`: upstream scheme used in `proxy_pass` (default `https`)
- `proxy.sslVerify`: NGINX `proxy_ssl_verify` setting (`on` or `off`)

## Route format

This chart proxies dynamically based on the URL path:

- `/<target-host>/<target-path>` -> `<scheme>://<target-host>/<target-path>`

Examples:

- `/api.example.com/v1/health` -> `https://api.example.com/v1/health`
- `/example.com/` -> `https://example.com/`

## No-clone deployment options

### 1. Deploy from an OCI registry (recommended for no clone)

Package and publish the chart once from CI/CD, then anyone can deploy with Helm.

Publisher step (one-time per version):

helm package .
helm push dynamic-reverse-proxy-0.1.0.tgz oci://<registry>/<charts>

GitHub Container Registry example:

helm registry login ghcr.io -u <github-username>
helm package .
helm push dynamic-reverse-proxy-0.1.0.tgz oci://ghcr.io/<github-owner>/charts

Consumer step (no repo clone):

helm upgrade --install dynamic-reverse-proxy oci://<registry>/<charts>/dynamic-reverse-proxy --version 0.1.0 --namespace proxy --create-namespace -f values.example.yaml

GitHub Container Registry consume example:

helm upgrade --install dynamic-reverse-proxy oci://ghcr.io/<github-owner>/charts/dynamic-reverse-proxy --version 0.1.0 --namespace proxy --create-namespace -f values.example.yaml

No local values file example:

helm upgrade --install dynamic-reverse-proxy oci://ghcr.io/<github-owner>/charts/dynamic-reverse-proxy --version 0.1.0 --namespace proxy --create-namespace --set nameOverride=proxy --set namespaceOverride=proxy --set proxy.scheme=https --set proxy.sslVerify=off

### 2. Deploy directly from a chart URL

If you host `.tgz` in artifact storage (GitHub Releases, Blob, S3):

helm upgrade --install dynamic-reverse-proxy https://<host>/dynamic-reverse-proxy-0.1.0.tgz --namespace proxy --create-namespace -f values.example.yaml

### 3. Deploy from a Git URL (no local clone)

If your Helm supports Git URLs/plugins, you can install from the chart path remotely. This is less portable than OCI and URL package hosting.

## Namespace behavior

- Helm release namespace is set with `--namespace`.
- `namespaceOverride` in values can force rendered resource namespace when needed.
- If both are set, templates use `namespaceOverride`.

## Local render check

helm template dynamic-reverse-proxy . -f values.example.yaml

## Project standards

- Contributing: [CONTRIBUTING.md](CONTRIBUTING.md)
- Security policy: [SECURITY.md](SECURITY.md)
- Code of conduct: [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
- License: [LICENSE](LICENSE)
- Changelog: [CHANGELOG.md](CHANGELOG.md)

## CI and release automation

- CI workflow: [.github/workflows/ci.yaml](.github/workflows/ci.yaml)
- Testing workflow: [.github/workflows/testing.yaml](.github/workflows/testing.yaml)
- Cut release workflow: [.github/workflows/cut-release.yaml](.github/workflows/cut-release.yaml)
- Release workflow: [.github/workflows/release.yaml](.github/workflows/release.yaml)
- Chart icon: [assets/icon.svg](assets/icon.svg)

Release workflow behavior:

1. Triggered by a tag in the format `vX.Y.Z`
2. Validates that `Chart.yaml` version equals the tag version without `v`
3. Runs `helm lint .`
4. Packages the chart as `.tgz`
5. Pushes chart to `oci://ghcr.io/<owner>/charts`
6. Creates a GitHub Release and uploads the chart archive

### Versioning and tag process

1. Update `version` in `Chart.yaml`
2. Commit and push to `main`
3. Create and push a matching tag

Example:

git tag v0.1.1
git push origin v0.1.1

The chart will be available from GHCR as:

`oci://ghcr.io/<owner>/charts/dynamic-reverse-proxy`

### Automated release cut

You can trigger releases from the GitHub Actions UI without running local git tag commands.

1. Open Actions -> Cut Release
2. Click Run workflow
3. Choose `bump` (`patch`, `minor`, or `major`) or provide explicit `version`

The workflow will:

1. Calculate and set the next chart version in `Chart.yaml`
2. Update `CHANGELOG.md` with a new version section
3. Commit and push the version bump to `main`
4. Create and push a matching `vX.Y.Z` tag
5. Trigger the Release workflow to publish to GHCR and GitHub Releases
