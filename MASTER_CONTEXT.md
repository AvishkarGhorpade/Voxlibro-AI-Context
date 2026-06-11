# VoxLibro — Master Context Document
> Last updated: Session 1 | Hand this to every AI agent before they write a single line of code.

---

## 1. What Is VoxLibro?

VoxLibro is a **Text-to-Speech Android app** that converts any text or PDF into high-quality audio files the user can listen to anytime — online or offline. Target users: students, professionals, commuters, people with reading difficulties, non-native readers.

**Core value proposition:** Paste any text → hear it in a natural neural voice → save it to your library → listen anywhere, even offline.

---

## 2. Tech Stack

| Layer | Technology |
|---|---|
| Language | Kotlin |
| UI | Jetpack Compose (Material 3) |
| Architecture | Single Activity (`MainActivity`) + Composable screens |
| State | `StateFlow` + `collectAsState()` |
| Storage | DataStore Preferences (`UserPreferencesManager`) |
| Audio playback | `AudioPlayerManager` (wraps `MediaPlayer`) |
| TTS online | `CustomTtsService` — calls two Render-hosted APIs |
| TTS offline | Android built-in `TextToSpeech` |
| PDF | PDFBox (`PdfProcessor`) |
| Dependency injection | None — manual constructor injection |
| Min SDK | 26 (Android 8) |
| Target SDK | 34 |
| Build system | Gradle (Kotlin DSL) |

---

## 3. Package Structure

```
com.avishkar.voxlibro/
├── MainActivity.kt                  — single activity, hosts all screens
├── StatsScreen.kt                   — usage statistics screen
├── audio/
│   ├── AudioPlayerManager.kt        — MediaPlayer wrapper, StateFlow state
│   ├── AudioFileManager.kt          — temp/final file management
│   ├── CustomTtsService.kt          — online TTS, dual API with fallback
│   └── TextToSpeechManager.kt       — orchestrates online/offline TTS
├── data/
│   └── UserPreferencesManager.kt    — DataStore: all user prefs + streak + stats
├── pdf/
│   └── PdfProcessor.kt              — PDF text extraction + online audio conversion
├── review/
│   └── InAppReviewManager.kt        — Google Play In-App Review API wrapper
├── ui/
│   ├── MainScreen.kt                — main convert screen
│   ├── AudioPlayerScreen.kt         — full-screen audio player
│   ├── SavedFilesScreen.kt          — library of saved audio files
│   ├── VoicePickerSheet.kt          — voice selection bottom sheet with preview
│   ├── SettingsBottomSheet.kt       — settings (mode, usage, widget, share)
│   ├── OnboardingScreen.kt          — 3-slide first-launch onboarding
│   ├── TtsModeDialog.kt             — online/offline picker (first launch only)
│   ├── StreakCelebrationOverlay.kt  — streak milestone animation
│   ├── UpdateBanner.kt              — in-app update banner
│   └── theme/
│       └── VoxLibroTheme.kt
├── update/
│   └── InAppUpdateManager.kt        — Google Play in-app update
├── util/
│   ├── NetworkUtils.kt              — internet connectivity check
│   └── ShareUtils.kt               — share audio file intent
└── widget/
    └── VoxLibroWidget.kt            — home screen widget
```

---

## 4. Brand & Design System

### Colors
```kotlin
Green          = Color(0xFF1DB954)   // primary accent — Spotify green
GreenDim       = Color(0x1A1DB954)   // 10% alpha green for backgrounds
Background     = Color(0xFF0D0D0D)   // near-black app background
CardBg         = Color(0xFF1A1A1A)   // card surfaces
CardBgAlt      = Color(0xFF222222)   // elevated card surfaces
Divider        = Color(0xFF242424)   // subtle dividers
TextPrimary    = Color(0xFFFFFFFF)   // primary text
TextSecondary  = Color(0xFF888888)   // secondary/hint text
TextTertiary   = Color(0xFF444444)   // disabled/label text
AmberWarning   = Color(0xFFFFA726)   // warnings
RedDanger      = Color(0xFFE53935)   // errors/destructive
```

### Typography
- Bold headers: `fontSize = 19–22.sp, fontWeight = FontWeight.Bold`
- Body text: `fontSize = 13–15.sp`
- Labels/hints: `fontSize = 10–12.sp`
- Letter spacing: tighten headers with `letterSpacing = (-0.3).sp`

### Shape language
- Cards: `RoundedCornerShape(16.dp)` standard, `14.dp` compact
- Chips: `RoundedCornerShape(12.dp)`
- Pills/badges: `RoundedCornerShape(99.dp)`
- Circles: `CircleShape`

### Spacing rhythm
- Section gaps: `16–20.dp`
- Card internal padding: `14–16.dp horizontal, 12–14.dp vertical`
- Element gaps: `8–12.dp`

---

## 5. Online TTS Architecture

```
User taps Convert
       ↓
TextToSpeechManager.synthesizeToFiles()
       ↓
  [Online mode?]
  YES → CustomTtsService.synthesize()
           ↓
        Ping PRIMARY API (8s timeout)
        Ping FALLBACK API (8s timeout)
           ↓
        Both unreachable?
          → Check NetworkUtils.isOnline()
              → false → NO_INTERNET error
              → true  → BOTH_APIS_FAILED error
           ↓
        Try PRIMARY API (voxlibro.onrender.com)
           ↓ fails
        Try FALLBACK API (voxlibro-api.onrender.com)
           ↓ fails
        BOTH_APIS_FAILED
  NO  → Android TextToSpeech (offline)
```

### API Timeouts
- Ping: 8 seconds
- Connect: 10 seconds
- Read: 40 seconds

### Error Codes
| Code | Meaning | User action |
|---|---|---|
| `NO_INTERNET` | Device has no internet | Auto-switches to offline |
| `BOTH_APIS_FAILED` | Servers unreachable (Render cold start) | Show error, stay in online mode |
| `CHAR_LIMIT_DAILY` | 20,000 char/day limit hit | Show error, suggest offline |
| `CHAR_LIMIT_REQUEST` | Single request > 5,000 chars | Show error, ask to shorten |
| `LONG_TEXT` | Warning only, continues | Toast warning |

---

## 6. DataStore Keys (UserPreferencesManager)

```kotlin
KEY_SPEED            = floatPreferencesKey("tts_speed")
KEY_PITCH            = floatPreferencesKey("tts_pitch")
KEY_ONLINE_MODE      = booleanPreferencesKey("tts_online_mode")
KEY_LANGUAGE         = stringPreferencesKey("tts_language")
KEY_FIRST_LAUNCH     = booleanPreferencesKey("first_launch")        // TtsModeDialog shown once
KEY_SELECTED_VOICE   = stringPreferencesKey("selected_voice")
KEY_DAILY_CHARS      = intPreferencesKey("daily_chars_used")
KEY_DAILY_DATE       = stringPreferencesKey("daily_chars_date")
KEY_TOTAL_CHARS      = longPreferencesKey("total_chars_used")
KEY_TOTAL_FILES      = intPreferencesKey("total_files_created")
KEY_TOTAL_MINUTES    = longPreferencesKey("total_minutes_listened")
KEY_ONBOARDING_DONE  = booleanPreferencesKey("onboarding_done")     // Onboarding shown once
KEY_STREAK_COUNT     = intPreferencesKey("streak_count")
KEY_STREAK_LAST_DATE = stringPreferencesKey("streak_last_date")
```

---

## 7. 12 Supported Voices

| Key | Display Name |
|---|---|
| `en-US-female` | 🇺🇸 American Female |
| `en-US-male`   | 🇺🇸 American Male |
| `en-GB-female` | 🇬🇧 British Female |
| `en-GB-male`   | 🇬🇧 British Male |
| `en-IN-female` | 🇮🇳 Indian Female |
| `en-IN-male`   | 🇮🇳 Indian Male |
| `en-AU-female` | 🇦🇺 Australian Female |
| `en-AU-male`   | 🇦🇺 Australian Male |
| `hi-female`    | 🇮🇳 Hindi Female |
| `hi-male`      | 🇮🇳 Hindi Male |
| `mr-female`    | 🇮🇳 Marathi Female |
| `mr-male`      | 🇮🇳 Marathi Male |

Voice sample files live at `res/raw/preview_<key_with_underscores>.mp3`
Example: `preview_en_us_female.mp3`

---

## 8. App Flows

### First Install
```
Splash → OnboardingScreen (3 slides, one-time) → TtsModeDialog (one-time) → MainScreen
```

### Normal Open
```
Splash → MainScreen
```

### Conversion Flow
```
MainScreen → type/paste text → (optional: pick voice from VoicePickerSheet)
→ tap Convert → FileNameDialog → TextToSpeechManager → saved to VoxLibro/ folder
→ StreakCelebrationOverlay (if new day) → file appears in SavedFilesScreen
```

### Playback Flow
```
SavedFilesScreen → tap play → AudioPlayerScreen
(vinyl disc, seek bar, speed presets: Study/Normal/Commute/Speed Run, sleep timer)
```

---

## 9. Key Constraints

- **No ViewModel** — all state is in `MainActivity.MainContent()` composable
- **No DI framework** — everything manually instantiated in `onCreate`
- **Single DataStore instance** — declared as `private val Context.dataStore` top-level extension in `UserPreferencesManager.kt` only — never duplicate this
- **File storage** — `context.getExternalFilesDir(null)/VoxLibro/` for MP3/WAV
- **Render free tier** — both APIs cold-start in 50–90s; ping before full request
- **`adjustResize`** — required in `AndroidManifest.xml` for keyboard IME button to work
- **`KEY_FIRST_LAUNCH`** — read via `.first()` not `collectAsState(initial=true)` to prevent dialog flashing on every open

---

## 10. Build Dependencies (key ones)

```kotlin
// Compose
implementation("androidx.compose.ui:ui")
implementation("androidx.compose.material3:material3")
implementation("androidx.activity:activity-compose")

// DataStore
implementation("androidx.datastore:datastore-preferences")

// Play
implementation("com.google.android.play:review:2.0.1")
implementation("com.google.android.play:review-ktx:2.0.1")
implementation("com.google.android.play:app-update-ktx")

// PDF
implementation("com.tom_roush:pdfbox-android")

// Splash
implementation("androidx.core:core-splashscreen")
```
