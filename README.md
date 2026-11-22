# SodDrop

100 PVP Scrunchy Goodness — a 3D battle-royale-style shooter that runs in the browser.

Solo free-for-all, or open a party and drop with friends. Pick **SOLO / DUO / TRIO / SQUAD**:
your team is filled with bots when nobody is around, and the rest of the lobby is bots too.

## Files

| Path | What it is |
| --- | --- |
| `index.html` | The whole game — HTML, CSS and the three.js client |
| `audio/menu-music.mp3` | Menu track, extracted out of the HTML |
| `audio/storm-music.mp3` | In-match track, extracted out of the HTML |
| `worker/index.js` | Cloudflare Worker + Durable Object party server |
| `wrangler.toml` | Cloudflare deploy config (assets + `PARTY` binding) |
| `.assetsignore` | Keeps `source/`, `worker/` etc. out of the deployed upload |
| `source/storm_drop_3d_pirate.html` | Reference build (locker, shop, OG pass), kept for history |
| `source/storm_drop_3d_smaller.html` | Original single-file build (audio still embedded) |

Audio used to be two ~1.2 MB base64 blobs inline in the page; it is now ordinary files served
next to it, so `index.html` stays ~126 KB.

## Play

Serve the folder over HTTP (any static server works) and open it:

```bash
python3 -m http.server 8080
# http://localhost:8080/
```

Click anywhere to start the music, then **READY UP**. Solo play needs no server; party play does.

## Party (Cloudflare multiplayer)

Multiplayer needs a WebSocket server, so it runs on a Cloudflare Worker with one Durable
Object per party code. The same deploy serves the page and the audio.

```bash
npx wrangler login      # once
npx wrangler deploy
```

Then open `https://soddrop.<your-subdomain>.workers.dev`.

### Auto-deploy on push

`.github/workflows/deploy.yml` deploys on every push to `main` (and on a manual
`workflow_dispatch`), so a commit is all it takes. Add two repository secrets under
**Settings → Secrets and variables → Actions → New repository secret**:

| Secret | Where to get it |
| --- | --- |
| `CLOUDFLARE_API_TOKEN` | Cloudflare dashboard → **My Profile → API Tokens → Create Token → "Edit Cloudflare Workers"** template |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare dashboard → **Workers & Pages** → *Account ID* in the right sidebar |

The workflow runs `npx wrangler@4 deploy` from the repo root, so it publishes `index.html`,
`audio/` and `worker/index.js` exactly like a manual deploy (`.assetsignore` still applies).
If the deploy step reports a Durable Objects permission error, edit the token and add
**Workers Durable Objects: Edit** to it.

- **CREATE PARTY** makes a 5-character code, or type a friend's code and **JOIN**.
- Everyone presses **READY UP**; when the whole party is ready a shared countdown starts and
  every player loads into the same match.
- The lobby shows the party: each member stands on their own pedestal with a name plate, and
  the roster lists team + ready state.
- Change **YOUR NAME** any time — before or during a party; it updates everywhere.
- **BACK TO LOBBY** on the results card returns to the lobby without reloading, so the party
  stays together.
- The host picks the team size; everyone else follows.

Hosting the page somewhere else (e.g. GitHub Pages)? Pass the worker origin:
`index.html?server=https://your-worker.workers.dev`.

## What's in a match

- **Battle bus** path is derived from a per-match seed the server sends, so every player in the
  party rides the identical route and can drop together.
- **Teams**: SOLO/DUO/TRIO/SQUAD. Bots fill your team when you are short-handed; teammates are
  marked with a green name plate and are never targeted by you or by friendly bots.
- **Bots have name plates** above their heads (long names are cut off with `…`).
- **Full map on `M`** — island map with storm ring, bus route, teammates, enemies and every
  named location: SUPER SOD HQ, Puddle Plaza, Mud Flats, Sod Springs, Compost Canyon,
  Dirt Docks, The Rootworks, Grub Gully, Brickyard, Grassfield.
- **Sod** is the character: blocky dirt head with grass, white eyes and a blue shirt.

## Controls

| Key | Action |
| --- | --- |
| `WASD` | Move |
| `SHIFT` | Sprint |
| `SPACE` | Glide / jump |
| Mouse | Aim, hold to fire, right button to ADS |
| `R` / `1-5` / wheel | Reload / switch weapon |
| `Z` `X` `C` | Wall / floor / ramp |
| `E` | Open chest |
| `M` | Full map |
| `ESC` | Pause |

## Netcode notes

- Positions stream at 10 Hz; remote players are interpolated client-side.
- Damage is reported by the shooter and applied by the target, with the server clamping each
  hit to 1–45. Fine for friends, not tournament-grade.
- Bots are simulated locally on each client (they are atmosphere, not authority); humans are
  synced through the Durable Object.
- A party code is a live session: it empties out and resets once everyone leaves.
