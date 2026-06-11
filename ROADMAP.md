# VoxLibro — Feature Roadmap
> Priority order based on Play Store impact, user value, and implementation complexity.

---

## Legend
- ✅ Done
- 🔄 In Progress
- 🔴 Critical — do next
- 🟡 High impact
- 🟢 Nice to have
- ⏭️ Skipped (with reason)

---

## Phase 1 — Play Store Readiness ✅ Mostly Done

| # | Feature | Status | Notes |
|---|---|---|---|
| 1 | Onboarding Screen | ✅ Done | 3 slides, progress bar, rich content per slide |
| 2 | In-App Review (Google Play) | ✅ Done | Pre-warm on 2nd, show on 3rd conversion |
| 3 | Error Empty States | ✅ Done | Inline error card + library empty states (3 variants) |
| 4 | Share App / Invite Friends | ✅ Done | In SettingsBottomSheet, Intent.ACTION_SEND |
| 5 | Voice Speed Presets | ✅ Done | Study/Normal/Commute/Speed Run chips in AudioPlayerScreen |
| 6 | Recently Converted History | ⏭️ Skipped | Library already serves this purpose — re-converting wastes quota |
| 7 | Sleep Timer in Widget | 🟡 Pending | Add sleep timer control to VoxLibroWidget |
| 8 | Audio Trim / Clip | 🟢 Pending | Complex — cut beginning/end of saved audio |
| 9 | Notification Channels | 🔴 Next | Required for Android 8+ best practices, Play Store expects this |
| 10 | App Shortcuts (long-press icon) | 🟢 Pending | "Convert Text", "Open Library", "Last Played" |
| 11 | Dynamic Colors (Android 12+) | 🟡 Pending | Material You, one-line change in theme |
| 12 | Backup & Restore | 🟢 Pending | Google Drive backup of audio files |
| 13 | Listen Later Queue | 🟢 Pending | Save texts to convert later |

---

## Phase 2 — Growth & Retention 🔴 Next Priority

### 2.1 Notification Channels (🔴 Do This First)
**Why:** Android 8+ requires notification channels. Play Store reviewers check. Without this, all notifications go to a single unnamed channel — bad UX, bad review scores.
**Channels needed:**
- `playback` — audio playback controls (ongoing, low importance)
- `conversion` — conversion complete notification (default importance)
- `updates` — app update available (low importance)

**Files to create/modify:**
- New: `util/NotificationChannelManager.kt`
- Modify: `MainActivity.kt` — call `createChannels()` in `onCreate`
- Modify: Any service that shows notifications

---

### 2.2 Dynamic Colors — Material You (🟡)
**Why:** Android 12+ users expect apps to adapt to wallpaper colors. One setting change.
**Implementation:**
```kotlin
// In VoxLibroTheme.kt
val dynamicColor = Build.VERSION.SDK_INT >= Build.VERSION_CODES.S
val colorScheme = when {
    dynamicColor && darkTheme -> dynamicDarkColorScheme(context)
    dynamicColor && !darkTheme -> dynamicLightColorScheme(context)
    else -> /* existing dark scheme */
}
```
**Caveat:** VoxLibro uses hardcoded `Color(0xFF1DB954)` green throughout. Dynamic colors would override this. Decision needed: fully adopt Material You OR keep brand green. **Recommendation: keep brand green, but apply dynamic colors only to system components (status bar, navigation).**

---

### 2.3 App Shortcuts (🟢)
**Why:** Long-press on app icon → quick actions. Power users love it.
**Shortcuts to add:**
- "Convert Text" → opens MainScreen with keyboard open
- "Open Library" → opens SavedFilesScreen directly
- "Last Played" → opens last file in AudioPlayerScreen

**Implementation:** `ShortcutManagerCompat` with `ShortcutInfoCompat`. Static shortcuts in `res/xml/shortcuts.xml`.

---

### 2.4 Sleep Timer in Widget (🟡)
**Why:** Users want sleep timer accessible from home screen without opening app.
**Implementation:** Add a sleep timer button to `VoxLibroWidget`. Requires `PendingIntent` → `BroadcastReceiver` → `AudioPlayerManager.setSleepTimer()`.
**Complexity:** Medium — widget communicates with service via broadcast.

---

## Phase 3 — Power Features 🟢

### 3.1 Audio Trim / Clip
**Why:** No other free TTS app has this. Unique differentiator.
**What:** Cut silence from beginning/end of converted audio.
**Implementation:** `MediaMuxer` + `MediaExtractor` for lossless trim. Show waveform preview using `Visualizer` API.
**Complexity:** High — 2–3 days of work.

### 3.2 Listen Later Queue
**Why:** Users discover articles they want to hear later but can't convert immediately.
**What:** "Save for Later" button — stores text snippet in DataStore. Queue shows on MainScreen.
**Implementation:** New DataStore key `KEY_LISTEN_LATER_QUEUE` storing JSON list of `{title, text, addedAt}`.

### 3.3 Backup & Restore (Google Drive)
**Why:** Users fear losing their audio library. Backup = retention.
**What:** Upload/download VoxLibro audio files to Google Drive.
**Implementation:** Google Drive API + OAuth. Complex — requires Play Console setup.

---

## Phase 4 — Monetization (Future)

| Feature | Model | Notes |
|---|---|---|
| VoxLibro Pro | One-time purchase or subscription | Unlimited daily chars, priority API |
| Voice packs | IAP | Additional language voices |
| Cloud sync | Subscription | Cross-device library sync |

**Do NOT implement monetization until:** 1000+ daily active users, strong retention (D7 > 30%), positive Play Store reviews.

---

## Immediate Next Actions (for next session)

1. **Notification Channels** — `NotificationChannelManager.kt` + wire into `MainActivity`
2. **Dynamic Colors** — update `VoxLibroTheme.kt` (keep brand green, dynamic system only)
3. **App Shortcuts** — `res/xml/shortcuts.xml` + `ShortcutManagerCompat` in `MainActivity`
4. **Widget sleep timer** — `VoxLibroWidget.kt` + new `BroadcastReceiver`
5. **Play Store listing** — screenshots, description, keywords (separate task)

---

## Performance Targets

| Metric | Current | Target |
|---|---|---|
| Cold start | ~1.5s | < 1s |
| Conversion (short text, warm API) | 3–8s | < 5s |
| Conversion (short text, cold API) | 16–90s | 16s (ping already helps) |
| Voice preview tap → audio | ~0s (pre-generated) | ✅ Already achieved |
| Library open | < 200ms | ✅ Already achieved |

---

## Play Store Requirements Checklist

- [ ] Notification channels (Android 8+)
- [ ] Privacy policy URL
- [ ] App icon (all densities)
- [x] Onboarding for new users
- [x] In-app review integration
- [ ] Screenshots (phone + tablet)
- [ ] Feature graphic (1024×500)
- [ ] Short description (80 chars)
- [ ] Full description (4000 chars)
- [ ] Content rating questionnaire
- [ ] Data safety form
- [ ] Target audience declaration
