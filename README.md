# Patch Bay

A browser-based audio-to-tab transcription proof of concept.

Drop in a monophonic guitar clip (single-note melody or solo). YIN pitch detection runs entirely in your browser — no upload. The detected note events are sent to Claude, which returns ASCII guitar tablature with fingerings chosen for playability.

## What works

- Single-note lines: melodies, riffs, picked solos, whistling, humming.
- Any tuning and capo position (as long as you tell it).
- All common audio formats (MP3, WAV, OGG, M4A, FLAC).

## What doesn't (yet)

- **Polyphony.** YIN picks one pitch per frame. Strummed chords will produce garbage. See the upgrade path below.
- **Rhythm.** Durations come from pitch stability, not beat detection. The tab is positionally correct but rhythmically suggestive.
- **Expression.** Bends, slides, hammer-ons, vibrato are not detected.

## Running locally

It's a single HTML file. Open it in a browser. That's it.

```bash
# A local server is recommended to avoid strict CORS on some browsers:
python3 -m http.server 8000
# then open http://localhost:8000
```

## API key setup

The app calls the Anthropic API directly from the browser. For local use, paste your API key into the key field in the UI — it lives only in that browser session and is never stored or committed.

For a shareable or hosted version, route the call through a proxy that supplies the key. The simplest option is a Cloudflare Worker:

```javascript
export default {
  async fetch(request, env) {
    const body = await request.text();
    return fetch("https://api.anthropic.com/v1/messages", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "x-api-key": env.ANTHROPIC_API_KEY,
        "anthropic-version": "2023-06-01",
      },
      body,
    });
  },
};
```

Then change the fetch URL in `index.html` to point at your worker.

**Do not commit your API key.**

## Upgrade path to polyphony

Replace the `analyze()` function's pitch-detection loop with `@spotify/basic-pitch`. The note-event contract (`{midi, start, end, duration}`) is unchanged, so everything downstream keeps working.

```bash
npm install @spotify/basic-pitch onnxruntime-web
```

Budget ~17 MB for the model weights and ~1–3 s of additional first-load time.

## Architecture

```
Browser: File → Web Audio → YIN (pitchy) → note segmenter → note events JSON
                                                                    ↓
                                                         Claude API → ASCII tab
```

No server. Audio never leaves the browser until it has been reduced to a small JSON payload of notes.

## License

MIT
