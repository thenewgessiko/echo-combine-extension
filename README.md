# 🎵 Echo Combine Extension ☆

[![Stable release][release-badge]][stable-download]
[![MAIN build][main-badge]][main-runs]
[![DEV snapshot][dev-badge]][dev-runs]
[![Android][android-badge]][echo]
[![JDK 21][jdk-badge]](#build-from-source)

**One metadata provider. Multiple audio providers.**

Combine Extension is a music extension for [Echo][echo] that separates metadata and library operations from audio playback. For example, you can use **Spotify** for metadata, **Deezer** for primary audio, and **YouTube Music** and **SoundCloud** as backups.

**The metadata provider is not used as an audio provider.** Audio availability, subscriptions, authentication, and regional restrictions still depend on the selected providers.

[Download stable][stable-download] · [DEV builds][dev-runs] · [Configuration](#configuration) · [Build from source](#build-from-source) · [Report an issue][issues]

## Features

- Configurable metadata extension for browsing, search, albums, artists, playlists, and supported library actions.
- Preferred audio provider and up to nine alternative audio providers.
- **First Available Mode**: search in priority order and stop at the first playable match.
- **Multi-provider mode**: search configured providers concurrently and collect matching streams.
- **Universal audio discovery**: up to six search queries, available search tabs, and paginated results.
- **Album fallback**: inspect matching album candidates when direct track searches fail.
- Strict title, artist, and recording-version matching to reduce wrong-track playback.
- Stream labels with the provider name and **☆** separator when the original stream has a title.
- Optional **Diagnostic Mode (DEV)** for troubleshooting unsuccessful searches.

## How it works

```text
Echo → Combine → metadata provider (e.g. Spotify)
                 ├─ Search, home, library, albums, artists, playlists
                 └─ Track details and supported library actions

Playback → preferred audio provider (e.g. Deezer)
         → alternatives in priority order (e.g. YouTube Music, SoundCloud)
         → matched track → playable streams
```

Combine retains the metadata track's identity and replaces its streams with results from configured audio providers. Available library actions depend on the metadata extension's capabilities.

## Installation

1. Install [Echo][echo] and enable the music provider extensions you want to use.
2. Download the `.eapk` asset from the [latest stable release][stable-download].
3. Import the `.eapk` through Echo's extension manager.
4. Open Combine Extension settings and configure the metadata and audio providers.

**Extension ID:** `echo_combine`

> [!IMPORTANT]
> A GitHub Actions artifact is not a published GitHub Release. If you download a workflow artifact, extract its ZIP to obtain the `.eapk`. Echo's GitHub update check requires an actual published release with a valid tag.

### Release channels

| Channel | Branch | Workflow | Package distribution | Updates |
| --- | --- | --- | --- | --- |
| **Stable** | `main` | `MAIN build` | [GitHub Releases][stable-download] and Actions artifacts | GitHub Releases updater |
| **Development** | `dev` | `DEV snapshot` | [GitHub Actions artifacts][dev-runs] | Manual import |

Both channels use the same extension ID, so installing a DEV package replaces the stable package. DEV packages do not use the stable release update endpoint.

## Configuration

Open **Echo → Settings → Extensions → Combine Extension**.

| Setting | Description |
| --- | --- |
| **Metadata Extension** | Provider for browsing, track information, and supported library actions. |
| **Preferred Stream Extension** | Highest-priority audio provider; `None` disables the audio chain. |
| **2nd–10th Alternative Stream Extension** | Up to nine backup audio providers, shown progressively. |
| **First Available Mode** | Return streams from the first provider with a playable match. |
| **Diagnostic Mode (DEV)** | Include search details in an Echo error report when no audio is found. |

Example configuration: **Spotify** (metadata) → **Deezer** (preferred audio) → **YouTube Music** → **SoundCloud**.

The provider order is configurable. Duplicate selections are ignored, and the metadata provider is excluded from audio choices. An unset slot or `None` ends the priority chain.

### Playback modes

| Mode | Behavior |
| --- | --- |
| **First Available ON** | Search sequentially by priority; return the first provider's playable streams. |
| **First Available OFF** | Search configured providers concurrently; return matching streams ordered by provider priority and quality. |

**First Available means highest priority, not fastest network response.**

## Universal track discovery

Combine tries up to six distinct, nonempty searches based on available metadata:

1. Track title + artist
2. Track title
3. Album title + artist
4. Album title
5. Simplified track title
6. Artist name

For each query, it inspects the default and available non-sort search tabs, with up to three pages per tab. Direct track matches are preferred. If none yields playable audio, Combine can inspect up to eight discovered album candidates and search their track listings, when supported by the provider.

### Matching rules

Combine compares normalized complete titles and artist names. Matching ignores case, punctuation, accents, and supported featured-artist credits, while preserving important version labels such as **Acoustic**, **Live**, and **Remix**.

| Metadata request | Audio candidate | Result |
| --- | --- | --- |
| `Evil Deeds` — Eminem | `Evil` — Eminem | ❌ Reject: shortened title |
| `Desires (ft. Future)` — Drake | `Desires` — Drake | ✅ Accept: featured credit |
| `Song (Acoustic)` | `Song` | ❌ Reject: version differs |
| `Song - Live` | `Song (Live)` | ✅ Accept: punctuation differs, version matches |
| `Song` — Artist A | `Song` — Artist B | ❌ Reject: artist differs |

Matching is based on provider metadata and cannot guarantee the recording behind a stream is correctly labeled.

## Playback fallback and limitations

If an audio provider fails during `loadStreamableMedia`, Combine can try later configured providers for matching playable streams. Search and track-loading failures are also handled without blocking other configured providers.

**Fallback can handle:** provider-side stream-resolution exceptions, including 403/429 errors raised during resolution.

**Fallback cannot handle:** HTTP errors raised later by Echo's player, expired cached URLs, queue prefetch, unsupported provider features, or incorrect source metadata.

> [!NOTE]
> The extension does not receive callbacks for HTTP failures occurring after Echo's player has received a media URL.

## Stream identification ☆

Stream labels include the provider name. When the original stream has a title, Combine adds the **☆** separator:

```text
Deezer ☆ FLAC
YouTube Music ☆ Audio
SoundCloud ☆ Audio
```

These are illustrative labels; actual stream titles depend on the provider. Original stream data is preserved for resolution.

## Diagnostic Mode (DEV)

Enable **Diagnostic Mode (DEV)** in Combine settings to record selected providers, search queries, tabs, pages, matching candidates, album attempts, and stream results. When no playable streams are found, these details can appear in Echo's error report.

Disable the mode during normal use. Remove credentials, account identifiers, cookies, and URL tokens before sharing logs.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| No playable audio | Confirm that a separate, enabled audio provider is configured and can play the track directly. |
| Wrong recording | Check the full title, artist, and version labels. |
| Deezer search misses a track | Enable Diagnostic Mode and inspect search and album discovery attempts. |
| Slow skipping | Try First Available Mode; network latency still applies. |
| Playback fails with 403/429 | Determine whether resolution failed inside the provider or the player failed afterward. |
| `GitHub release tag missing` | Confirm a stable release has been published on GitHub with a valid tag. |
| DEV build does not update | Download and import a newer DEV artifact manually. |

Report issues at [GitHub Issues][issues], including Echo and Combine versions, configured providers and order, exact track details, reproduction steps, and relevant redacted errors.

## Build from source

### Requirements

- JDK **21**
- Android SDK Platform **36** and Build Tools **35.0.0**
- Node.js for automation checks
- Git and network access for Gradle dependencies

```bash
git clone https://github.com/thenewgessiko/echo-combine-extension.git
cd echo-combine-extension
git switch main
```

Use `git switch dev` to work on the development branch. Keep the Git history because version codes derive from commit counts.

### Run checks

```bash
bash .github/scripts/test-release-metadata.sh
node .github/scripts/test-autoassign.mjs
bash ./gradlew :ext:check :ext:shadowJar --no-daemon
```

### Build a local debug package

```bash
bash ./gradlew :app:assembleDebug --no-daemon
mkdir -p dist
cp app/build/outputs/apk/debug/app-debug.apk dist/Combine-Local.eapk
```

Local debug builds use Android's debug signing configuration. Signing keys can differ between environments, affecting update compatibility.

## GitHub Actions

| Workflow | Trigger | Purpose |
| --- | --- | --- |
| [`build.yml`](.github/workflows/build.yml) — **MAIN build** | Push to `main`, manual run | Verify and build the stable `.eapk` artifact. |
| [`dev-build.yml`](.github/workflows/dev-build.yml) — **DEV snapshot** | Push to `dev`, manual run | Run tests and build a DEV `.eapk` artifact. |
| [`test.yml`](.github/workflows/test.yml) — **tests** | Manual run | Additional checks and extension JAR build. |
| [`autoassign.yml`](.github/workflows/autoassign.yml) | New/reopened issues and pull requests | Assign unassigned items according to repository configuration. |

**Release publication is manual:** download the stable workflow artifact, extract its `.eapk`, then create a GitHub Release (for example `v1.0.0`) targeting `main` and attach the `.eapk` asset. The workflow does not automatically publish GitHub Releases.

## Credits

Based on the original [Echo Combine Extension by dutixlf](https://github.com/dutixlf/echo-combine-extension).

Fork maintained by **[TheNewGessiko](https://github.com/thenewgessiko)**.

**Made with ❤️ by TheNewGessiko**

[echo]: https://github.com/brahmkshatriya/echo
[issues]: https://github.com/thenewgessiko/echo-combine-extension/issues
[stable-download]: https://github.com/thenewgessiko/echo-combine-extension/releases/latest
[main-runs]: https://github.com/thenewgessiko/echo-combine-extension/actions/workflows/build.yml?query=branch%3Amain
[dev-runs]: https://github.com/thenewgessiko/echo-combine-extension/actions/workflows/dev-build.yml?query=branch%3Adev
[release-badge]: https://img.shields.io/github/v/release/thenewgessiko/echo-combine-extension?label=stable&color=2ea44f
[main-badge]: https://img.shields.io/github/actions/workflow/status/thenewgessiko/echo-combine-extension/build.yml?branch=main&label=MAIN%20build
[dev-badge]: https://img.shields.io/github/actions/workflow/status/thenewgessiko/echo-combine-extension/dev-build.yml?branch=dev&label=DEV%20snapshot
[android-badge]: https://img.shields.io/badge/platform-Android-3ddc84?logo=android&logoColor=white
[jdk-badge]: https://img.shields.io/badge/build-JDK%2021-007396
