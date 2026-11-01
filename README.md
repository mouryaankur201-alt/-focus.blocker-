# FocusBlocker 🛡️
> **Strict Instagram Reels & YouTube Shorts Blocker for Android**  
> Protect your attention, reclaim your focus, and eliminate mindless doomscrolling directly within official mobile apps.

---

## 💡 Overview

**FocusBlocker** is a native, open-source Android application that uses Android's **Accessibility Service** to detect and immediately block short-form vertical video feeds (**Instagram Reels** and **YouTube Shorts**) in real-time.

Unlike browser extensions or DNS blockers, FocusBlocker runs at the OS level to enforce strict boundaries inside the **official Android applications**.

### 🌟 Key Features
- **Strict Content Blocking**: Detects the moment a Reel or Short begins playing and immediately performs a back action (`GLOBAL_ACTION_BACK`).
- **Strict Home Fallback**: If a Reel/Short was opened directly via a deep link or notification where "Back" does not exit, the app automatically navigates to the Home screen (`GLOBAL_ACTION_HOME`) after 2 failed attempts to ensure you never get stuck.
- **Granular Toggles**: Enable or disable blocking for Instagram and YouTube independently.
- **Time Saved Dashboard**: Live counter tracking how many Reels and Shorts have been blocked and estimating hours and minutes of your life saved.
- **Debounced Inspection**: Intelligent cooldown system to prevent CPU drain and infinite loop conflicts.
- **100% Local & Privacy-First**: Requires **ZERO** internet permissions (`android.permission.INTERNET` is not even requested in `AndroidManifest.xml`). No tracking, no analytics, no external network requests.
- **Background Reliability**: Built-in shortcut to whitelist the app from aggressive OEM battery killers (Samsung OneUI, Xiaomi MIUI, Pixel Adaptive Battery).

---

## 🏗️ Architecture & How It Works

```
                     ┌───────────────────────────────┐
                     │   Android Window / UI Event   │
                     │  (YouTube / Instagram Active) │
                     └───────────────┬───────────────┘
                                     │
                                     ▼
                     ┌───────────────────────────────┐
                     │     FocusBlockerService       │
                     │   (AccessibilityService)      │
                     └───────────────┬───────────────┘
                                     │
            ┌────────────────────────┴────────────────────────┐
            ▼                                                 ▼
┌───────────────────────────────┐         ┌───────────────────────────────┐
│     YouTubeShortsDetector     │         │    InstagramReelsDetector     │
├───────────────────────────────┤         ├───────────────────────────────┤
│ Inspects Node Hierarchy:      │         │ Inspects Node Hierarchy:      │
│ • reel_watch_fragment         │         │ • clips_viewer                │
│ • shorts_container            │         │ • clips_video_container       │
│ • reel_player_page_tree       │         │ • reel_viewer / clip_land     │
│ • "Shorts" selected tab       │         │ • "Reels" selected tab        │
│ • "Like this short" buttons   │         │ • "Remix this reel" buttons   │
└───────────────┬───────────────┘         └───────────────┬───────────────┘
                │                                         │
                └────────────────────┬────────────────────┘
                                     │ [Match Confirmed]
                                     ▼
                     ┌───────────────────────────────┐
                     │       Enforcement Engine      │
                     ├───────────────────────────────┤
                     │ 1. Debounce Check (650ms)     │
                     │ 2. Increment Block Counter    │
                     │ 3. Display Brief Toast Alert  │
                     │ 4. GLOBAL_ACTION_BACK         │
                     │    (or GLOBAL_ACTION_HOME)    │
                     └───────────────────────────────┘
```

---

## 📁 Project Structure

```
focusblocker/
├── app/
│   ├── src/
│   │   └── main/
│   │       ├── AndroidManifest.xml                    # Accessibility service declaration
│   │       ├── java/com/focusblocker/app/
│   │       │   ├── MainActivity.kt                    # Dashboard & Settings UI
│   │       │   ├── FocusBlockerService.kt             # Background Accessibility Service
│   │       │   ├── detector/
│   │       │   │   ├── IContentDetector.kt            # Detector contract
│   │       │   │   ├── YouTubeShortsDetector.kt       # YouTube & ReVanced signatures
│   │       │   │   └── InstagramReelsDetector.kt      # Instagram & Lite signatures
│   │       │   └── utils/
│   │       │       └── PreferenceManager.kt           # SharedPreferences & statistics
│   │       └── res/
│   │           ├── layout/activity_main.xml           # Material 3 Dashboard
│   │           ├── values/                            # Colors, strings, themes
│   │           ├── drawable/                          # Vector icons & card backgrounds
│   │           └── xml/accessibility_service_config.xml # Accessibility flags & target packages
│   └── build.gradle                                   # App module configuration
├── build.gradle                                       # Top-level build file
├── settings.gradle                                    # Project settings
├── gradle.properties                                  # JVM & AndroidX settings
└── README.md
```

---

## 🚀 How to Build & Run

### Option 1: Using Android Studio (Recommended)
1. Open **Android Studio**.
2. Select **File > Open** and browse to this directory:
   ```
   .../scratch/focusblocker
   ```
3. Let Gradle sync dependencies.
4. Connect an Android phone (with USB Debugging enabled) or start an Emulator.
5. Click **Run (`Shift + F10`)**.

### Option 2: Using the Command Line (Gradle)
If you have the Android SDK and Java 17 installed:
```bash
# Build the debug APK
./gradlew assembleDebug

# Install on connected device via ADB
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

---

## 📱 How to Enable on Your Phone

1. **Install and Open FocusBlocker**.
2. Tap the **Enable in Settings** button at the top of the screen.
3. You will be redirected to Android's **Accessibility Settings**:
   - Tap **Installed apps** or **Downloaded services**.
   - Find and tap **FocusBlocker Service**.
   - Toggle **Use FocusBlocker Service** to **ON**.
   - Tap **Allow** on the permission prompt.
4. **Disable Battery Optimization**:
   - In the FocusBlocker dashboard, tap **Disable Battery Optimization** so Android does not kill the accessibility listener during deep sleep.
5. **Test it out**:
   - Open YouTube and tap on a Shorts video or navigation tab -> *Watch it immediately kick you back out!*
   - Open Instagram and tap on Reels -> *Instant exit back to your feed or home screen.*

---

## 🔒 Privacy & Permissions
- **No Internet Permission**: The app cannot make network calls or transmit any telemetry.
- **On-Device Only**: Inspects view hierarchies purely in local memory to trigger standard system navigation commands.
- **Free & Open Source**: Safe, transparent, and bloatware-free under the GPL-3.0 License.

---

## ☕ Configuring Your UPI ID (Direct Support)
FocusBlocker includes a native UPI tip integration for Indian users (supporting Google Pay, PhonePe, Paytm, and BHIM).

To receive tips directly to your bank account:
1. Open [`app/src/main/res/values/strings.xml`](file:///data/user/0/com.termux/files/home/.gemini/antigravity-cli/scratch/focusblocker/app/src/main/res/values/strings.xml).
2. Update the `developer_upi_id` string:
   ```xml
   <string name="developer_upi_id">yourupi@okaxis</string>
   <string name="developer_payee_name">Your Name</string>
   ```
3. Whenever users tap "Chai (₹29)" or "Coffee (₹59)", their UPI apps will open directly with your details.

---

## 🌐 Publishing to GitHub & F-Droid

### 1. GitHub Distribution (With Auto-Build APK)
1. Initialize a git repository in this folder and push to your GitHub account:
   ```bash
   git init
   git add .
   git commit -m "Initial release of FocusBlocker"
   git branch -M main
   git remote add origin https://github.com/<your-username>/focusblocker.git
   git push -u origin main
   ```
2. The included GitHub Actions workflow (`.github/workflows/build.yml`) will automatically compile the APK.
3. Go to your repository's **Actions** tab -> Click on the latest workflow run -> Download the ready-to-install **`FocusBlocker-Debug-APK`**!
4. Create a **GitHub Release** and attach the APK for your users.

### 2. Publishing to F-Droid
Because FocusBlocker is 100% open-source, uses standard libraries, and has no proprietary trackers or binary blobs:
1. Push your repository to GitHub or GitLab.
2. Tag a release (e.g. `v1.0.0`).
3. Submit a Merge Request to the [F-Droid Data repository](https://gitlab.com/fdroid/fdroiddata) following their [inclusion policy](https://f-droid.org/docs/Inclusion_Policy/).
4. F-Droid will build the APK directly from your source code and make it available to millions of FOSS users worldwide!

