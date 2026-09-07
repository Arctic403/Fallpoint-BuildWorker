# Fallpoint Build Worker

Public build worker for Fallpoint Android builds.

This repository intentionally contains **no Fallpoint game source, licensed character source files, or private game assets**. Its workflow checks out `Arctic403/Fallpoint` only inside an ephemeral GitHub-hosted runner, builds the universal Android APK, verifies both supported ARM ABIs, and publishes only the completed APK back to the private Fallpoint repository as a prerelease.

## Automatic build flow

The public worker checks private `Fallpoint/main` every 5 minutes. It resolves the exact current commit SHA and compares it with previously published worker prereleases. If that exact commit already has an APK, the run exits before any Android/Gradle work. If the commit is new, the public worker builds it and sends the APK back to Fallpoint automatically.

`workflow_dispatch` remains available for manual rebuilds of any Fallpoint branch, tag, or commit.

```text
Private Arctic403/Fallpoint main changes
        │
        │ public BuildWorker detects a new private commit
        ▼
Public Arctic403/Fallpoint-BuildWorker
        │
        ├─ resolves exact Fallpoint commit SHA
        ├─ skips commits already built
        ├─ ephemeral private checkout
        ├─ JDK 17 / Gradle 9.6 / Android SDK 36
        ├─ NDK 28.2.13676358 / CMake 3.22.1
        ├─ source + portable smoke checks
        ├─ private character cooker
        ├─ universal APK build
        └─ verifies armeabi-v7a + arm64-v8a
        │
        ▼
Private Fallpoint prerelease containing
Fallpoint-universal-debug.apk
```

## Security contract

- Fallpoint source remains private and is never committed to this public repository.
- The source repository is hard-coded to `Arctic403/Fallpoint`.
- `actions/checkout` uses `persist-credentials: false` so the cross-repository token is not left in the checked-out game's git config.
- Raw `.glb`, `.fbx`, `.blend`, texture files, and cooked character intermediates are never uploaded as BuildWorker artifacts.
- The public worker's own `GITHUB_TOKEN` is read-only.
- No public APK artifact is uploaded; completed private builds are returned only to Fallpoint.

## Required secret

The BuildWorker repository must contain the Actions secret:

`FALLPOINT_PRIVATE_TOKEN`

Use a fine-grained GitHub personal access token restricted to **only** `Arctic403/Fallpoint` with:

- **Metadata: Read**
- **Contents: Read and write**

Read access is required to inspect and check out private Fallpoint source. Write access is used only to create the private prerelease and upload the completed APK.

Do not put the token in a workflow input, file, commit, issue, or log.
