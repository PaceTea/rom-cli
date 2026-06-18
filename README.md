# rom-cli

A tiny, ani-cli-style command-line tool to **search and download legal ROMs, homebrew, and public-domain games** straight from the [Internet Archive](https://archive.org).

No scraping, no copyrighted/commercial sources — it talks only to the public Internet Archive API and restricts searches to openly-licensed / public-domain / homebrew collections.

```
search → fzf menu → pick item → pick file(s) → aria2 download → done
```

## Features

- 🔎 Fuzzy search via `fzf`, just like ani-cli
- 📦 Multi-file selection within an archive item (`TAB` to mark)
- ⚡ Fast, resumable, parallel downloads via `aria2`
- 🧩 Configurable collections — point it at any IA collection you like
- 🐧 Pure Bash, no runtime beyond the four deps

## Dependencies

- `curl`
- `jq`
- `fzf`
- `aria2`

On CachyOS / Arch:

```bash
sudo pacman -S curl jq fzf aria2
```

## Install

### From the AUR-style PKGBUILD

```bash
git clone https://github.com/PaceTea/rom-cli.git
cd rom-cli
makepkg -si
```

### Manual

```bash
sudo install -Dm755 rom-cli /usr/bin/rom-cli
```

## Usage

```bash
rom-cli                       # interactive: prompts for a search term
rom-cli homebrew snes         # search directly
rom-cli -c opensource_media doom
rom-cli --help
```

### Options

| Option | Description | Default |
|--------|-------------|---------|
| `-d, --download-dir DIR` | Download target | `~/Downloads/roms` |
| `-c, --collection LIST` | `;`-separated IA collections to search | `open_source_software;softwarelibrary_misc;homebrew` |
| `-n, --results N` | Number of search results | `50` |
| `-x, --connections N` | Parallel connections per download | `8` |
| `-V, --version` | Print version | |
| `-h, --help` | Show help | |

### Environment variables

All options have matching env vars: `ROM_CLI_DOWNLOAD_DIR`, `ROM_CLI_COLLECTIONS`, `ROM_CLI_RESULTS`, `ROM_CLI_CONNECTIONS`.

## Self-hosted mirror

`rom-cli` can also browse **your own HTTP file server** instead of the Internet Archive — handy for pulling your personal dumps of games you own onto a Steam Deck or handheld without USB shuffling:

```bash
rom-cli -m https://example.com/roms zelda
```

See [MIRROR.md](MIRROR.md) for full setup (nginx autoindex, FileBrowser, Tailscale/Cloudflare for remote access).

## Finding collections

Browse [archive.org](https://archive.org) and look at the URL of a collection page — the part after `/details/` is the collection id you pass to `-c`. Good legal starting points: homebrew scenes, public-domain software libraries, and openly-licensed game collections.

## Legal note

rom-cli is intended for **freely distributable** content only — homebrew, public domain, and openly-licensed material. Please respect copyright; don't point it at collections you don't have the right to download.

## License

GPL-3.0
