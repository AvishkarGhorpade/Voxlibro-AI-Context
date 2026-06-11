# VoxLibro — Current State
> Snapshot of every file's status as of Session 1. Update this after every session.

---

## Screen / File Status

| File | Status | Last Changed | Notes |
|---|---|---|---|
| `MainActivity.kt` | ✅ Production ready | Session 1 | Onboarding gate, InAppReview, error state, VoicePicker wired |
| `MainScreen.kt` | ✅ Production ready | Session 1 | Full redesign, voice selector row, keyboard convert button, error card |
| `OnboardingScreen.kt` | ✅ Production ready | Session 1 | 3 slides, progress bar, animated dots, feature content cards |
| `AudioPlayerScreen.kt` | ✅ Production ready | Session 1 | Speed presets (Study/Normal/Commute/Speed Run), sleep timer pill |
| `SavedFilesScreen.kt` | ✅ Production ready | Session 1 | 3 empty state variants, multi-select, search, sort, rename, share |
| `VoicePickerSheet.kt` | ✅ Production ready | Session 1 | Pre-generated previews, speed+pitch sliders, play/pause per chip |
| `SettingsBottomSheet.kt` | ✅ Production ready | Session 1 | Mode cards, usage bar, widget, share app (removed voice selector) |
| `UserPreferencesManager.kt` | ✅ Production ready | Session 1 | `KEY_ONBOARDING_DONE` added, uses `applicationContext` |
| `CustomTtsService.kt` | ✅ Production ready | Session 1 | Server ping, NO_INTERNET vs BOTH_APIS_FAILED distinction |
| `TextToSpeechManager.kt` | ✅ Production ready | Session 1 | Fast fail on BOTH_APIS_FAILED, no silent offline fallback |
| `InAppReviewManager.kt` | ✅ Production ready | Session 1 | Pre-warm on 2nd conversion, show on 3rd |
| `StatsScreen.kt` | ✅ Exists | Unchanged | Not touched this session |
| `TtsModeDialog.kt` | ✅ Exists | Unchanged | Shown once via `.first()` not `collectAsState(initial=true)` |
| `StreakCelebrationOverlay.kt` | ✅ Exists | Unchanged | |
| `UpdateBanner.kt` | ✅ Exists | Unchanged | |
| `AudioPlayerManager.kt` | ✅ Exists | Unchanged | |
| `AudioFileManager.kt` | ✅ Exists | Unchanged | |
| `PdfProcessor.kt` | ✅ Exists | Unchanged | |
| `VoxLibroWidget.kt` | ✅ Exists | Unchanged | |
| `NetworkUtils.kt` | ✅ Exists | Unchanged | |
| `ShareUtils.kt` | ✅ Exists | Unchanged | |

---

## Resource Files

| Resource | Status | Notes |
|---|---|---|
| `res/raw/preview_en_us_female.mp3` | ✅ Added | Voice preview sample |
| `res/raw/preview_en_us_male.mp3` | ✅ Added | |
| `res/raw/preview_en_gb_female.mp3` | ✅ Added | |
| `res/raw/preview_en_gb_male.mp3` | ✅ Added | |
| `res/raw/preview_en_in_female.mp3` | ✅ Added | |
| `res/raw/preview_en_in_male.mp3` | ✅ Added | |
| `res/raw/preview_en_au_female.mp3` | ✅ Added | |
| `res/raw/preview_en_au_male.mp3` | ✅ Added | |
| `res/raw/preview_hi_female.mp3` | ✅ Added | |
| `res/raw/preview_hi_male.mp3` | ✅ Added | |
| `res/raw/preview_mr_female.mp3` | ✅ Added | |
| `res/raw/preview_mr_male.mp3` | ✅ Added | |

---

## AndroidManifest.xml — Required Settings

```xml
<!-- Activity tag must have: -->
android:windowSoftInputMode="adjustResize"

<!-- Permissions needed: -->
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
<uses-permission android:name="android.permission.VIBRATE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

---

## Known Issues / Bugs

| Issue | Severity | Status |
|---|---|---|
| Error cards briefly overlap when two errors fire in quick succession | Low | Mitigated with null→set pattern |
| Render free tier cold-start still causes ~16s wait before BOTH_APIS_FAILED | Medium | Accepted — ping reduces from 2min to 16s |
| Voice preview pitch/speed not saved between sheet opens | Low | By design — preview is independent of conversion |
| `adjustResize` must be set manually in manifest | Medium | Documented |

---

## What Was NOT Changed This Session

- `StatsScreen.kt` — untouched
- `AudioPlayerManager.kt` — untouched  
- `AudioFileManager.kt` — untouched
- `PdfProcessor.kt` — untouched
- `VoxLibroWidget.kt` — untouched (sleep timer in widget is on roadmap)
- `TtsModeDialog.kt` — untouched (trigger logic fixed in MainActivity)
- `StreakCelebrationOverlay.kt` — untouched
- Widget layout XML files — untouched
- All theme files — untouched

---

## Compilation Requirements

Before building, ensure:
1. `preferencesDataStore` declared in exactly ONE file (`UserPreferencesManager.kt`)
2. All 12 `res/raw/preview_*.mp3` files present
3. `android:windowSoftInputMode="adjustResize"` in manifest
4. Play Review dependency in `build.gradle.kts`:
   ```kotlin
   implementation("com.google.android.play:review:2.0.1")
   implementation("com.google.android.play:review-ktx:2.0.1")
   ```
5. `FakeReviewManager` import available for debug testing:
   ```kotlin
   import com.google.android.play.core.review.testing.FakeReviewManager
   ```
