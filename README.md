# Fallpoint Build Worker

Public build worker for Fallpoint Android builds.

This repository intentionally contains **no Fallpoint game source, licensed character source files, or private game assets**. Its workflow checks out `Arctic403/Fallpoint` only inside an ephemeral GitHub-hosted runner, builds the universal Android APK, verifies both supported ARM ABIs, and can publish only the completed APK back to the Fallpoint repository as a private prerelease.

## Security contract

- Normal builds are manual (`workflow_dispatch`) only.
- No pull-request or fork-triggered workflow is allowed to receive the private-repository credential.
- The source repository is hard-coded to `Arctic403/Fallpoint`.
- `actions/checkout` uses `persist-credentials: false` so the cross-repository token is not left in the checked-out game's git config.
- Raw `.glb`, `.fbx`, `.blend`, texture files, and cooked character intermediates are never uploaded as BuildWorker artifacts.
- The public worker's own `GITHUB_TOKEN` is read-only.
- The completed APK is published back to Fallpoint only when explicitly requested.

## One-time setup before Fallpoint becomes private

Create a fine-grained GitHub personal access token restricted to **only** `Arctic403/Fallpoint`, then add it to this repository under **Settings → Secrets and variables → Actions → New repository secret** with the exact name:

`FALLPOINT_PRIVATE_TOKEN`

Repository permissions for that token:

- **Metadata: Read**
- **Contents: Read and write**

Read access is needed to check out private Fallpoint source. Write access is used only to create the private prerelease and upload the completed APK.

Do not put the token in a workflow input, file, commit, issue, or log.

## Build flow

```text
Private Arctic403/Fallpoint
        │
        │ repository-scoped secret
        ▼
Public Arctic403/Fallpoint-BuildWorker
        │
        ├─ ephemeral checkout
        ├─ JDK 17 / Gradle 9.6 / Android SDK 36
        ├─ NDK 28.2.13676358 / CMake 3.22.1
        ├─ source + portable smoke checks
        ├─ private character cooker
        ├─ universal APK build
        └─ verify armeabi-v7a + arm64-v8a
        │
        ▼
Private Fallpoint prerelease containing completed APK
```

To build, open **Actions → Fallpoint Private Universal Build → Run workflow**, choose the Fallpoint git ref (normally `main`), and choose whether to publish the completed APK back to Fallpoint.
