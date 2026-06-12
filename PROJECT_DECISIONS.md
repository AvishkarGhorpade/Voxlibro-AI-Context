# VoxLibro — Project Decisions
> Every significant decision made, with the reasoning. Read this before changing anything.

---

## Architecture Decisions

### D-001: No ViewModel, State in MainActivity
**Decision:** All UI state lives in `MainActivity.MainContent()` composable.
**Reason:** App started without MVVM and refactoring mid-development adds risk. State hoisting to composable level works correctly for this app size.
**Implication:** Don't introduce ViewModel without a full refactor plan. Keep state in composables.

### D-002: Single DataStore Instance
**Decision:** `preferencesDataStore` delegate declared only in `UserPreferencesManager.kt`.
**Reason:** Kotlin extension properties on the same type in the same package cause "Property must be initialized or be abstract" compile errors if declared twice.
**Implication:** Never add `preferencesDataStore` to any other file. If you need DataStore elsewhere, import from `UserPreferencesManager`.

### D-003: `applicationContext` in UserPreferencesManager
**Decision:** Constructor accepts `Context` but stores `context.applicationContext`.
**Reason:** Prevents Activity context leaks since `UserPreferencesManager` outlives the Activity.
**Implication:** Always use `ctx` (applicationContext) internally, never `context` directly.

### D-004: Read KEY_FIRST_LAUNCH with `.first()` not `collectAsState(initial=true)`
**Decision:** `prefsManager.isFirstLaunch.first()` inside `LaunchedEffect`.
**Reason:** `collectAsState(initial=true)` emits `true` before DataStore loads, causing `TtsModeDialog` to flash on every app open for returning users.
**Implication:** Never use `collectAsState(initial=true)` for one-time-show dialogs. Always suspend and read the real value.

---

## TTS / API Decisions

### D-005: Server Ping Before Full Request
**Decision:** `isServerReachable()` HEAD request (8s timeout) before full TTS API call.
**Reason:** Render free tier cold-starts take 50–90s. Without the ping, users wait the full `readTimeout` (40s × 2 APIs = 80s) before seeing any error.
**Implication:** Adds ~8–16s to error detection but saves ~64–80s of silent hanging.

### D-006: NO_INTERNET vs BOTH_APIS_FAILED Distinction
**Decision:** Check `NetworkUtils.isOnline()` AFTER both pings fail, not before.
**Reason:** If we check internet first, offline users always get `NO_INTERNET` even before we try the servers. Checking after ping fails correctly distinguishes "your internet is down" vs "our servers are down."
**Implication:** The ping → internet check order must not be reversed.

### D-007: BOTH_APIS_FAILED Does NOT Auto-Switch to Offline
**Decision:** When both APIs fail (servers down), show error and stop. Do NOT silently fallback to offline TTS.
**Reason:** User is in online mode for quality reasons. Silently producing a different-sounding voice is worse UX than showing a clear error. User can manually switch to offline if they want.
**Exception:** `NO_INTERNET` DOES auto-switch to offline because the device has no connection at all — online mode is simply impossible.

### D-008: Text Splitting for Online TTS
**Decision:** Split by sentences first (regex on `.!?`), then pack into ≤5000 char chunks.
**Reason:** Splitting mid-sentence produces unnatural audio. Sentence-boundary splitting keeps chunks coherent.
**Implication:** Very long single sentences (> 5000 chars) fall back to word-boundary splitting.

---

## UI/UX Decisions

### D-009: Voice Picker is a Separate Sheet
**Decision:** Voice selection moved from `SettingsBottomSheet` to `VoicePickerSheet`.
**Reason:** Settings sheet was overcrowded. Voice selection is a frequent, high-importance action that deserves its own focused UI.
**Implication:** `SettingsBottomSheet` handles mode/widget/share. `VoicePickerSheet` handles voice selection + preview.

### D-010: Pre-Generated Voice Samples (res/raw)
**Decision:** 12 MP3 preview samples bundled in `res/raw/` instead of generating on-demand.
**Reason:** On-demand generation via API takes 5–15 seconds. Users need instant feedback when previewing voices. Pre-generated = instant playback, works offline.
**Implication:** App size increases by ~12 × sample_size. Keep samples short (5–8 seconds). Filename convention: `preview_<voice_key_with_underscores>.mp3`.

### D-011: Speed Presets as Inline Chips (not dialog)
**Decision:** 4 speed preset chips (Study/Normal/Commute/Speed Run) shown directly on the player screen.
**Reason:** Opening a dialog for something users change frequently (speed) adds unnecessary friction. Inline chips = one tap.
**Fine control:** "More options" link still opens the full speed picker dialog for 0.5×–2.0× range.

### D-012: Voice Selector Row Position in MainScreen
**Decision:** Voice selector row sits between text input card and voice controls card.
**Reason:** Previously placed below the header, it collided visually with the clipboard banner. Between the cards it has a clear, permanent, predictable position.
**Implication:** Only visible when `!isBusy && isOnlineMode`.

### D-013: Onboarding Shows Before TtsModeDialog
**Decision:** Onboarding → TtsModeDialog → MainScreen for brand new users.
**Reason:** TtsModeDialog requires the user to understand online vs offline. Onboarding teaches them what the app does first, making the mode choice meaningful.
**Implementation:** `KEY_ONBOARDING_DONE` checked first; `KEY_FIRST_LAUNCH` checked after onboarding completes.

### D-014: Error Card Instead of Just Snackbar
**Decision:** Persistent inline error card shown when conversion fails, in addition to snackbar.
**Reason:** Snackbar auto-dismisses in 4s — if user missed it, they have no idea why conversion stopped. Inline card stays until dismissed. Snackbar still shows for immediate feedback.

### D-015: VoiceControlsCard Collapsed by Default → Changed to Expanded
**Decision:** Voice controls expanded by default, collapses when conversion starts.
**Reason:** User feedback — collapsing by default hid important controls. Users need to see and adjust speed/pitch easily. Auto-collapse during conversion keeps the processing view clean.

---

## Empty State Decisions

### D-016: 3 Library Empty State Variants
**Decision:** Three distinct states in `SavedFilesScreen`:
1. Fresh install (never converted) → full illustrated state + CTA button
2. Library cleared (had files, deleted all) → lighter state + CTA button
3. Search no results → search-specific state + clear search button
**Reason:** Generic "empty" states are lazy. Context-specific states guide the user to the right action.

---

## Rating / Review Decisions

### D-017: Google Play In-App Review (not custom dialog)
**Decision:** Replace custom rate-us dialog with `ReviewManager` from Google Play In-App Review API.
**Reason:** Native sheet = higher conversion, no redirect out of app, looks trustworthy.
**Pre-warm:** Called on 2nd conversion so token is ready for instant display on 3rd.
**Implication:** Google throttles display — calling `requestReview()` doesn't guarantee the sheet shows. Never make decisions based on whether it showed.

---

## Share Decisions

### D-018: Share App via Intent.ACTION_SEND
**Decision:** "Share VoxLibro" in Settings opens a standard Android share sheet with Play Store link.
**Reason:** Simple, standard, works everywhere. No custom analytics needed at this stage.
**Message format:** Pre-written text with emoji, app description, and Play Store URL using `context.packageName`.

---

## Notification Decisions

### D-019: Notification Channel Strategy
**Decision:** Three channels with distinct importance levels: `IMPORTANCE_LOW` for playback and updates, `IMPORTANCE_DEFAULT` for conversion complete.
**Reason:**
- Playback channel must be silent — posting a sound/vibration notification while audio is actively playing interrupts the listening experience.
- Conversion channel uses `IMPORTANCE_DEFAULT` (sound + heads-up) because the user may have left the app; they need a clear alert that their file is ready.
- Updates channel is informational and should not interrupt the user.
**Implementation:** `object NotificationChannelManager` in `util/` — singleton, no constructor arguments. Called once in `MainActivity.onCreate()` immediately after `super.onCreate()` and before any notification-posting code. Android deduplicates on repeated launches.
**Single source of truth for channel IDs:**
- `NotificationChannelManager.CHANNEL_PLAYBACK`   → `"voxlibro_playback"`
- `NotificationChannelManager.CHANNEL_CONVERSION` → `"voxlibro_conversion"`
- `NotificationChannelManager.CHANNEL_UPDATES`    → `"voxlibro_updates"`
**Implication:** Every `NotificationCompat.Builder` call in the codebase MUST reference one of the three constants above. Never hardcode a channel ID string in any other file.

### D-020: Channel Registration Ordering in onCreate
**Decision:** `NotificationChannelManager.createChannels(this)` is the FIRST call after `super.onCreate()`, before `installSplashScreen()` is moved, before the `POST_NOTIFICATIONS` permission request, and before any service or manager is instantiated.
**Reason:** Any code path that runs between `super.onCreate()` and the channel registration call could theoretically post a notification to a non-existent channel. On Android 8+, notifications posted to unregistered channels are silently dropped. Registering first eliminates this race.
**Exception:** `installSplashScreen()` is called before `super.onCreate()` per Jetpack SplashScreen API requirements — that is a platform constraint, not a violation of this rule.