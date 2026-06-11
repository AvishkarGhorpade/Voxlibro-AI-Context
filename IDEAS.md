# VoxLibro — Ideas & Future Directions
> Unfiltered ideas. Not all will be built. Evaluate each for user value vs complexity.

---

## User-Requested Features

### From Friend Testing Session 1
- **Voice preview before selecting** ✅ Done — pre-generated samples in VoicePickerSheet
- **Test speed/pitch before converting** ✅ Done — preview sliders in VoicePickerSheet
- **Don't re-convert to change speed** → Solution: speed is adjustable in AudioPlayerScreen during playback. Pre-conversion preview now available.

---

## UI/UX Ideas

### Waveform Visualization
Show a real-time waveform animation during playback in AudioPlayerScreen. Replace or enhance the vinyl disc.
- **Implementation:** Android `Visualizer` API (requires `RECORD_AUDIO` permission — check if acceptable)
- **Alternative:** Fake animated waveform using `infiniteRepeatable` animations (no permission needed)
- **Verdict:** Fake waveform preferred — avoids permission friction

### Bookmarks in Audio
Let users drop a bookmark at any point in a long audio file. Resume from bookmark next time.
- **Implementation:** Store `{fileName, positionMs}` list in DataStore
- **UI:** Bookmark icon in AudioPlayerScreen, list in SavedFilesScreen

### Text Highlighting Sync
While audio plays, highlight the corresponding word in the original text.
- **Implementation:** Very complex — requires word-level timestamps from TTS API. Not supported by current Edge TTS API.
- **Future:** Worth exploring if API adds word-level timestamps

### Dark/Light Theme Toggle
Currently always dark. Add a theme toggle in Settings.
- **Implementation:** `VoxLibroTheme` already uses `MaterialTheme`. Add `isSystemInDarkTheme()` check.
- **Note:** All hardcoded colors need to be moved to theme tokens first.

### Playlist / Queue
Multiple audio files in a queue that plays sequentially.
- **Implementation:** `AudioPlayerManager` currently plays one file. Add a queue list + `onCompletion` → next file logic.

---

## Monetization Ideas

### VoxLibro Pro
**Price:** ₹99/month or ₹699/year or $1.99/$9.99
**Features:**
- Unlimited daily character quota (remove 20k/day limit)
- Priority API (separate faster Render instance)
- Background conversion (convert while app is in background)
- Cloud library sync

### Voice Packs (IAP)
Additional voices beyond the 12 included:
- Celebrity-style voices
- Regional Indian accents (Tamil, Telugu, Bengali, Gujarati)
- Kids voice (higher pitch, slower, friendly)

### B2B / API
Expose VoxLibro as an API for developers who want TTS in their apps.

---

## Growth Ideas

### Referral Program
"Invite a friend, both get 5,000 bonus characters"
- Track via unique referral codes stored in DataStore
- Apply bonus to `MAX_CHARS_PER_DAY` dynamically

### Social Sharing of Audio
"Share this audio clip" — share the actual MP3 via WhatsApp, Telegram etc.
- Already partially implemented (`ShareUtils.shareAudioFile`)
- Enhancement: add a short intro ("Shared from VoxLibro") to the beginning of shared audio

### Widgets for iOS-style Lock Screen
Android 12+ lock screen widgets showing "Now Playing" info.

### Siri Shortcuts / Google Assistant Integration
"Hey Google, convert my clipboard with VoxLibro"

---

## Technical Debt

### Move to ViewModel
The current single-Activity approach works but `MainContent()` is ~700 lines. Long term, extract:
- `ConversionViewModel` — handles TTS state, file naming, error
- `LibraryViewModel` — handles file list, sorting, search
- `PlayerViewModel` — wraps AudioPlayerManager state

### Move to Navigation Component
Currently screen switching is manual state (`var currentScreen by remember`). Should use `NavHost` for back stack, deep links, and screen transition animations.

### Dependency Injection (Hilt)
`UserPreferencesManager`, `TextToSpeechManager`, `CustomTtsService`, `AudioPlayerManager` are all manually instantiated. Hilt would make testing easier.

### Unit Tests
Zero test coverage currently. Priority areas:
- `CustomTtsService` — error code logic
- `UserPreferencesManager` — streak calculation
- `TextToSpeechManager` — online/offline routing

---

## Market Positioning Ideas

### "VoxLibro for Education"
Partner with schools and colleges in India. Students convert textbook chapters to audio.
- Bulk conversion feature
- Teacher dashboard
- Student accounts

### "VoxLibro for Accessibility"
Position as a tool for people with dyslexia, visual impairments, learning disabilities.
- Apply for Google.org grants
- Submit to accessibility app directories
- Partner with NGOs

### Regional Language Expansion
Currently: English (4 accents), Hindi, Marathi
Next wave: Tamil, Telugu, Bengali, Gujarati, Kannada, Malayalam
- Check Edge TTS API support for these languages
- May need different API or model

---

## App Store Optimization (ASO) Ideas

### Keywords to target
- text to speech android
- listen to articles
- PDF reader audio
- study audio converter
- Hindi text to speech
- Marathi voice reader
- audiobook converter

### Competitor analysis
- NaturalReader — paid, complex UI
- Speechify — expensive subscription
- Voice Aloud Reader — outdated UI
- **VoxLibro advantage:** Free tier, 12 neural voices, Hindi/Marathi, beautiful UI, offline support

---

## Session Notes

### Session 1 Observations
- User is a developer (Avishkar) building this as a real product
- Friend testing revealed the voice preview gap → led to pre-generated samples feature
- Render free tier is a real pain point — consider upgrading or self-hosting API
- App has strong foundation, needs notification channels before Play Store submission
- Design language is clean, consistent, market-ready after session 1 redesigns
