# Gonzkart (WebAssembly)

Gonzkart running in the browser. This is a static site: no server, no build
step, no backend.

## Playing

1. Open the site.
2. Pick a texture quality (all three packs are included).
3. Press **Start Game** and wait for the first download to finish.

The first launch downloads the chosen data pack and keeps it in the browser's
IndexedDB (about 120-250 MB depending on quality). Later launches start from
that cache, so only the files in `overrides/` are re-fetched, which keeps
reloads fast.

### Controls

| Action | Keys |
| --- | --- |
| Drive / navigate menus | `W` `A` `S` `D` or arrow keys |
| Accelerate / brake | `W` / `S` |
| Steer | `A` / `D` |
| Nitro | `Shift` |
| Drift | `Space` |
| Look back | `B` |
| Pause | `Esc` |

`W`/`A`/`S`/`D` are forwarded to the arrow keys by a small script in
`index.html`, so both schemes work at the same time.

## How it works

```
index.html            start screen, key remapping, tab title
script.js             downloads + unpacks the data pack, applies overrides
coi-serviceworker.js  adds COOP/COEP headers (see below)
game/                 the emscripten build (.js + .wasm) and chunked data packs
overrides/            GUI textures applied on top of the data pack at launch
```

### Cross-origin isolation

The build uses pthreads, which need `SharedArrayBuffer`, which browsers only
expose to cross-origin isolated pages. That requires two response headers:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

Static hosts such as GitHub Pages cannot send custom headers, so
`coi-serviceworker.js` installs a service worker that injects them. It detects
when the real server already sends them and does nothing in that case.

### Data packs

`game/data_*.tar.gz.manifest` lists the size and the chunk file names; the
chunks are plain byte ranges of one gzipped tarball, which is why each file
stays comfortably below hosting file-size limits.

### Restyling

`overrides/index.json` lists files that are copied into the game's virtual
filesystem after the data pack is unpacked. Menu art lives there, so changing a
texture only means replacing a small file - no repack, no cache wipe, and the
player's save game is untouched.

## Running locally

Any static server with streaming support works. The simplest option that also
sends the isolation headers:

```powershell
# from the repository root
.\serve_local.ps1     # http://localhost:8000
```

Without those headers the service worker takes over, exactly as it does in
production.

## Credits

Gonzkart is a browser port based on SuperTuxKart, which is free software under
the GNU General Public License. See `COPYING` in the upstream project for the
full license text; the game data (tracks, karts, sounds) is licensed
separately by its respective authors.
