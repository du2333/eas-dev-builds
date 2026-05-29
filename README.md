# expo-build-runner

Minimal GitHub Actions runner for Android Expo debug builds.

You write code in your Expo app repository. This repository acts as a manual
Android build button and uploads the APK as a GitHub Actions artifact.

## How It Works

1. Open the `Build Android Dev APK` workflow in GitHub Actions.
2. Click `Run workflow`.
3. Enter an Expo app repository in `owner/repo` format.
4. Optionally enter a branch, tag, or commit SHA.
5. Optionally enter the Expo app path inside the repository.
6. Optionally enter build environment variables as a JSON object.
7. The workflow checks out that repository ref, or the default branch if left blank.
8. It installs dependencies from the committed root lockfile.
9. It runs an Expo CLI Android debug build in the app path.
10. It uploads the APK as an artifact.

## Required Secrets

Add these repository secrets to this runner repository:

| Secret | Purpose |
| --- | --- |
| `APP_REPO_READ_TOKEN` | Fine-grained GitHub PAT with read-only `Contents` access to the Expo app repo. |

For `APP_REPO_READ_TOKEN`, use a fine-grained personal access token scoped only
to the app repositories you want to build.

## Expo App Requirements

Each Expo app repo should include:

- `package.json`
- exactly one supported lockfile:
  - `bun.lock`
  - `pnpm-lock.yaml`
  - `yarn.lock`
  - `package-lock.json`
- `app.json` or `app.config.*`

For a development client build, the app should include `expo-dev-client`.
If the app does not have an `android` directory, Expo CLI will generate it with
prebuild before compiling.

## Manual Run

From GitHub:

```txt
Actions -> Build Android Dev APK -> Run workflow
```

Or with GitHub CLI:

```sh
gh workflow run build-android-dev.yml \
  -f app_repo=YOUR_NAME/YOUR_EXPO_APP
```

To build a specific branch, tag, or commit:

```sh
gh workflow run build-android-dev.yml \
  -f app_repo=YOUR_NAME/YOUR_EXPO_APP \
  -f ref=feature/my-branch
```

For a monorepo where the Expo app lives under `apps/mobile`:

```sh
gh workflow run build-android-dev.yml \
  -f app_repo=YOUR_NAME/YOUR_MONOREPO \
  -f app_path=apps/mobile
```

To pass public build-time environment variables:

```sh
gh workflow run build-android-dev.yml \
  -f app_repo=YOUR_NAME/YOUR_EXPO_APP \
  -f build_env='{"EXPO_PUBLIC_API_URL":"https://api.example.com"}'
```

## Notes

- The workflow builds the target repository's default branch latest commit unless `ref` is set.
- Dependencies are installed from the repository root, then Expo CLI runs inside `app_path`.
- `build_env` is for non-secret build-time values. `EXPO_PUBLIC_*` values are bundled into the app.
- The workflow runs `npx expo prebuild --platform android --non-interactive` when needed, then builds with Gradle `assembleDebug`.
- The Android platform is fixed.
- The APK artifact is retained for 7 days.
- GitHub Actions artifacts require a signed-in GitHub user with repository read access.
- `bun.lockb` is intentionally not supported.
