# VoxLibro — Changelog
> Every change made, in reverse chronological order.

---

## Session 3 — Notification Channels (Android 8+ / Play Store Compliance)

### New Files Created
- `util/NotificationChannelManager.kt`

### Files Modified
- `MainActivity.kt` — one import added, one call added in `onCreate()`
- `PROJECT_DECISIONS.md` — decisions D-019 and D-020 added

---

**`NotificationChannelManager.kt`** — `object` (singleton) in `com.avishkar.voxlibro.util`:

Three channels registered via `nm.createNotificationChannels()` in a single batched call:

| Constant | ID | Importance | Sound | Badge | Used for |
|---|---|---|---|---|---|
| `CHANNEL_PLAYBACK` | `voxlibro_playback` | LOW | ❌ | ❌ | Ongoing MediaPlayer foreground notification |
| `CHANNEL_CONVERSION` | `voxlibro_conversion` | DEFAULT | ✅ | ✅ | "Your file is ready" alert when app is backgrounded |
| `CHANNEL_UPDATES` | `voxlibro_updates` | LOW | ❌ | ❌ | InAppUpdateManager banners |

`createChannels()` is guarded with `Build.VERSION.SDK_INT < Build.VERSION_CODES.O` — complete no-op on API 25 and below. Android deduplicates channel registration by ID on every subsequent launch.

**Why `IMPORTANCE_LOW` for playback:** Posting a sound/vibration notification while audio is playing would interrupt the spoken audio. `IMPORTANCE_DEFAULT` would do exactly that.

**Why `IMPORTANCE_DEFAULT` for conversion:** Conversion runs in the background. The user needs an audible heads-up that their file is ready; a silent notification would go unnoticed.

---

**`MainActivity.kt`** — two changes only:

1. Import added (line 88):
   ```kotlin
   import com.avishkar.voxlibro.util.NotificationChannelManager
   ```

2. First call in `onCreate()`, before `POST_NOTIFICATIONS` permission request and before any manager is instantiated:
   ```kotlin
   NotificationChannelManager.createChannels(this)
   ```

No composables, no state, no other files touched.

---

**Play Store checklist update:**
- [x] Notification channels (Android 8+) ← **Done this session**

**Follow-up required (not in scope of this session):**
Any existing `NotificationCompat.Builder` calls in `CustomTtsService` or any foreground service must be updated to pass `NotificationChannelManager.CHANNEL_PLAYBACK` (or the relevant constant) as the channel ID. Without this, notifications on Android 8+ will be silently dropped even though channels are now registered.

---


## Session 2 — Dynamic Colors (Material You)

### Files Modified
- `VoxLibroTheme.kt` — Dynamic color support added for Android 12+

**What changed:**
- `VoxLibroTheme()` now resolves the color scheme at runtime:
  - API 31+ → `dynamicDarkColorScheme(context).lockBrandColors()`
  - API < 31 → `VoxLibroDarkColorScheme` (unchanged, same as before)
- New private `ColorScheme.lockBrandColors()` extension: copies the dynamic scheme and replaces every brand-identity role with our design-system tokens. The static scheme and the locked-dynamic scheme produce identical role assignments — the only thing that varies is which dynamic roles are left free (see below).
- `rememberSystemUiController()` + `SideEffect` added: both system bars set to `Color.Transparent`, `darkIcons = false`, `isNavigationBarContrastEnforced = false`. This replaces any previous ad-hoc status bar colour logic.
- `LocalContext.current` + `isSystemInDarkTheme()` imports added; unused `androidx.compose.foundation.isSystemInDarkTheme` not included since the app is always dark.

**What dynamic color is allowed to touch (wallpaper-adaptive):**
- `surfaceTint` — the subtle tint Material 3 applies to elevated surfaces
- `outline` / `outlineVariant` — focus rings and dividers inside Material widgets
- Any role not explicitly overridden in `lockBrandColors()`

**What is locked (brand-identity roles — never change with wallpaper):**
- `primary` / `onPrimary` / `primaryContainer` / `onPrimaryContainer` → `AccentGreen`
- `secondary` / `onSecondary` / `secondaryContainer` / `onSecondaryContainer` → card surface tokens
- `tertiary` / `onTertiary` → `AccentGreenDim`
- All `error` roles → `DangerRed` family
- All `background` / `surface` / `surfaceVariant` roles → dark palette tokens
- `inverseSurface` / `inverseOnSurface` / `inversePrimary` / `scrim`

**Why surfaces are also locked:** All screens use manually-colored composables (`SurfaceCard`, `SurfaceDark` etc.). If dynamic colors changed these M3 roles, wallpaper-tinted Material surfaces would clash with the hardcoded card colors throughout the app.

**No screen files need to change.** All token references (`AccentGreen`, `SurfaceDark`, etc.) remain in `Color.kt` untouched.

### New Dependency Required
```kotlin
// build.gradle.kts
implementation("com.google.accompanist:accompanist-systemuicontroller:0.34.0")
```

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

## Bugs Fixed Session 1

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
implementation("com.google.accompanist:accompanist-systemuicontroller:0.34.0")  // Session 2
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