
# 🎵 Echo Combine Extension

[![Latest Release][release-badge]][stable-download]
[![MAIN Build][main-build-badge]][main-runs]
[![DEV Snapshot][dev-build-badge]][dev-runs]
[![Android][android-badge]][echo]
[![JDK 21][jdk-badge]][build-from-source]

**One metadata provider. Multiple audio sources. One seamless music experience.**

**Combine Extension** is a music extension for [Echo][echo] that separates music metadata from audio playback.

Choose your preferred provider for browsing, searching, albums, playlists, and library management, while using different providers to find and play audio.

For example:

- **Spotify** → Music metadata
- **Deezer** → Primary audio source
- **YouTube Music** → First backup source
- **SoundCloud** → Second backup source

> [!NOTE]
> Provider priority is fully configurable. The example above represents a recommended configuration, not a mandatory setup.

[**Download Latest Release**][stable-download] · [**Installation**](#-installation) · [**Configuration**](#-configuration) · [**Build from Source**](#-build-from-source) · [**Report an Issue**][issues]

---

## ✨ Features

- **Independent metadata provider** for music browsing and library operations.
- **Multiple audio providers** with configurable priority.
- **Universal audio search** using up to six search strategies.
- **Album-based discovery** when direct track matching fails.
- **Automatic provider fallback** for supported resolution failures.
- **First Available Mode** for priority-based playback selection.
- **Multi-Provider Mode** for collecting streams from several providers.
- **Strict track matching** to reduce incorrect audio results.
- **Audio source identification** using the `☆` separator.
- **Diagnostic Mode (DEV)** for investigating unsuccessful searches.
- **GitHub Actions builds** for stable and development versions.

---

## 🧩 How It Works

```text
                       Echo
                         |
                  Combine Extension
                         |
             +-----------+-----------+
             |                       |
      Metadata Provider        Audio Providers
             |                       |
           Spotify                   |
             |                +------+------+
             |                |      |      |
       Search / Library     Deezer  YTM  SoundCloud
       Albums / Artists       |      |      |
       Playlists / Tracks     +------+------+
             |                       |
             +-----------+-----------+
                         |
                  Track Matching
                         |
                 Available Streams
                         |
                      Playback
```

### Provider Responsibilities

| Provider Role | Responsibility |
|---|---|
| **Metadata Provider** | Search, home feed, albums, artists, playlists, track information, and supported library operations. |
| **Preferred Audio Provider** | Highest-priority source for finding playable audio. |
| **Alternative Audio Providers** | Additional sources used for stream selection and fallback. |

Combine preserves the metadata track's identity and replaces its playback streams with streams discovered through the configured audio providers.

**The metadata provider is excluded from the audio provider chain.**

> [!IMPORTANT]
> Combine cannot provide library actions or playback capabilities that the underlying extensions do not support.

---

## 📥 Installation

### Requirements

- [Echo Music Player][echo]
- At least one compatible metadata provider
- At least one separate audio provider
- An Android device supported by Echo

### Installation Steps

1. Install **Echo**.
2. Install and enable your preferred music provider extensions.
3. Open the [latest Combine Extension release][stable-download].
4. Download the `.eapk` file from **Assets**.
5. Open Echo's extension manager.
6. Import the downloaded `.eapk` package.
7. Open **Combine Extension** settings.
8. Configure your metadata and audio providers.

**Extension ID:** `echo_combine`

> [!NOTE]
> GitHub Actions artifacts are distributed as ZIP archives. Extract the `.eapk` file before importing it into Echo.

### Updates

Stable versions use the official GitHub Releases endpoint for update checks.

**Official releases:**

https://github.com/thenewgessiko/echo-combine-extension/releases

> [!IMPORTANT]
> GitHub Actions artifacts are not GitHub Releases. Automatic update checks require a published release with a valid tag.

---

## ⚙️ Configuration

Open **Echo → Settings → Extensions → Combine Extension**.

| Setting | Description |
|---|---|
| **Metadata Extension** | Selects the provider used for music metadata and library operations. |
| **Preferred Stream Extension** | Selects the highest-priority audio provider. |
| **2nd Alternative Stream Extension** | First backup audio provider. |
| **3rd Alternative Stream Extension** | Second backup audio provider. |
| **4th Alternative Stream Extension** | Third backup audio provider. |
| **5th Alternative Stream Extension** | Fourth backup audio provider. |
| **First Available Mode** | Returns streams from the first provider with a playable match. |
| **Diagnostic Mode (DEV)** | Enables additional diagnostic information when searches fail. |

Alternative provider settings appear progressively as preceding providers are configured.

### Recommended Configuration

| Setting | Recommended Value |
|---|---|
| Metadata Extension | Spotify |
| Preferred Stream Extension | Deezer |
| 2nd Alternative | YouTube Music |
| 3rd Alternative | SoundCloud |
| First Available Mode | Enabled |
| Diagnostic Mode (DEV) | Disabled during normal use |

This configuration prioritizes Deezer while retaining alternative audio sources.

### Provider Priority

```text
1. Deezer
   |
   | No playable match
   v
2. YouTube Music
   |
   | No playable match
   v
3. SoundCloud
   |
   | No playable match
   v
No audio source found
```

Provider order follows the extension settings.

Duplicate selections are ignored, and unavailable providers do not prevent later configured backups from being considered.

---

## 🎧 Playback Modes

Combine supports two audio discovery modes.

### First Available Mode — ON

Audio providers are searched sequentially according to their configured priority.

```text
Deezer
  |
  +-- Match found --> Return Deezer streams
  |
  +-- No match --> YouTube Music
                     |
                     +-- Match found --> Return YTM streams
                     |
                     +-- No match --> SoundCloud
```

**Advantages:**

- Respects provider priority.
- Avoids unnecessary searches after a successful match.
- Can reduce search overhead when the preferred provider succeeds.

### First Available Mode — OFF

Configured providers are searched concurrently.

```text
       Track Request
             |
      +------+------+
      |      |      |
    Deezer  YTM  SoundCloud
      |      |      |
      +------+------+
             |
       Matched Streams
             |
      Provider Priority
             |
        Echo Playback
```

**Advantages:**

- Collects available streams from multiple providers.
- Provides additional source choices.
- Preserves the configured provider order.

> [!NOTE]
> First Available Mode prioritizes the configured provider order, not whichever provider responds fastest.

---

## 🔎 Universal Audio Search

Combine uses a multi-strategy search system to locate matching audio tracks.

### Search Strategies

Depending on available metadata, Combine attempts:

1. **Track Title + Artist**
2. **Track Title**
3. **Album Title + Artist**
4. **Album Title**
5. **Simplified Track Title**
6. **Artist Name**

Empty and duplicate queries are automatically excluded.

### Search Pagination

Combine examines available default and non-sort search tabs.

For each search strategy, it can inspect up to **three pages per examined tab**, depending on the provider's pagination support.

This improves discovery of tracks that are not present in the first search results.

### Album-Based Discovery

When direct track searches fail, Combine can attempt album-based resolution.

```text
Search Track
     |
     v
Direct Track Matching
     |
     +-- Match found --> Load Audio
     |
     +-- No match
           |
           v
     Album Candidates
           |
           v
     Open Album
           |
           v
     Search Album Tracks
           |
           +-- Match found --> Load Audio
           |
           +-- No match --> Continue Search
```

Album-based discovery requires the audio provider to support the necessary album operations.

The extension limits the number of album candidates and examined pages to keep searches bounded.

---

## 🎯 Track Matching

Combine uses strict matching rules to reduce incorrect playback results.

Track matching considers:

- Normalized track titles.
- Primary artist identity.
- Featured-artist credits.
- Recording version labels.
- Common audio filename extensions.

### Normalization

Matching ignores differences in:

- Letter case.
- Punctuation.
- Accents.
- Supported featured-artist formatting.

Version labels such as **Acoustic**, **Live**, and **Remix** remain relevant to matching.

### Examples

| Metadata Track | Audio Candidate | Result |
|---|---|---|
| `Evil Deeds` | `Evil` | ❌ Rejected |
| `Desires (ft. Future)` | `Desires` | ✅ Accepted with matching artist |
| `Song (Acoustic)` | `Song` | ❌ Rejected |
| `Song - Live` | `Song (Live)` | ✅ Accepted with matching artist |
| `Song` — Artist A | `Song` — Artist B | ❌ Rejected |

> [!WARNING]
> Strict matching helps prevent incorrect versions from being selected, but it cannot guarantee that a provider's metadata accurately describes the underlying recording.

---

## 🔄 Automatic Provider Fallback

Combine can retry audio resolution through later configured providers when supported stream resolution fails.

### Supported Cases

- Provider search failures.
- Missing matching tracks.
- Track-loading failures.
- Supported stream resolution exceptions.
- Provider-side HTTP 403/429 errors raised during stream resolution.

### Limitations

Fallback does **not** automatically recover from:

- HTTP failures raised later by Echo's player.
- Expired media URLs already returned to Echo.
- Cached queue entries.
- Unsupported provider functionality.
- Incorrect metadata that prevents a valid match.

> [!IMPORTANT]
> Fallback operates within the extension's available APIs. Errors that occur after Echo's player receives a media URL may require handling outside Combine.

---

## ⭐ Audio Source Identification

Combine adds the provider name to stream labels.

When an original stream title exists, the label uses the **☆** separator.

**Examples:**

```text
Deezer ☆ FLAC
YouTube Music ☆ Audio
SoundCloud ☆ Audio
```

The actual stream title depends on the audio provider.

Original stream data is preserved internally for playback resolution.

---

## 🐞 Diagnostic Mode (DEV)

Diagnostic Mode provides additional information about audio discovery.

When enabled, Combine records details such as:

- Selected audio providers.
- Search strategies.
- Search tabs.
- Pagination progress.
- Track candidates.
- Matching results.
- Album discovery attempts.
- Available stream results.

When no playable streams are found, diagnostic information can be included in an Echo error report.

### How to Enable

1. Open Combine Extension settings.
2. Enable **Diagnostic Mode (DEV)**.
3. Reproduce the unsuccessful track search.
4. Inspect the resulting error details.
5. Disable Diagnostic Mode when troubleshooting is complete.

> [!TIP]
> Include relevant diagnostic information when reporting search or playback issues. Remove personal information, tokens, and account credentials before sharing logs.

---

## 📦 Development Channels

The repository uses separate branches for stable development and experimental builds.

| Channel | Branch | Workflow | Distribution |
|---|---|---|---|
| **Stable** | `main` | MAIN build | GitHub Releases / Actions artifacts |
| **Development** | `dev` | DEV snapshot | GitHub Actions artifacts |
| **Nightly** | `nightly` | MAIN build | GitHub Actions artifacts |

### Stable

- Official version numbering, beginning with `1.0.0`.
- Intended for regular use.
- Published through GitHub Releases.
- Supports release-based update checks.

### Development

- Built automatically from the `dev` branch.
- Uses development version identifiers.
- Intended for testing and troubleshooting.
- Distributed as GitHub Actions artifacts.
- Does not use the stable release update endpoint.

### Nightly

- Uses commit-based nightly identifiers.
- Intended for experimental testing.
- Distributed as GitHub Actions artifacts.
- Automatic stable-release updates are disabled.

> [!NOTE]
> Development and nightly packages share the extension ID `echo_combine` with stable builds. They are not separate installations.

---

## 🛠️ Build from Source

### Requirements

- **JDK 21**
- Android SDK
- Android SDK Platform 36
- Android SDK Build Tools 35.0.0
- Git
- Gradle Wrapper (included)
- Node.js for repository automation checks

### Clone Repository

```bash
git clone https://github.com/thenewgessiko/echo-combine-extension.git
cd echo-combine-extension
```

### Select a Branch

**Stable:**

```bash
git switch main
```

**Development:**

```bash
git switch dev
```

### Run Tests

```bash
bash .github/scripts/test-release-metadata.sh
node .github/scripts/test-autoassign.mjs
bash ./gradlew :ext:check --no-daemon
```

### Build Extension

```bash
bash ./gradlew :app:assembleDebug --no-daemon
```

**Output:**

```text
app/build/outputs/apk/debug/app-debug.apk
```

### Package for Echo

```bash
mkdir -p dist
cp app/build/outputs/apk/debug/app-debug.apk dist/Combine-Local.eapk
```

The generated `.eapk` can be imported into Echo.

> [!WARNING]
> Local debug builds use the Android debug signing configuration. Package compatibility and update behavior can depend on the signing key used.

---

## ⚡ GitHub Actions

### MAIN build

**Workflow:** `.github/workflows/build.yml`

Runs on:

- Pushes to `main`.
- Pushes to `nightly`.
- Manual workflow dispatch.
- Scheduled nightly execution.

Performs:

1. Source checkout.
2. JDK 21 setup.
3. Release metadata verification.
4. Automation checks.
5. Extension tests.
6. Release metadata preparation.
7. Debug extension compilation.
8. `.eapk` artifact upload.

### DEV snapshot

**Workflow:** `.github/workflows/dev-build.yml`

Runs on:

- Pushes to `dev`.
- Manual workflow dispatch.

Performs:

1. Source checkout.
2. JDK 21 setup.
3. Extension tests.
4. Diagnostic report upload.
5. Development debug build.
6. `.eapk` artifact upload.

### Manual Tests

**Workflow:** `.github/workflows/test.yml`

Runs through manual workflow dispatch.

Includes release metadata checks, automation tests, extension checks, and extension JAR generation.

> [!IMPORTANT]
> The current build workflows upload artifacts. Publishing an official GitHub Release is a separate step.

---

## 🩺 Troubleshooting

| Problem | Recommended Action |
|---|---|
| **No playable audio** | Verify provider selection, availability, and track matching. |
| **Wrong track version** | Check the exact title, artist, and recording version. |
| **Deezer cannot find a track** | Enable Diagnostic Mode and inspect search and album discovery results. |
| **Provider unavailable** | Confirm that the provider extension is installed and enabled. |
| **Slow track loading** | Try First Available Mode to avoid unnecessary backup searches. |
| **Playback fails with 403/429** | Determine whether the error occurs during provider resolution or later in Echo's player. |
| **GitHub release tag missing** | Verify that a stable GitHub Release has been published with a valid tag. |
| **Development build does not update** | Download and import a newer development artifact manually. |

---

## 🐛 Bug Reports

Found an issue?

[**Open a GitHub Issue**][issues]

Please include:

- Echo version.
- Combine Extension version.
- Metadata provider.
- Audio providers and priority order.
- First Available Mode status.
- Diagnostic Mode status.
- Exact track title and artist.
- Steps to reproduce.
- Relevant error messages or stack traces.

**Never share authentication tokens, cookies, private account information, or authorization headers.**

---

## 📜 Credits

**Combine Extension** is based on the original project by [dutixlf](https://github.com/dutixlf/echo-combine-extension).

This fork is developed and maintained by **[TheNewGessiko](https://github.com/thenewgessiko)**.

Special thanks to the original project contributors and the Echo community.

---

## ❤️ Support & Development

For bug reports, suggestions, and improvements, visit the official repository:

**[TheNewGessiko / echo-combine-extension](https://github.com/thenewgessiko/echo-combine-extension)**

Stable releases are available through **GitHub Releases**.

Development snapshots are available through **GitHub Actions**.

**Made with ❤️ by TheNewGessiko**

---

<!-- Reference Links -->

[echo]: https://github.com/brahmkshatriya/echo
[issues]: https://github.com/thenewgessiko/echo-combine-extension/issues
[stable-download]: https://github.com/thenewgessiko/echo-combine-extension/releases/latest
[main-runs]: https://github.com/thenewgessiko/echo-combine-extension/actions/workflows/build.yml?query=branch%3Amain
[dev-runs]: https://github.com/thenewgessiko/echo-combine-extension/actions/workflows/dev-build.yml?query=branch%3Adev
[build-from-source]: #️-build-from-source

<!-- Badges -->

[release-badge]: https://img.shields.io/github/v/release/thenewgessiko/echo-combine-extension?label=stable&color=2ea44f
[main-build-badge]: https://img.shields.io/github/actions/workflow/status/thenewgessiko/echo-combine-extension/build.yml?branch=main&label=MAIN%20build
[dev-build-badge]: https://img.shields.io/github/actions/workflow/status/thenewgessiko/echo-combine-extension/dev-build.yml?branch=dev&label=DEV%20snapshot
[android-badge]: https://img.shields.io/badge/platform-Android-3ddc84?logo=android&logoColor=white
[jdk-badge]: https://img.shields.io/badge/build-JDK%2021-007396
