# Deployment Steps for expo-background-task Fork

## Current Status ✅

The forked package is ready in: `C:\git\expo-background-task-custom\`

- ✅ Package extracted from node_modules
- ✅ Precompiled AAR removed
- ✅ Source code verified (NetworkType.NOT_REQUIRED at line 94)
- ✅ package.json updated with new name and version
- ✅ Git repository initialized and pushed to GitHub
- ✅ README-FORK.md created explaining the fork
- ✅ Prepare script removed for Windows compatibility
- ✅ Successfully installed in app from GitHub

## Next Steps (Manual Actions Required)

### Step 1: Create GitHub Repository

1. Go to https://github.com/organizations/MTV-development/repositories/new
2. Repository settings:
   - **Name**: `expo-background-task`
   - **Description**: "Custom fork of expo-background-task v1.0.8 without network requirement (compiles from source)"
   - **Visibility**: Private ✓
   - **DO NOT** initialize with README, .gitignore, or license
3. Click "Create repository"

### Step 2: Push to GitHub

```bash
cd /c/git/expo-background-task-custom

# Add remote (use the URL from your created repo)
git remote add origin https://github.com/MTV-development/expo-background-task.git

# Rename branch to main
git branch -M main

# Push to GitHub
git push -u origin main
```

### Step 3: Update Your App to Use the Fork

In your app's directory (e.g., `/path/to/your/app`):

```bash
# Remove old package and patches
npm uninstall expo-background-task
rm -rf patches/expo-background-task*

# Install from GitHub
npm install --save github:MTV-development/expo-background-task#main

# Clean and rebuild
cd android && ./gradlew.bat clean && cd ..
npm run local:build-release
```

### Step 4: Update package.json

The `package.json` will automatically update to:

```json
{
  "dependencies": {
    "expo-background-task": "github:MTV-development/expo-background-task#main"
  }
}
```

### Step 5: Clean Build and Test

```bash
# Clean gradle cache
cd android && ./gradlew.bat clean && cd ..

# Build release APK
npm run local:build-release

# Install on device
adb uninstall <your.package.name>
adb install -r "path/to/your-app.apk"

# Start app
adb shell am start -n <your.package.name>/.MainActivity

# Wait 5 seconds, then check WorkManager constraints
sleep 5
adb shell dumpsys jobscheduler | grep -A40 "<your-package>.*SystemJobService"
```

### Step 6: Verify Success

Expected output from dumpsys should show:

```
Required constraints: TIMING_DELAY [0x80000000]
```

**NO CONNECTIVITY constraint should be present!**

## Alternative: Test Locally First (Before Pushing)

If you want to test before pushing to GitHub:

```bash
cd /path/to/your/app

# Install from local directory
npm install --save file:../expo-background-task-custom

# Build and test
npm run local:build-release
```

This installs directly from your local fork without needing GitHub.

## Troubleshooting

### If You See CONNECTIVITY Constraint

1. Check that the fork compiled from source:
   ```bash
   # The AAR should NOT exist in node_modules
   ls node_modules/expo-background-task/local-maven-repo/
   # Should show: No such file or directory
   ```

2. Verify source code:
   ```bash
   grep "NetworkType" node_modules/expo-background-task/android/src/main/java/expo/modules/backgroundtask/BackgroundTaskScheduler.kt
   # Should show: .setRequiredNetworkType(NetworkType.NOT_REQUIRED)
   ```

3. Clean gradle completely:
   ```bash
   cd android
   ./gradlew.bat clean
   rm -rf .gradle build app/build
   cd ..
   npm run local:build-release
   ```

### If Installation Fails

Make sure you have access to the GitHub repository:
- Repository must be created in MTV-development organization
- Your GitHub account must have access to the organization
- Use GitHub personal access token if needed:
  `npm config set //npm.pkg.github.com/:_authToken YOUR_TOKEN`

## Success Criteria

✅ Package installs from GitHub
✅ No precompiled AAR in node_modules
✅ Build completes successfully
✅ APK installs on device
✅ WorkManager job created
✅ WorkManager has ONLY TIMING_DELAY constraint (no CONNECTIVITY)
✅ Background tasks execute without network

## Files to Commit in Your App

After successful testing, commit these changes to your app repo:

```
Modified:
- package.json (updated dependency to github:MTV-development/expo-background-task#main)
- package-lock.json (updated dependency lock)

Deleted:
- patches/expo-background-task+1.0.8.patch (no longer needed!)
```

## Maintaining the Fork

If Expo releases expo-background-task v1.0.9+:

1. Check if they fixed the AAR issue:
   ```bash
   npm view expo-background-task@1.0.9 dist.tarball
   # Download and check for local-maven-repo/
   ```

2. If fixed, switch back to official package:
   ```bash
   npm uninstall expo-background-task
   npm install expo-background-task@^1.0.9
   ```

3. If not fixed, update your fork:
   ```bash
   cd /c/git/expo-background-task-custom
   # Copy new version from node_modules
   # Remove AAR
   # Update version to 1.0.9-custom.1
   # Commit and push
   ```

## Contact

For issues with the fork, open an issue at:
https://github.com/MTV-development/expo-background-task/issues
