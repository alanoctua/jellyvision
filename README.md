# JellyVision for Roku

JellyVision turns a Jellyfin library into deterministic, always-on television
channels. It is based on the official Jellyfin Roku client, retaining its server
setup, login, saved sessions, device profiles, direct-play/transcoding support,
subtitles, and mature video player.

## Current milestone

- Existing Jellyfin server connection, password login, Quick Connect, and saved
  sessions from the upstream Roku client.
- A watch-first home screen: JellyVision first finds one random live-action comedy
  series and starts a temporary Comedy Preview channel. The programme panel and
  guide remain available while the complete scan continues in the background.
- JellyVision reads every page of movies, episodes, and parent-series metadata,
  then builds all 34 permanent channels. Large libraries take longer, but are no
  longer cut off at an alphabetical item limit. Preview playback never relaxes
  or changes the permanent channels' matching rules.
- Random TV, Movie Night, and Random Anime are complete overview channels.
  Outside those three intentional overviews, every movie or series is assigned
  to at most one specialist in each relevant lane to prevent excessive
  repetition. Anime has separate genre and era lanes, so one series may appear
  in Random Anime, one genre channel, and one decade channel.
- Live-action television, live-action films, non-anime animation, anime series,
  and anime films use separate pools. JellyVision never fills an empty channel with
  unrelated content.
- Deterministic schedules anchored to a fixed UTC epoch and tune-in playback at
  the exact elapsed offset of the programme currently on air.
- Remote channel control from the guide, channel-up/down keys where available,
  and best-effort numeric-key handling for remotes that expose number buttons to
  SceneGraph applications.
- `*` toggles between the JellyVision guide layout and full-screen video.
- The in-player guide uses a two-hour timeline with 15-minute markers. Programme
  cards are positioned and sized from their real scheduled start and end times.
- Ghost Mode defaults to enabled for JellyVision playback. It suppresses Jellyfin
  start, progress, and stop activity reports so channel surfing does not alter
  Recently Played, resume positions, watched status, or Next Up. Ordinary
  Jellyfin playback remains unchanged.

## Architecture

```text
ChannelEngine (Task)
  quick strict comedy preview -> paginated full-library scan
  -> classify once -> build all 34 permanent channels
            |
            v
ScheduleEngine (pure functions)
  stable item ordering + epoch schedule + current playback offset
            |
            v
ChannelGuide / CableFinPlayerOverlay / CableFinChannelRow
  startup state + watch-first programme panel + scrolling guide window
            |
            v
ChannelPlayer
  live schedule window -> official QueueManager -> official VideoPlayerView
```

The boundaries preserve a clean path for editable custom channels, channel
logos, richer overlays, surfing animations, favourites, and persisted schedule
snapshots. The current schedule stays stable while a channel's matching item IDs
and durations remain unchanged.

## Build

Requirements are the same as the upstream app: Node.js, npm, and a Roku in
developer mode.

```sh
npm ci
npm run build
```

The compiled sideload tree is written to `build/staging`; a packaged zip is
written to `out`. The upstream VS Code launch configuration can also deploy to a
Roku after its IP address and developer password are configured.

The original JellyVision artwork is preserved in `artwork`. Roku-ready splash
screens and home-screen posters are generated in `images` at the dimensions
referenced by the manifest.

## First run and controls

1. Enter the Jellyfin server URL, including `http://` or `https://` when needed.
2. Sign in with a Jellyfin user or Quick Connect.
3. JellyVision makes a small comedy-series request first. If a live-action match is
   available, the temporary channel `000 Comedy Preview` starts at the correct
   pseudo-live offset.
4. The complete Jellyfin scan continues while the preview plays. Its current
   phase, item count, and progress bar appear above the guide at the bottom.
5. When all 34 permanent channels are ready, they replace the preview in the
   guide. The preview keeps playing until you choose a permanent channel, so a
   programme is never interrupted merely because loading finished.
6. Press up/down to browse the visible guide and OK to tune. Use a supported
   number pad to enter a three-digit channel number such as `401`.
7. Press `*` for full-screen video; press Back to restore the guide, then Back
   again to leave playback.
8. On the main JellyVision guide, press `*` to open the side menu and switch Ghost
   Mode on or off. The same option is available under Settings / JellyVision.

## Current limits

- Channel rules are built in and strictly combine Jellyfin media type, genres,
  tags, year, and exclusions. Series genres and tags are inherited by episodes
  before classification. JellyVision never substitutes unrelated library content
  when a rule has no matches.
- The startup preview is also strict: it requires a comedy or sitcom series and
  excludes anime and animation. If no suitable series is found quickly, the app
  shows the normal loading screen rather than playing unrelated content.
- All available movies, episodes, and television series are retrieved in
  500-item pages. Channel schedules are no longer capped at 240 items.
- Numeric buttons are not part of Roku's guaranteed SceneGraph key set, so the
  app supports common digit key names but availability depends on the remote and
  Roku firmware.
- The playback queue is prepared 12 programmes ahead. A later milestone can
  extend it indefinitely and resynchronise after long pauses.
- Ghost Mode cannot hide authentication, library, playback-information,
  transcoding, or media-stream requests because those are required to play the
  file. It suppresses only the activity reports that update Jellyfin user
  history.
- Adding or removing matching media changes deterministic ordering. Persisted
  schedule snapshots remain a future feature.

## Channel lineup and matching rules

The matching pass separates content by Jellyfin item type and genre or tag
values. `Anime` is treated as its own required classification; JellyVision also
recognises a Jellyfin library or media-path folder containing the word `Anime`,
as well as tags such as `Anime Series`. Ordinary animation is excluded from
live-action channels.
Within each content family, competing genre channels are exclusive with one
another. Anime genre and decade channels use separate lanes so both kinds remain
populated. The three overview channels intentionally include their whole content
family.

| Channel | Name | Selection rule |
| --- | --- | --- |
| 101 | Random TV | Every live-action episode; excludes anime and animation |
| 102 | 90s Sitcoms | Comedy or sitcom episodes, 1988–1999; excludes anime and animation |
| 103 | Movie Night | Every live-action movie; excludes anime and animation |
| 104 | Drama TV | Drama episodes; excludes anime and animation |
| 105 | Kids & Family TV | Family, children, or kids episodes; excludes anime and animation |
| 106 | Animated TV | Animation episodes; excludes anime |
| 107 | Documentary TV | Documentary episodes; excludes anime and animation |
| 108 | Comedy TV | Comedy or sitcom episodes not assigned to 102; excludes anime and animation |
| 109 | Crime & Mystery TV | Crime or mystery episodes; excludes anime and animation |
| 110 | Action & Adventure TV | Action or adventure episodes; excludes anime and animation |
| 111 | Sci-Fi & Fantasy TV | Science fiction, sci-fi, sci fi, or fantasy episodes; excludes anime and animation |
| 112 | Horror & Thriller TV | Horror or thriller episodes; excludes anime and animation |
| 201 | Kids & Family Movies | Family, children, or kids movies; excludes anime and animation |
| 202 | Animated Movies | Animation movies; excludes anime |
| 203 | Documentary & History | Documentary, history, or biography movies; excludes anime and animation |
| 204 | Horror Movies | Horror movies; excludes anime and animation |
| 205 | Sci-Fi & Fantasy Movies | Science fiction, sci-fi, sci fi, or fantasy movies; excludes anime and animation |
| 206 | Crime, Mystery & Thriller | Crime, mystery, or thriller movies; excludes anime and animation |
| 207 | Action & Adventure Movies | Action or adventure movies; excludes anime and animation |
| 208 | Comedy Movies | Comedy movies; excludes anime and animation |
| 209 | Drama & Romance Movies | Drama or romance movies; excludes anime and animation |
| 210 | War & Westerns | War or western movies; excludes anime and animation |
| 400 | Random Anime | Every anime episode |
| 401 | Comedy Anime | Anime episodes tagged/genre comedy or sitcom |
| 402 | Action Anime | Anime episodes tagged/genre action or adventure |
| 403 | Drama Anime | Anime episodes tagged/genre drama |
| 404 | Feel-Good & Family Anime | Anime episodes tagged/genre feel good, family, kids, slice of life, iyashikei, heartwarming, wholesome, romance, friendship, school, coming of age, or healing |
| 405 | Horror & Thriller Anime | Anime episodes tagged/genre horror or thriller |
| 406 | Classic Anime | Anime episodes released through 1989 |
| 407 | 90s Anime | Anime episodes released 1990–1999 |
| 408 | 2000s Anime | Anime episodes released 2000–2009 |
| 409 | 2010s Anime | Anime episodes released 2010–2019 |
| 410 | Modern Anime | Anime episodes released from 2020 onward |
| 420 | Anime Movies | Movies tagged/genre anime |

## Upstream and licence

This repository is a derivative of
[jellyfin/jellyfin-roku](https://github.com/jellyfin/jellyfin-roku), the official
Jellyfin Roku client. The upstream GPL-2.0 licence is retained in `LICENSE`.
