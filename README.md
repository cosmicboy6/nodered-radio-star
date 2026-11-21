# Node-RED – Radio-Star


A **self-contained morning radio announcement** flow for Node-RED: it gathers
weather + day length + news, turns them into speech, and plays them on a
network media player with a volume ramp.

It needs **no Home Assistant** — everything is fetched and generated inside
Node-RED. This repo holds the exported flow, its audio assets and the
documentation.

## What the flow does

| Trigger (cron) | Step |
| --- | --- |
| `:26` / `:56` | Scrape weather from `wien.gv.at` (200-char split at sentence boundary) → Google TTS, with `highbell24.mp3` as separator |
| `:25` / `:55` | ORF news (`rss.orf.at`) → TTS, one highbell per item |
| `23:54` | Day length for Vienna (`sunrise` node) → TTS |
| `00:59` / `12:59` | Sunrise / sunset TTS |
| `:59:40` | Combine everything into `final.mp3` |
| `RADIO STAR` (manual inject) | Play `final.mp3` on the media player |

Playback goes to the media player at **`192.168.0.25`** (its `httpapi.asp`
endpoint), which pulls the audio from the web server on **`192.168.0.50`**;
the volume ramps **60 → 75 → 85 → 100**.

The "Combining" node stitches `final.mp3` from these parts, in order:

```
notification.mp3
<hour>.mp3                              (0.mp3 … 23.mp3)
Die aktuellsten kurz Nachrichten.mp3
nachrichten.mp3
Wetter prognose für heute <weekday>.mp3
wetter.mp3
tageslaenge.mp3
Sonnenaufgang.mp3 / Sonnenuntergang.mp3 (time-dependent, optional)
spacer.mp3
```

> **Note:** the flow maps `Dienstag → Diensstag` on purpose, because the
> matching TTS file is named `Wetter prognose für heute Diensstag.mp3`.

## Requirements

- **Node-RED 5.0.9** (exported from this version)
- Palette module: **`node-red-node-suncalc`** (provides the `sunrise` node)
- A web server on `192.168.0.50` serving `/share/htdocs/audio/tts/…`
- A network media player at `192.168.0.25` with an `httpapi.asp` command
  interface
- Internet access to `wien.gv.at`, `rss.orf.at` and
  `translate.google.com/translate_tts`

> **TTS note:** the flow uses Google Translate's TTS endpoint
> (`translate.google.com/translate_tts`). It is unofficial and can change or
> rate-limit without notice.

## Install

1. In Node-RED: menu → **Import** → paste or upload `flows/radio-star.json`.
2. Install the palette dependency: **Manage palette → Install → `node-red-node-suncalc`**.
3. Copy the contents of `audio/` into your web server's
   `/share/htdocs/audio/tts/` directory.
4. Adjust the addresses/paths below to your network.
5. Deploy.

## Configuration

| Value | Default | Meaning |
| --- | --- | --- |
| `192.168.0.25` | media player | device that plays the announcement |
| `192.168.0.50` | audio/TTS server | web server hosting the generated `*.mp3` |
| `/share/htdocs/audio/tts/…` | output path | where TTS files are written |
| Cron times | see table | when each part runs |

No credentials are stored in the flow. Never commit `flows_cred.json`.

## Audio assets

The `audio/` folder contains the pre-generated TTS clips the flow uses, so it
works out of the box:

- **`0.mp3` … `23.mp3`** — spoken hour.
- **`Wetter prognose für heute <weekday>.mp3`** — spoken weekday intro (7 files).
- **`nachrichten.mp3`, `wetter.mp3`, `tageslaenge.mp3`** — pre-generated news /
  weather / day-length clips (the flow overwrites these at runtime).
- **`Sonnenaufgang.mp3`, `Sonnenuntergang.mp3`, `Sunrise.mp3`, `Sunset.mp3`,
  `sunHours.mp3`** — sunrise / sunset clips.
- **`notification.mp3`, `notification1.mp3`, `Notify.mp3`, `Titel.mp3`,
  `spacer.mp3`, `ding.mp3`, `halb.mp3`, `highbell24.mp3`** — jingles, chime and
  separators.
- **`final.mp3`** — a sample of the combined announcement (demo).
- **`audio/automation/`** — the related TTS set from the legacy HA-automation
  output directory.

No third-party music is bundled.

## Files

```
flows/radio-star.json    exported Node-RED tab (1 tab + 35 nodes)
audio/                   TTS clips, chime and a final.mp3 demo
audio/automation/        related TTS set
README.md                this documentation
LICENSE                  MIT
```

## Credits

- Built with **[OpenCode](https://opencode.ai)**, running in the
  [`sprisa/opencode`](https://hub.docker.com/r/sprisa/opencode) Docker image
  (source: [`sprisa/opencode-server`](https://github.com/sprisa/opencode-server)):

  ```sh
  docker pull sprisa/opencode:latest
  ```

- Built by **Big Pickle GO** 🥒

## License

MIT — see [LICENSE](LICENSE).

[![Sponsor this project](https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png)](https://buymeacoffee.com/hanshauberger)
