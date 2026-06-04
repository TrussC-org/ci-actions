# ci-actions

Reusable GitHub Actions workflows for the **TrussC** ecosystem. The first
workflow, `build-addon`, builds a TrussC addon and its examples across desktop
platforms (macOS / Windows / Linux) and runs the addon's optional test harness.

## Usage

In your addon repo, add `.github/workflows/ci.yml`:

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    uses: TrussC-org/ci-actions/.github/workflows/build-addon.yml@v1
```

That's it — no per-addon configuration in the common case. The workflow infers
everything from the repo:

- **Addon name** = the repository name (override with `addon_name`).
- **Platforms** = `addon.json` `"platforms"` (see below). Default: macOS,
  Windows, Linux.
- **Examples** = every `example-*/` directory is built.
- **Tests** = `tests/` is built and run **if it exists** (a non-zero exit fails
  the job); skipped if absent. Tests are opt-in by presence of the directory.

### Platforms (`addon.json`)

```json
{
  "platforms": ["macos", "windows", "linux"]
}
```

- Omit the field to get the default desktop trio (`macos`, `windows`, `linux`).
- Restrict it for platform-specific addons, e.g. `["macos"]`.
- `web` / `android` / `ios` are recognized but **not built yet** (a warning is
  emitted, the job is not failed). Desktop-only for now.

### Tests (`tests/`)

Add a `tests/` directory containing a normal TrussC console project whose
`main()` returns non-zero on failure. CI builds and runs it automatically. See
[`tcxPly`](https://github.com/tettou771/tcxPly) for a worked example.

### Inputs (all optional)

| Input | Default | Description |
|---|---|---|
| `addon_name` | repo name | Override the addon directory name |
| `trussc_ref` | `main` | TrussC branch/tag/commit to build against |
| `configs` | `["Release"]` | JSON array of CMake configs |
| `extra_apt_packages` | `""` | Extra Linux apt packages |
| `extra_addons` | `""` | Extra addons to clone (`owner/repo name ...`) |
| `cache_key_suffix` | `v1` | Bust the trusscli cache |

## Credit

Originally based on [**2bbb/trussc-actions**](https://github.com/2bbb/trussc-actions)
(MIT, Copyright (c) 2025 2bbb). This is an independent, TrussC-org–maintained
rewrite that adds addon-name auto-detection, `addon.json`-driven platform
selection, example building, and opt-in `tests/`. Many thanks to 2bbb for the
original.

## License

MIT — see [LICENSE](LICENSE).
