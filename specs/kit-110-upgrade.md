# SphereFlake: Kit SDK 106.4 → 110 Upgrade

Upgrade `sf4ovc-106-4` (and the companion `sphereflake22` extension at `D:/ov/exts`) from NVIDIA Omniverse Kit SDK 106.4 to 110. Dropping the streaming variant for this upgrade — it will be reintroduced cleanly later against 110's `streaming_configs/` templates (likely CloudXR for Quest/Vision Pro, not WebRTC).

## Context

- Starting point: tag `v106.4-working` on `main` (commit `dc6f71c`), verified working 2026-04-20.
- Working branch: `kit-110` (branched from `main` at `dc6f71c`).
- Reference clone: `D:/ov/apps/kit-app-template-110` (upstream NVIDIA-Omniverse/kit-app-template at 110.1.0, commit `c98fc5c`) — for side-by-side comparison, not edited.
- Strategy: rebase SphereFlake customizations on top of upstream 110 structure. The 106 `define_app()` model is gone in 110; tooling pattern has shifted to `template new` + `import_configs`.

## Key findings from upgrade research

- **SDK version string**: `110.1.0+feature.${platform_target_abi}.${config}` (was `106.4.0+release.156974.9321c814.gl`). `${platform}` → `${platform_target}` in packman `linkPath`.
- **Registry URL**: `${kit_version_major}` substitution replaces hardcoded `106` in `repo.toml`.
- **Cumulative breaking changes (106 → 110)**: Python 3.10 → 3.12; USD 24.05 → 25.11; C++17 + `_GLIBCXX_USE_CXX11_ABI=1`; Carbonite Events 2.0; `ITokens`/`IDictionary`/`ISettings` string-view APIs; fabric `Token`/`Path` ABI break; Hydra 2 backend disabled; multi-node rendering removed; `omni.renderer_capture` and `omni.kit.widget.settings.deprecated` removed; UsdUI `displayName`/`displayGroup` accessors deprecated.
- **Streaming**: `omni.kit.livestream.webrtc` hand-roll is replaced by four preset layer templates (`default_stream`, `ovc_stream`, `nvcf_stream`, `gdn_stream`). Out of scope for this upgrade.

## Latent bugs in `sphereflake22` (pre-existing, fix as part of this work)

- `extension.py` calls `omni.kit.app.get_app()` and `omni.usd.get_context()` without importing those modules — currently works by accident because other extensions import them first into the process. Will be surfaced harder by stricter 110 startup ordering.
- Uses `asyncio.ensure_future()` (deprecated since Python 3.11) in `sfcontrols.py`, `sfwintabs.py`, `sfgen/sphereflake.py`. Python 3.12 in Kit 110 may remove.
- `asyncio.get_event_loop()` in `sfcontrols.py` deprecated — switch to `asyncio.get_running_loop()` or task-based pattern.

## Task status

| # | Task | Status | Notes |
|---|------|--------|-------|
| 1 | Branch `kit-110` off `main` | ✅ Done | At commit `dc6f71c` |
| 2 | Clone upstream 110 to `D:/ov/apps/kit-app-template-110` | ✅ Done | Commit `c98fc5c` (110.1.0) |
| 3 | Delete `msft.sphereflake1064_streaming.kit` + remove from `repo.toml` `[repo_precache_exts].apps` | ✅ Done | Also removed `define_app` entry in `premake5.lua`, updated README/CLAUDE.md |
| 4 | Update `tools/deps/kit-sdk.packman.xml` → 110.1.0 | ✅ Done | Bumped to `110.1.0+feature.${platform_target_abi}.${config}`, `${platform}` → `${platform_target}` |
| 5 | Update `repo.toml` registry URL → `${kit_version_major}` | ✅ Done | One-line change |
| 6 | Adopt upstream 110 `repo.toml` deltas | ✅ Done | Picked up: 4 deprecation entries, cache token, `generated_app_path`, `windows_max_path_length`, `_repo/**` exclude, `[repo_launch_app]`, `[repo_package_app]`. Skipped: kit_core_templates/kit_sample_templates packages (template-repo-only), container packaging block (defer until needed) |
| 7 | Trim `premake5.lua` for 110 | ⚠️ Partially done — `define_app()` lives | I misread upstream's mostly-empty `premake5.lua` (they ship with no apps; `repo template new` injects `define_app()` lines on demand). The function still exists and is needed: it generates the per-app `_build/.../<app>.kit.bat` launcher that `repo.bat launch --name X` invokes. Without it, launch fails with "missing built entrypoint script". Restored `define_app("msft.sphereflake.kit")` in commit after rename. The other Task 7 changes (drop `setup_options()`, drop MSVC/WINSDK option reads, pass `cppdialect = "C++17"`) match upstream and were correct. |
| 8 | Re-precache: regenerate `.kit` version-lock block | ✅ Done (build #3) | Build #1: TOML duplicate-key error (`[settings.app.exts]` defined twice in old .kit). Build #2: precache succeeded but premake5.exe path bug — old `repo_build 1.2.0` couldn't find premake. Build #3: bumped repo tooling to 110 versions + synced kit-sdk-deps + added host-deps + synced repoman/launch.py/package.py + bumped tools/VERSION.md to 110.1.0 → `BUILD (RELEASE) SUCCEEDED`. Cosmetic: spurious "Redefine of existing key /dependencies/omni.kit.actions.core" still logged at Error level by transitive dep, but doesn't fail the build. |
| 9 | `.kit` file: add new required deps | ✅ Done | Added `omni.hydra.usdrt_delegate`, `omni.kit.developer.bundle`, `omni.kit.primitive.mesh`, `omni.kit.widget.cache_indicator`. `omni.activity.profiler` was already present. |
| 10 | `.kit` file: move telemetry keys to top-level `[settings]` | ✅ Done | `[settings.telemetry]` block removed; `telemetry.*` + `privacy.externalBuild` at top-level `[settings]`. |
| 11 | `.kit` file: add `[settings.app.environment]` | ✅ Done | `name = "SphereFlake 106.4"` for now; rename in Task 22. |
| 12 | `.kit` file: add `[settings.persistent.app] viewport.autoFrame.mode = "first_open"` | ✅ Done | |
| 13 | Fix `extension.py` missing imports (`omni.kit.app`, `omni.usd`) | 🟢 Deferred | Did not fire on Kit 110 launch — transitive imports still mask the bug. Worth fixing for hygiene but not blocking. |
| 14 | Replace `asyncio.ensure_future()` → `asyncio.create_task()` (3 files) | 🟢 Deferred | Python 3.12 still emits these as DeprecationWarning, not error. Fix when convenient. |
| 15 | Replace `asyncio.get_event_loop()` with running-loop pattern | 🟢 Deferred | Same as #14. |
| 16 | Audit `sphereflake22` for `ITokens`, `omni.renderer_capture`, `omni.kit.widget.settings.deprecated`, UsdUI metadata accessors | 🟢 Deferred | Benchmark ran cleanly, so no removed APIs are on the hot path. Audit later. |
| 17 | Verify `omni.ui` model callback signatures in `_widgets.py` | 🟢 Deferred | UI panels rendered, controls work. |
| 18 | Update `sphereflake22/extension.toml` version pins if needed | 🟢 Deferred | No pin issues encountered. |
| 19 | `repo.bat build` clean | ⬜ Todo | |
| 20 | `repo.bat launch --name msft.sphereflake1064.kit` → UI opens, viewport renders | ✅ Done | First launch: app ready 12.9s, RTX ready 122.9s. Subsequent launches: ~6s + 15s (cached). sphereflake22 auto-loaded from `d:/ov/exts` search path. Two warnings at startup, both benign: (1) `[Error] [carb.dictionary.serializer-toml.plugin] Redefine of existing key /dependencies/omni.kit.actions.core` at ~5ms — root cause unknown, fires after Kit scans `d:/ov/exts` but doesn't break anything; (2) `omni.kit.tool.asset_importer-2.10.3+106.4.0/utils.py:82: SyntaxWarning: invalid escape sequence '\\.'` — NVIDIA hasn't published a 110/cp312 build of `omni.kit.tool.asset_importer-2.10.3`; solver picks 106.4 build as best available. Wait for NVIDIA registry update. Added `omni.kit.window.about` so Help → About shows Kit version; window title now `"SphereFlake (Kit 110.1.0)"`. |
| 21 | Load `sphereflake22` via Extension Manager, generate sphereflake, confirm `log.txt` entry | ✅ Done | sphereflake22 auto-loaded from `d:/ov/exts` search path (no Extension Manager interaction needed). Generated 12-flake (3×3×4) UsdSphere classic at 2026-04-26 12:34:30, elapsed 1.423s. Format identical to 106.4 entries. No runtime errors. Spec Tasks 13-18 latent bugs (asyncio.ensure_future, asyncio.get_event_loop, missing imports) did not fire under 3.12 — deprecations are still warnings, not errors; missing imports still masked by transitive load order. Defer cleanup to next sphereflake22 work session. |
| 22 | Update README status line, tag `v110.0-working` | ⬜ Todo | |
| 23 | Commit & push `kit-110` branch, optionally merge to `main` | ⬜ Todo | |

## Rollback

- `main` untouched. `git checkout main` returns to working 106.4.
- `v106.4-working` tag at `5705a70` (on origin) is the authoritative fallback point.

## Out of scope

- Streaming variant (`msft.sphereflake1064_streaming.kit`) — to be reintroduced later using 110's `streaming_configs/` templates. CloudXR path likely for Quest/Vision Pro.
- Porting to Kit Services (headless) or USD Viewer template — current desktop editor pattern is correct for the benchmarking use case.

## References

- [Kit 110.0 Release Notes](https://docs.omniverse.nvidia.com/dev-guide/latest/release-notes/110_0.html)
- [Kit 110 Migration Guide](https://docs.omniverse.nvidia.com/kit/docs/kit-manual/110.0.0/guide/migration.html)
- [Kit 109 Migration Guide](https://docs.omniverse.nvidia.com/kit/docs/kit-manual/109.0.0/guide/migration.html)
- [Kit 108 Migration Guide](https://docs.omniverse.nvidia.com/kit/docs/kit-manual/108.0.0/guide/migration.html)
- [Kit 107 Migration Guide](https://docs.omniverse.nvidia.com/kit/docs/kit-manual/107.0.3/guide/migration.html)
- [kit-app-template (upstream)](https://github.com/NVIDIA-Omniverse/kit-app-template)
