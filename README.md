# KamRos build runner

Public GitHub Actions runner for the **private** KamRos Android source repository.

This repository intentionally contains **no application source code** — only this
workflow and documentation. The private source is checked out at build time into a
temporary `source/` directory with a short-lived, read-only token, compiled, and the
resulting APK is uploaded as an Actions artifact. Nothing from the private repository
is committed here.

## Private source

- Repository: `sketchnoobdeveloper/KamRos` (**private** — visibility never changes)
- Default branch: `main`
- Toolchain discovered at build time from the checkout (no hardcoded Java/Gradle/SDK assumptions)

## One-time setup (manual, by the repository owner)

1. Create a **fine-grained personal access token**:
   - GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens → **Generate new token**
   - Token name: `kamros-build-readonly`
   - Resource owner: `sketchnoobdeveloper`
   - Repository access: **Only select repositories** → `KamRos`
   - Permissions → Repository permissions → **Contents: Read-only**
   - No other permissions (Actions, Metadata, etc. left at defaults)
   - Generate and copy the token.

2. Add it as a repository secret here:
   - `kamros-build` → Settings → Secrets and variables → Actions → **New repository secret**
   - Name: `PRIVATE_REPO_TOKEN`
   - Value: the token from step 1

The token is never written to a file, printed in logs, or stored in this repository.
Checkout uses `persist-credentials: false`.

## Run a build

Actions → **Build KamRos (private source)** → **Run workflow**.

Optional input `source_ref` selects a branch, tag, or commit in the private source
repository (defaults to `main`). For example, to build a feature branch enter its name.

The workflow:

1. verifies the secret exists,
2. checks out the private source with the read-only token,
3. detects Java version, Gradle root/version (wrapper or AGP-derived), `compileSdk`,
   optional NDK/CMake, the Android application module and the assemble task,
4. installs the required Android SDK packages,
5. writes placeholder-only `local.properties` (no real API keys or secrets),
6. runs `:app:assembleDebug`,
7. verifies and uploads the APK(s) as the `kamros-debug-apks` artifact.

## Download

Open the completed run → **Artifacts** → `kamros-debug-apks`. Artifacts are retained
for 14 days and are never published as a GitHub Release.
