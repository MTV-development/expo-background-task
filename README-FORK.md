# @mtv-development/expo-background-task

## Custom Fork of expo-background-task v1.0.8

This is a fork of `expo-background-task` from the Expo SDK, created to solve a critical issue where the published npm package ships with a precompiled AAR (Android Archive) that contains outdated code.

### Why This Fork Exists

**The Problem:**
- The official `expo-background-task@1.0.8` source code correctly uses `NetworkType.NOT_REQUIRED` (line 94 of BackgroundTaskScheduler.kt)
- However, the package ships with a precompiled AAR at `local-maven-repo/` containing old compiled bytecode
- Gradle uses the precompiled AAR instead of compiling from source
- The AAR has the old `NetworkType.CONNECTED` constraint, requiring network for background tasks
- Source code patches with `patch-package` don't work because they only patch source, not the AAR

**The Solution:**
- Extract the package from Expo monorepo
- Remove the `local-maven-repo/` directory with precompiled AAR
- Force Gradle to compile from source (which already has the correct `NetworkType.NOT_REQUIRED`)

### Changes from Original

1. **Removed precompiled AAR** - Deleted `local-maven-repo/` directory
2. **Updated package name** - `@mtv-development/expo-background-task`
3. **Updated version** - `1.0.8-custom.1`
4. **Updated repository URLs** - Point to this fork

### Key Source Code

The important line in `android/src/main/java/expo/modules/backgroundtask/BackgroundTaskScheduler.kt:94`:

```kotlin
val constraints = Constraints.Builder()
  .setRequiredNetworkType(NetworkType.NOT_REQUIRED)  // ← No network required!
  .build()
```

This was ALREADY correct in the source code - we just needed Gradle to actually use it!

### Installation

In your project's `package.json`:

```json
{
  "dependencies": {
    "expo-background-task": "github:MTV-development/expo-background-task#main"
  }
}
```

Or after pushing to npm:

```json
{
  "dependencies": {
    "@mtv-development/expo-background-task": "^1.0.8-custom.1"
  }
}
```

Then run:

```bash
npm install
npx expo prebuild --clean
```

### Verification

After installation, you can verify WorkManager no longer requires network:

```bash
# Install APK on device
adb install path/to/app.apk

# Check WorkManager job constraints
adb shell dumpsys jobscheduler | grep -A40 "your-package.*SystemJobService"

# Should show:
# Required constraints: TIMING_DELAY [0x80000000]
# (NO CONNECTIVITY constraint!)
```

### Compatibility

- **Expo SDK**: 53+
- **expo-task-manager**: ~14.0.7
- **Android**: API 21+ (Android 5.0+)
- **iOS**: iOS 13+

### Maintenance

This fork tracks `expo-background-task` v1.0.8. If Expo publishes v1.0.9+ without the precompiled AAR issue, we can switch back to the official package.

### License

MIT (same as original Expo package)

### Credits

Original package by 650 Industries, Inc. (Expo team)
Fork maintained for MTV Development projects

### Links

- Original package: https://github.com/expo/expo/tree/main/packages/expo-background-task
- Expo docs: https://docs.expo.dev/versions/latest/sdk/background-task/
- This fork: https://github.com/MTV-development/expo-background-task
