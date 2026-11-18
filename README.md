# Node-RED – Radio-Star

A **self-contained morning radio announcement** flow for Node-RED: it gathers
weather + day length + news, turns them into speech, and plays them on a
network media player with a volume ramp.

It needs **no Home Assistant** — everything is fetched and generated inside
Node-RED. This repo holds the exported flow, its assets and documentation.

## What the flow does

| Trigger (cron) | Step |
| --- | --- |
| `:26` / `:56` | Scrape weather from `wien.gv.at` (200-split, highbell at the end) |
| `:25` / `:55` | ORF news (`rss.orf.at`) → TTS + highbell per item |
| `23:54` | Day length for Vienna (`sunrise` node) → TTS |
| `00:59` / `12:59` | Sunrise / sunset TTS |
| `:59:30` | Pick a random jingle (1–4) |
| `:59:40` | Combine everything into `final.mp3` |
| `RADIO STAR` (manual inject) | Play `final.mp3` on the media player |

Playback goes to the media player at **`192.168.0.25`** (its `httpapi.asp`
endpoint), which pulls the audio from the web server on **`192.168.0.50`**;
the volume ramps **60 → 75 → 85 → 100**.

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
3. Copy the audio assets into your web server's `/share/htdocs/audio/tts/`:
   - `highbell24.mp3` (included in this repo)
   - `random_1.mp3` … `random_4.mp3` (your own jingles — not included, see below)
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

- `audio/highbell24.mp3` — the chime played between items (included).
- `random_1.mp3` … `random_4.mp3` — **not included**: these are third-party
  music tracks. Supply your own jingles with those names, or edit the
  "Random 1–4" node.

## Files

```
flows/radio-star.json    exported Node-RED tab (1 tab + 43 nodes)
audio/highbell24.mp3     chime used by the flow
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
