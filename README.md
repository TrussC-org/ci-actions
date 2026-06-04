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

That's it — no per-addon configuration in the common case.

### Which branches to run on

The recommended default is **every branch** — so a push never silently skips CI
and there's no "only `main`/`dev` are magic" surprise:

```yaml
on:
  workflow_dispatch:
  push:
    branches: ['**']
    tags: ['v*']
  pull_request:
```

This is cheap: the slow part (building `trusscli`) is cached per repo and shared
across branches (the default branch's cache is restorable by all branches), and
`concurrency` cancels superseded runs. If a particular addon's CI is genuinely
too heavy or low-value to run on every push, **narrow it** (opt-out), e.g.
`branches: [main]`. The workflow infers
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
| `setup_script` | `""` | Bash run before the build, for addons needing an external SDK (see below) |

### External SDKs (`setup_script`)

Hardware addons (depth cameras, capture devices, …) often need a vendor SDK that
isn't an apt package. Pass a `setup_script`: a bash snippet that runs on **every**
platform just before the build (after `trusscli` is ready, before any cmake
configure). Branch on `$RUNNER_OS`, download/install the SDK, and export whatever
the build needs via `$GITHUB_ENV`. CWD is the workspace root; the addon is checked
out at `./$ADDON_NAME`.

```yaml
jobs:
  build:
    uses: TrussC-org/ci-actions/.github/workflows/build-addon.yml@v1
    with:
      setup_script: |
        case "$RUNNER_OS" in
          Linux)   url=".../SDK_linux_x64.tar.gz" ;;
          macOS)   url=".../SDK_macos_arm64.tar.gz" ;;
          Windows) url=".../SDK_windows.zip" ;;
        esac
        # download + extract into ./sdk ...
        echo "CMAKE_PREFIX_PATH=$PWD/sdk/lib/cmake/MySDK" >> "$GITHUB_ENV"
```

Combine with `addon.json` `"platforms"` to drop platforms a SDK doesn't support
(e.g. `["windows","linux"]` skips macOS). With no `tests/` directory, examples
are built but not run — the right default when the addon needs real hardware.

## Credit

Originally based on [**2bbb/trussc-actions**](https://github.com/2bbb/trussc-actions)
(MIT, Copyright (c) 2025 2bbb). This is an independent, TrussC-org–maintained
rewrite that adds addon-name auto-detection, `addon.json`-driven platform
selection, example building, and opt-in `tests/`. Many thanks to 2bbb for the
original.

## License

MIT — see [LICENSE](LICENSE).
