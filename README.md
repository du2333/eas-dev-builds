# expo-build-runner

Minimal GitHub Actions runner for Android Expo development builds.

You write code in your Expo app repository. This repository acts as a manual
Android build button and uploads the APK as a GitHub Actions artifact.

## How It Works

1. Open the `Build Android Dev APK` workflow in GitHub Actions.
2. Click `Run workflow`.
3. Enter an Expo app repository in `owner/repo` format.
4. The workflow checks out that repository's default branch.
5. It installs dependencies from the committed lockfile.
6. It runs an EAS local Android development build.
7. It uploads `dist/app.apk` as an artifact.

## Required Secrets

Add these repository secrets to this runner repository:

| Secret | Purpose |
| --- | --- |
| `APP_REPO_READ_TOKEN` | Fine-grained GitHub PAT with read-only `Contents` access to the Expo app repo. |
| `EXPO_TOKEN` | Expo access token used by EAS CLI in CI. |

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
- `eas.json`
- `app.json` or `app.config.*`

The workflow expects an EAS build profile named `development`. For APK output,
the app repo's `eas.json` should set Android `buildType` to `apk`.

Example:

```json
{
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      }
    }
  }
}
```

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

## Notes

- The workflow builds the target repository's default branch latest commit.
- The EAS profile is fixed to `development`.
- The Android platform is fixed.
- The APK artifact is retained for 7 days.
- `bun.lockb` is intentionally not supported.
