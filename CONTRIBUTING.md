# Contributing

Thanks for contributing.

## Local validation

Run these commands before opening a pull request:

```bash
helm lint .
helm template dynamic-reverse-proxy . -f values.example.yaml
```

## Pull request checklist

- Keep changes focused and minimal.
- Update documentation when behavior changes.
- Keep `values.yaml` defaults backwards-compatible where possible.
- Include an example command when adding new values.

## Commit style

Use clear, imperative commit messages, for example:

- `Add proxy timeout values`
- `Document OCI publish workflow`
