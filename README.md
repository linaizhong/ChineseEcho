# Chinese Echo (Android)

A language-practice app for Android. It started as a reader that turns a PDF into a
listening-and-speaking exercise, and it now also scans documents with the camera, runs a
fully offline **voice chat** with an on-device language model, and does **real-time
speech translation**.

Everything below works on the phone itself. The only network use is downloading models
(the language model for Voice Chat, ML Kit translation models) and the optional Ollama
meaning-check.

## Features at a glance

| Feature | What it does |
| --- | --- |
| **Article Reader** | Loads learning material (PDF), reads it aloud, drills single lines, checks your speech |
| **Quiz** | Flashcard quiz built from the loaded material, with audio prompts and hints |
| **Document Scan** | Camera/gallery photo → OCR → tap a line to hear it or translate it; scans can be saved |
| **Voice Chat** | Offline spoken conversation with an on-device model, with scenarios, levels and tone practice |
| **Real-time translation** | Speak, stop, the on-device model translates, the translation is spoken |
| **Vocabulary** | A word list you build up, with vocabulary tests |
| **My languages** | Your own language, the language you learn and your level, used across the app |

---

## Try the app: Google Play internal testing

Chinese Echo is currently available to volunteer testers through Google Play's internal
testing track. To become a tester:

1. **Send the email address of your Google Play account to the developer of this app at
   [linaizhong@gmail.com](mailto:linaizhong@gmail.com).** This is the Google account you
   use in the Play Store on your Android phone. The developer must add it to the tester
   list first; the link below will not work until that is done.
2. Once the developer confirms that you have been added, open the testing link on the
   phone, signed in to that same Google account:
   **<https://play.google.com/apps/internaltest/4700366973030082423>**
3. Accept the invitation to become a tester, then install the app from the Play Store
   page that opens.

Updates then arrive through the Play Store like any other app. Feedback on speech
recognition, text-to-speech, Voice Chat and real-time translation on your phone model is
especially welcome.

---

## My languages (first-run setup)

On first launch a **My languages** dialog asks:

- **I speak**: your own language.
- **I want to learn**: any of 16 languages except your own.
- **Level**: Beginner / Intermediate / Advanced (used by Voice Chat).

The choices are saved and can be changed later from the Settings button. They drive the
Built-in material list, the reading and scanning languages, pronunciation guides and
Voice Chat.

## Article Reader (main screen)

- **Load Learning Material** opens a picker with two tabs: **Built-in** (PDFs bundled in
  the app's assets) and **My phone** (any PDF on the device, via the Storage Access
  Framework, so no storage permission is needed). The last source used is remembered.
- The **Built-in** list is filtered by your languages. PDFs are matched by file name, e.g.
  `Chinese_Speaking_Practice_for_Beginners.pdf` (English translations) or
  `Chinese_Speaking_Practice_for_Russian_Advanced.pdf` (Russian translations). Names that
  do not start with a language word are always shown. See `MaterialCatalog.kt`.
- PDF text extraction uses [PdfBox-Android](https://github.com/TomRoush/PdfBox-Android)
  (PDFs with a real text layer; for scanned pages use Document Scan). It fixes a
  font-encoding quirk where some Chinese characters appear as Kangxi Radical codepoints.
- **Bilingual dialogue detection, line by line.** `A: 你好！(Nǐ hǎo!) — Hello!`
  (Chinese-target) and `A: Hello! (/həˈloʊ/) — 你好！` (English-target) are both
  recognised automatically, as are Russian and Vietnamese editions. Dialogue lines that a
  PDF wrapped across several physical lines are re-joined first.
- **Read Document** (Play / Pause / Stop) reads line by line with speed and pitch
  sliders. Android's `TextToSpeech` has no pause, so pause is simulated by chunking.
- **Per-line speaker button** reads one phrase. Russian and Vietnamese editions speak the
  Chinese phrase with a Chinese voice and the translation with a matching voice.
- **Read Dialogues** plays only the labelled A/B turns, with two different voices (or
  pitches as a fallback).
- A large **"now reading" popup** follows the current line, with pinyin or IPA underneath.

### Listen and feedback

Tap a line, then **REPEAT** to speak it back; **STOP** ends it.

- Listen is **push-to-talk**: nothing ends until you tap STOP, however long your pauses
  are. Everything you said is recognised together afterwards (`ListenSession.kt`).
- Matching is lenient (edit distance + word overlap, `TextComparison`), so small
  transcription differences still count.
- **Optional AI meaning-check**: a Settings toggle sends the target phrase and your
  transcript to a **user-hosted Ollama server** to judge whether they mean the same thing.
  Off by default; if the server cannot be reached it silently falls back to the local
  match. See `SemanticChecker.kt`.

### Pronunciation guides

- Pinyin / IPA already written in a line's own brackets is shown as is.
- **Pinyin is generated** for Chinese text that has none (`PinyinEngine`).
- **IPA is generated offline** for English text that has none, from the CMU dictionary
  (`assets/ipa/cmudict.txt`, licence in `assets/ipa/NOTICE.txt`; keep both together).
  It is shown only when English is not your own language.

## Quiz

Builds a shuffled flashcard quiz from the loaded material's dialogue lines. Each card
shows the translation; you say the answer (checked like Listen) or reveal it, then mark
yourself right or wrong. A running score and an end-of-quiz summary with reshuffle are
included.

- **Hear the prompt** speaks each card's prompt aloud; an **Auto** switch (remembered)
  does it for every card.
- **Hint** shows pinyin / IPA (IPA for English answers).

## Document Scan

Photograph any document, menu or sign (system camera app or gallery; no CAMERA
permission is needed) and read it line by line.

- On-device OCR with ML Kit for Chinese, Latin, Japanese, Korean and Devanagari
  (Hindi); the reader for each line is chosen from your languages, and language
  identification tells English from Spanish, French and so on.
- **Tap a line** to hear it in its own language, with pinyin / IPA underneath.
- **Translate on tap** with offline ML Kit Translate (each language model is about
  30 MB, downloaded on first use). Only the tapped line is translated by default.
- **Read all** reads every line and each line's translation after it.
- **Save** a scan (name, lines, translations and a thumbnail), browse **Saved scans**, and
  **Open in reader** to continue in the main screen.
- On phones without Google Play services the app says so, because ML Kit OCR and
  translation need it.

## Voice Chat

An offline spoken conversation partner. You speak, an on-device language model writes
the reply, and the reply is read aloud sentence by sentence while it is still being
written. The chat is in the language you are learning, at your level.

- **On-device model** (Google LiteRT-LM, `OnDeviceLlm`). Models are downloaded from
  inside the screen (currently Qwen2.5 1.5B and Qwen3 0.6B; see `ModelCatalog`), with
  progress, a Wi-Fi-only option, delete, and a model manager.
- **A turn** goes Talk → listen → Done speaking → think → speak. The microphone is only
  opened when the phone is idle, so it never hears itself.
- **Scenarios**: free conversation or role-plays (for example ordering in a restaurant)
  at three levels. A scenario card shows the goal and useful phrases; in a role-play the
  model speaks first.
- **Say it**: repeat a phrase from the scenario card and get a match score.
- **Practice options**: *listening mode* (replies stay hidden until tapped), *show
  reading* (pinyin / IPA under each message) and *hands-free* (listens by itself after
  each reply).
- **Chat history**: conversations are saved and can be reopened.
- **Vocabulary from chat**: select a word in a reply, add its meaning and save it
  (Chinese and English).
- **Tone practice** for Mandarin learners (`ToneDrillActivity`).
- **Teacher assignments**: a link such as `chineseecho://chat?scenario=restaurant&level=beginner`
  (copy, share or QR code) opens Voice Chat on that scenario after the learner confirms.
- **Crash protection**: if the model crashed the app while loading, the next start
  retries in safe mode and then stops instead of looping; there is a low-RAM check, an
  optional GPU "faster mode" that falls back to the CPU, and a diagnostics report (no
  conversation text) that can be copied or shared.

## Real-time translation

Open it from the **translate icon** in the toolbar of the main screen.

1. Pick **I speak** and **Translate to** (16 languages; the swap button exchanges them).
2. Tap **Start speaking** and talk.
3. Tap **Stop speaking**. The on-device model (the one downloaded in Voice Chat)
   translates what you said.
4. The translation is read aloud in the target language, sentence by sentence while it is
   being written.

Details:

- Each utterance is translated on its own: the model's conversation is reset before each
  one, and the instructions ask for the translation only.
- **Stop** cancels a round; **Play again** repeats the last translation.
- The conversation (what you said and its translation) stays on screen; the trash icon
  clears the screen only.
- **History**: every finished translation is saved automatically (languages, time, what
  was said, the translation; latest 200, in `live_translate_history.json` in the app's
  private storage). The clock icon opens the list with **Play**, **Copy**, **Delete**,
  **Clear all** and **Export**.
- **Export** saves the whole history to a file you choose (Storage Access Framework, so no
  permission is needed): a readable **.txt**, or a **.csv** (time, from, to, source,
  translation) that opens in Excel or Google Sheets, oldest first, UTF-8.
- If the phone has no voice for the target language, the translation is shown but not
  spoken, and a message says so.
- It needs a downloaded model; without one the screen offers a button to open Voice Chat.
  Quality depends on the model: the smaller model is weak for some language pairs.
- The language pair is remembered. Speech recognition uses the "I speak" language and
  prefers the phone's offline pack when it has one.

## Vocabulary

A word list (term, meaning, language, and a sentence clue) filled from Voice Chat or by
hand, with vocabulary tests (`VocabularyActivity`, `VocabularyTestActivity`).

---

## Opening the project

1. Use **Android Studio** (Hedgehog / 2023.1.1 or newer): *File ▸ Open…* and choose the
   project folder.
2. If the Gradle wrapper jar is missing, Android Studio offers to regenerate it; accept.
3. Let Gradle sync. Dependencies include PdfBox-Android, ML Kit text recognition
   (unbundled, from Google Play services), ML Kit language identification, ML Kit
   Translate, Google LiteRT-LM and ZXing (QR codes).
4. Run on a device or emulator with **API 24+** (Android 7.0+). Target/compile SDK is 36.
5. In `app/build.gradle`, `litertlm-android` is set to `latest.release`. After a
   successful sync, replace it with the version Gradle resolved so builds stay
   reproducible.

## Permissions and network

- **`RECORD_AUDIO`**: speech recognition (Listen, quiz, Voice Chat, translation).
- **`INTERNET`**: downloading the Voice Chat language model, downloading ML Kit
  translation models, and the optional Ollama meaning-check (to a server URL the user
  types in). Nothing else is sent, and conversations never leave the phone.
- **`android:usesCleartextTraffic="true"`** lets the optional meaning-check reach a local
  Ollama server over plain `http://`. If other network features are added, scope this
  with a Network Security Config.
- **No camera or storage permission**: photos use the system camera app through a
  `FileProvider`, and files are picked through the Storage Access Framework.
- The manifest declares package visibility for text-to-speech engines, speech
  recognisers and Google Play services, which phones with non-Google speech services
  (many models sold in mainland China) need.

## Requirements and gotchas

- **Text-to-speech**: at least one TTS engine and the voice data for the languages used.
  If a voice is missing, Android prompts to download it, or install it under
  *Settings ▸ Accessibility ▸ Text-to-speech output*.
- **Speech recognition**: needs `SpeechRecognizer.isRecognitionAvailable()` to be true.
  The app asks for the phone's offline language pack first and falls back to the normal
  recogniser when none is installed.
- **On-device model**: needs enough free storage for the download (a few hundred MB to
  about 1.6 GB) and enough RAM; the app warns on low-RAM phones. A native crash while
  loading is tracked so the app does not repeat it; open Voice Chat to recover.
- **Only one screen holds the model at a time.** The translation screen and Voice Chat
  each load it themselves and release it when closed.
- **Document Scan and translation** need Google Play services.
- **Scanned PDFs** have no text layer; use Document Scan instead of Load Learning
  Material.
- **Ollama meaning-check**: needs a reachable server with the model pulled. On the
  emulator, `http://10.0.2.2:<port>` reaches the host; on a device use the host's LAN IP.
  If unreachable it falls back silently by design.

## Releasing to Google Play

- Raise `versionCode` in `app/build.gradle` for every upload.
- The release build uses R8 (`minifyEnabled true`), which packs the mapping file into the
  bundle, and `ndk { debugSymbolLevel 'SYMBOL_TABLE' }`, which packs native debug
  symbols and clears Play Console's "no debug symbols" warning.

## Possible extensions

- Unit tests for `ContentLineParser` (many edge cases have come up by hand).
- Rejoining hyphenated PDF word-wraps (`exam-\nple`).
- Word-level highlighting during single-line playback.
- Offline speech recognition (for example Vosk) for phones without Google speech services.
- Hands-free mode for real-time translation (listen again after each translation).
- Share the translation history directly from the app (export currently saves to a file).
