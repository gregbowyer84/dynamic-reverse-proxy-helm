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
