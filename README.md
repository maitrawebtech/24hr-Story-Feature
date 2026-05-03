# 📸 Stories

A client-side Instagram-style ephemeral story feature — images you post vanish automatically after 24 hours. No backend, no dependencies, just a single HTML file.

## Features

- **Upload stories** — click the `+` button to pick any image from your device
- **Auto-expiry** — stories disappear after 24 hours using `localStorage` timestamps
- **Full-screen viewer** — tap any story circle to open it full-screen
- **Progress bars** — one bar per story at the top, the active one fills over 3 seconds then auto-advances
- **Swipe navigation** — swipe left/right (or use arrow keys on desktop) to move between stories
- **Seen state** — viewed stories get a dimmed ring, unseen ones have a coloured gradient ring
- **Expiry warning** — an orange dot appears on stories with less than 2 hours remaining
- **Delete** — remove a story early from inside the viewer
- **Image resizing** — uploaded images are capped at 1080×1920px client-side before storage to avoid `localStorage` quota issues

## How It Works

### Storage

Stories are serialised as JSON and stored in `localStorage` under the key `maitra_stories_v1`. Each entry looks like:

```json
{
  "id": "uuid-v4",
  "img": "data:image/jpeg;base64,...",
  "createdAt": 1714000000000,
  "seen": false
}
```

On every read, entries older than 24 hours are filtered out and the cleaned list is written back. A `setInterval` also runs every 60 seconds to purge expired stories while the app is open.

### Image Pipeline

```
File input → FileReader → Canvas resize (max 1080×1920) → JPEG base64 → localStorage
```

Compression is set to 0.82 quality to balance size and visual fidelity.

### Viewer & Progress

Each story displays for **3 seconds**. A `setInterval` running at ~30ms updates the active progress bar fill width linearly. When it hits 100% it clears itself and calls `showStory(idx + 1)`, which closes the viewer automatically after the last story.

## Running It

No build step needed. Just open the file in a browser:

```bash
open stories.html
# or serve it locally
npx serve .
```

Works on Chrome, Firefox, Safari, and mobile browsers.

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `→` | Next story |
| `←` | Previous story |
| `Esc` | Close viewer |

## Constraints & Limitations

- **Storage limit** — browsers typically allow 5–10 MB of `localStorage` per origin. High-resolution images compressed to JPEG/0.82 average ~200–400 KB each, so expect room for roughly 10–25 stories before hitting the limit.
- **No persistence across origins** — stories are tied to the origin (domain + port) the file is served from.
- **24h timer is wall-clock** — if the user's system clock changes, expiry behaviour may shift accordingly.
- **Single user** — this is a personal story shelf; there is no multi-user or sharing layer.

## Tech Stack

| Concern | Approach |
|---------|----------|
| Storage | `localStorage` |
| Image resizing | `<canvas>` + `toDataURL` |
| Unique IDs | `crypto.randomUUID()` |
| Fonts | Google Fonts (DM Sans + DM Serif Display) |
| Styling | Vanilla CSS with custom properties |
| Framework | None — single HTML file |
