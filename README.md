# SodDrop

100 PVP Scrunchy Goodness — a 3D battle-royale-style shooter that runs in the browser.

Free-for-all only (no teams): jump off the battle bus, loot chests, build, and be the last
one standing. Play alone against bots, or open a party and fight your friends.

## Files

| Path | What it is |
| --- | --- |
| `index.html` | The whole game — HTML, CSS and the three.js client |
| `audio/menu-music.mp3` | Menu track, extracted out of the HTML |
| `audio/storm-music.mp3` | In-match track, extracted out of the HTML |
| `worker/index.js` | Cloudflare Worker + Durable Object party server |
| `wrangler.toml` | Cloudflare deploy config (assets + `PARTY` binding) |
| `.assetsignore` | Keeps `source/` etc. out of the deployed upload |
| `source/storm_drop_3d_smaller.html` | Original single-file build (audio still embedded), kept for reference |

The two tracks used to be ~1.2 MB base64 blobs inline in the page; they are now ordinary
files served next to it, so `index.html` is ~86 KB.

## Play

Serve the folder over HTTP (any static server works) and open it:

```bash
python3 -m http.server 8080
# http://localhost:8080/
```

Click anywhere to start the music, then **READY UP** for a solo match against bots.

## Party (Cloudflare multiplayer)

Multiplayer needs a server, so it runs on a Cloudflare Worker with one Durable Object per
party code. The worker also serves the page and the audio, so a single deploy gives you a
complete game.

```bash
npx wrangler login
npx wrangler deploy      # from the repo root
```

Wrangler prints a URL like `https://soddrop.<your-subdomain>.workers.dev`. That's the game.

1. One player clicks **CREATE PARTY** — a 5-character code appears (COPY LINK copies an invite).
2. Friends type the code and hit **JOIN** (up to 10 players per party).
3. Everyone flips to **READY UP**. When the whole party is ready, a 5-second countdown runs
   and everybody drops into the same match at once.
4. Last player standing wins. **BACK TO LOBBY** returns you to the same party for a rematch.

**Hosting the page somewhere else (GitHub Pages, etc.)?** Deploy the worker, then point the
page at it — either set `PARTY_SERVER` in `index.html` to your worker origin, or open the page
with `?server=https://your-worker.workers.dev`. Without a worker the page still plays solo;
joining a party just reports that the server is unreachable.

### How the netcode works

- The Durable Object owns the roster, the ready flags, the countdown and the match phase, so
  everyone starts on the same tick.
- Each client sends its own position/rotation/state about 10 times a second; the room relays
  it to the other players, who are drawn as name-tagged characters and interpolated locally.
- Hits are reported by the shooter and replayed on the target (`hit` → local damage), with
  damage clamped server-side. Good enough between friends; not tournament-grade.
- Bots are local filler (12 per party match, 40 in solo) and are not synced — the humans are.

## Controls

`WASD` move · `SHIFT` sprint · `SPACE` glider/jump · mouse aim · click fire · right click ADS
· `R` reload · `Z`/`X`/`C` wall/floor/ramp · `E` open chest · `ESC` pause
