# macOS signing verification (Issue #148)

Community verification that onot **v1.2.0** desktop artifacts fail strict signing checks when built with `mac.identity: null`, and that a **Developer ID + Hardened Runtime** build passes the same checks on current macOS.

Full command output: [`electron/build/MACOS_SIGNED_BUILD_VERIFY.log`](../electron/build/MACOS_SIGNED_BUILD_VERIFY.log) on [stein2nd/onot](https://github.com/stein2nd/onot).

## Environment

| Item | Value |
|------|--------|
| Machine | Apple Silicon (M4 class), arm64 |
| OS | macOS **27.2.1** (build **26B131**) |
| Tooling | Cursor **v3.29.15**; electron-builder **26.15.3**; Electron **43.2.0** |
| Source | `sktelecom/onot` @ `72ba211` (v1.2.0-aligned tree) |

## Official GitHub Release (`onot-1.2.0-arm64.dmg` → `/Applications/onot.app`)

| Check | Result |
|-------|--------|
| `hdiutil verify` | Valid |
| `codesign --verify --deep --strict` | **Fail** — `code has no resources but signature indicates they must be present` |
| `codesign -dv` | `Signature=adhoc`, `Identifier=Electron`, `Sealed Resources=none` |
| `spctl --assess --type execute` | **Fail** (same message as codesign) |

## Local signed build (personal **Developer ID**, Team ID `PJWXCWS5U5`; not an official SK telecom release)

Built from the same tree with CLI overrides (`mac.identity`, `mac.hardenedRuntime=true`). Notarization was **not** run (no CI/notary credentials). Output directory was under `/tmp` to avoid Dropbox extended attributes during signing.

| Check | Result |
|-------|--------|
| `codesign --verify --deep --strict` | **Pass** (`valid on disk`, satisfies Designated Requirement) |
| `codesign -dv` | `Identifier=dev.onot.desktop`, Hardened Runtime, `Sealed Resources version=2`, Team ID set |
| `spctl --assess --type execute` | **accepted** (`source=Developer ID`) |
| `stapler validate` | No staple ticket (notarization not performed) |

## Build notes for maintainers

- No application source changes were required; unsigned release matches `electron/electron-builder.yml` (`identity: null`).
- Signing from a cloud-synced working copy may need `xattr -cr` on `dist/onot-sidecar` and `frontend/dist`, and an output path outside the sync folder (e.g. `/tmp/...`).
- Official releases still need organization **Developer ID**, workflow secrets, and notarization/staple as discussed in #148.
