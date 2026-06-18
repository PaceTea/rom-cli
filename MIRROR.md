# Self-Hosted Mirror Mode

`rom-cli` can browse and download from **your own HTTP file server** instead of the Internet Archive. This is meant for your **personal dumps of games you own** — pull them onto a Steam Deck, handheld, or any other device over your network without USB shuffling.

```
rom-cli -m http://<your-server>/<path> [filter]
```

In mirror mode, `rom-cli` lists files from a standard HTTP directory index, walks one level of subdirectories, lets you fuzzy-pick (multi-select with `TAB`), and pulls the chosen files with `aria2`.

## How it works

Mirror mode needs nothing more than a server that emits a plain **directory index** of `<a href="...">` links. That includes:

- nginx with `autoindex on;`
- Apache / Caddy `file_server browse`
- `python3 -m http.server`
- FileBrowser's raw endpoint

It recursively reads **one level** of subdirectories, so a layout like this is fully browsable from a single command:

```
roms/
├── snes/Super Mario World.sfc
├── gba/Minish Cap.gba
└── nes/Metroid.nes
```

## Quick option reference

| Flag | Env var | Meaning |
|------|---------|---------|
| `-m, --mirror URL` | `ROM_CLI_MIRROR` | Base URL of your directory index |
| `-d, --download-dir` | `ROM_CLI_DOWNLOAD_DIR` | Where files land (default `~/Downloads/roms`, mirror files go to `…/roms/mirror`) |
| `-x, --connections` | `ROM_CLI_CONNECTIONS` | Parallel connections per file |

Set it permanently in your shell so you don't have to type `-m` each time:

```bash
echo 'export ROM_CLI_MIRROR="http://192.168.178.50/roms"' >> ~/.bashrc
```

Then just:

```bash
rom-cli              # lists everything on the mirror
rom-cli zelda        # filters by "zelda" (matches path + filename)
```

---

## Option A — nginx autoindex (recommended)

This gives the cleanest index for `rom-cli` to parse. On the box that holds your dumps:

```nginx
server {
    listen 80;
    server_name _;

    location /roms/ {
        alias /opt/sheska/data/roms/;   # adjust to where your dumps live
        autoindex on;
        autoindex_exact_size off;
        autoindex_localtime on;
    }
}
```

Reload and test:

```bash
nginx -t && systemctl reload nginx
curl -s http://localhost/roms/ | grep href
```

Then from your Steam Deck / PC:

```bash
rom-cli -m http://<server-ip>/roms zelda
```

---

## Option B — your existing FileBrowser (Sheska)

You already run FileBrowser on Sheska (`192.168.178.50:8080`) with `/opt/sheska/data` mounted at `/srv`. Two ways to use it:

**B1 — drop a folder in and serve via FileBrowser's raw path**

Put your dumps under `/opt/sheska/data/roms` on the host. FileBrowser exposes file content under a raw API path; point `rom-cli` at the directory:

```bash
rom-cli -m http://192.168.178.50:8080/api/raw/roms
```

> The exact raw path depends on your FileBrowser version and whether the share needs auth. If listing returns nothing, the index may be JSON rather than an HTML `href` index — in that case use Option A (a tiny nginx autoindex next to FileBrowser is the most reliable), or a "create share" link.

**B2 — public share link**

Create a share in the FileBrowser UI for your `roms` folder and use that share's URL as the mirror base. Good if you want a token-protected path rather than an open index.

---

## Option C — throwaway / on-demand index

If you just want to grab something quickly from any machine that has the files:

```bash
cd /path/to/your/roms
python3 -m http.server 8000
```

Then elsewhere:

```bash
rom-cli -m http://<that-machine-ip>:8000 metroid
```

Stop the server with `Ctrl-C` when done.

---

## Reaching it from outside your LAN

Your connection is DS-Lite / CGNAT, so classic port-forwarding won't work. Use the tunnels you already have:

- **Tailscale** — install on the server and the Steam Deck, then use the server's Tailscale IP as the mirror base (`rom-cli -m http://100.x.y.z/roms`). This is the clean, always-on option for your own devices.
- **Cloudflare Tunnel** — if you finish the named tunnel + `.tech` domain you were setting up, point `rom-cli` at `https://roms.yourdomain.tech`.

On the LAN itself, just use the `192.168.178.x` address directly — no tunnel needed.

---

## Notes

- Filenames with spaces work; `rom-cli` requests them as-is and `aria2` handles the encoding. If a specific server is picky, rename to underscores or ensure the index emits URL-encoded hrefs.
- Downloads are resumable — re-running a partial pull continues where it left off.
- Mirror mode only ever **reads** from the URL you give it; it never writes back to the server.
