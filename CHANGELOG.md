 # VoxLibro — Changelog
> Every change made, in reverse chronological order.

---

## Session 1 — Foundation & Polish Sprint

### New Files Created
- `OnboardingScreen.kt` — 3-slide onboarding with progress bar, rich slide content, animated dots
- `InAppReviewManager.kt` — Google Play In-App Review wrapper with pre-warm support
- `VoicePickerSheet.kt` — Dedicated voice picker with pre-generated previews, speed/pitch sliders

### Files Completely Rewritten
- `MainScreen.kt` — Full production redesign:
  - New layout order: header → banners → text input → voice row → controls → buttons
  - Voice selector row between text box and controls (flag + VOICE label + name + chevron)
  - Online pill moved inline into header row with dot indicator
  - Clipboard banner: compact single-row design
  - Error state card: inline, amber tint, specific messages per error code
  - Keyboard convert button: floats above IME using `WindowInsets.ime`
  - `Background = #0D0D0D`, `CardBg = #1A1A1A` for premium dark feel
  - `voiceControlsExpanded = true` by default, collapses on `isBusy`

- `AudioPlayerScreen.kt` — Speed presets added:
  - 4 inline preset chips: Study(0.75×) / Normal(1.0×) / Commute(1.25×) / Speed Run(1.75×)
  - Animated color transitions on active chip
  - "More options" link opens fine-grained speed dialog
  - Sleep timer redesigned as compact pill below controls
  - `statusBarsPadding()` + `navigationBarsPadding()` added
  - Palette aligned with MainScreen (#0D0D0D, #1A1A1A)

- `SavedFilesScreen.kt` — Empty states upgraded:
  - 3 variants: fresh install / library cleared / search no results
  - Each has contextual icon, title, subtitle, action button
  - Action buttons wired to `onBack()` for immediate navigation
  - Feature hint chips on fresh install state

- `SettingsBottomSheet.kt` — Cleaned up:
  - Voice selector section removed (moved to VoicePickerSheet)
  - "Share VoxLibro" row added with pre-written message + Play Store link
  - Reusable `SettingsActionRow` component for widget + share rows
  - Header renamed from "Voice Settings" to "Settings"

- `CustomTtsService.kt` — Error handling overhauled:
  - Server reachability ping (HEAD request, 8s timeout) before full TTS request
  - `NO_INTERNET` vs `BOTH_APIS_FAILED` correctly distinguished
  - Removed redundant pre-flight internet check (was short-circuiting ping logic)
  - Timeouts: connect=10s, read=40s, ping=8s (down from 30s/60s)

- `TextToSpeechManager.kt` — Error propagation fixed:
  - `BOTH_APIS_FAILED` → immediate error + progress reset, no silent offline fallback
  - `NO_INTERNET` → still auto-switches to offline (correct)
  - Offline progress starts at 10% (was 50%, which was confusing)

### Files Modified (surgical changes)

**`UserPreferencesManager.kt`**
- Added `KEY_ONBOARDING_DONE = booleanPreferencesKey("onboarding_done")`
- Added `isOnboardingDone(): Boolean` suspend function
- Added `setOnboardingDone()` suspend function
- Changed constructor to store `applicationContext` to prevent leaks
- All DataStore accesses use `ctx` (applicationContext) instead of `context`

**`MainActivity.kt`**
- Added `InAppReviewManager` — pre-warm on 2nd conversion, show on 3rd
- Added onboarding gate at top of `MainContent()` (null → loading, true → onboarding, false → app)
- Removed custom rate-us dialog (65 lines deleted)
- Added `lastErrorCode` state for inline error card
- Fixed `TtsModeDialog` trigger: `collectAsState(initial=true)` → `prefsManager.isFirstLaunch.first()`
- Added `showVoicePickerSheet` state + `VoicePickerSheet` composable call
- Moved `selectedVoice` declaration to top of `MainContent()` (was inside `if (showSettingsSheet)` block — caused "Unresolved reference" error)
- `lastErrorCode` declared before `LaunchedEffect` that writes to it (fixed ordering bug)
- `lastErrorCode = null` before setting new value (prevents error card flash)
- `lastErrorCode = null` when new conversion starts

---

## Bugs Fixed This Session

| Bug | Fix |
|---|---|
| TtsModeDialog shows on every app open | Read DataStore with `.first()` not `collectAsState(initial=true)` |
| `lastErrorCode` unresolved reference | Moved declaration above the `LaunchedEffect` that uses it |
| `selectedVoice` unresolved in VoicePickerSheet call | Moved `collectAsState()` to top of `MainContent()` |
| Error cards briefly overlapping | `lastErrorCode = null` then set new value (two-step) |
| NO_INTERNET showing as BOTH_APIS_FAILED | Check internet AFTER ping fails, not before |
| Progress stuck at 80% for minutes | Server ping (8s) before full request — fast fail |
| Voice picker button colliding with clipboard banner | Moved voice selector below text input, removed from header |
| `Property must be initialized or be abstract` | Removed duplicate `preferencesDataStore` declaration |
| Convert + Upload PDF buttons overlapping | Wrapped in `Column(spacedBy(10dp))` |

---

## Dependencies Added

```kotlin
// build.gradle.kts
implementation("com.google.android.play:review:2.0.1")
implementation("com.google.android.play:review-ktx:2.0.1")
```

## Resources Added

```
app/src/main/res/raw/
├── preview_en_us_female.mp3
├── preview_en_us_male.mp3
├── preview_en_gb_female.mp3
├── preview_en_gb_male.mp3
├── preview_en_in_female.mp3
├── preview_en_in_male.mp3
├── preview_en_au_female.mp3
├── preview_en_au_male.mp3
├── preview_hi_female.mp3
├── preview_hi_male.mp3
├── preview_mr_female.mp3
└── preview_mr_male.mp3
```
