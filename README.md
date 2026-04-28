# Simple Stack Exchange Archiver

A single shell script that archives any Stack Exchange user profile into a fully self-contained static website. All pages, questions, answers, and navigation work offline with no external dependencies.

## What it does

1. Downloads the 5 profile tab pages (top, accounts, reputation, activity, subscriptions)
2. Detects and captures all pagination (questions, answers, activity)
3. Downloads every linked question page across all SE sites
4. Generates and runs a stitch script that:
   - Rewrites tab navigation to local files
   - Inlines paginated content with client-side JS switchers
   - Rewrites all question/answer links to local pages
   - Kills remaining external links (converts to non-clickable spans)
5. Cleans up temp files

The result is a directory you can serve with any static file server or browse directly from the filesystem.

## Requirements

- POSIX shell (`sh`)
- `curl`
- `grep` with `-P` (Perl regex) support
- Python 3 (for the stitch step)

## Installation

```sh
git clone https://github.com/krisyotam/sse-archiver.git
cd sse-archiver
chmod +x se-archive

# symlink into PATH (optional)
ln -s "$PWD/se-archive" ~/.local/bin/se-archive
```

## Usage

```sh
se-archive <stackexchange_user_url>
```

The user URL is the network profile page on stackexchange.com:

```sh
# archive a math.SE user
se-archive https://stackexchange.com/users/3364210/cleo

# archive a stackoverflow user
se-archive https://stackexchange.com/users/12345/someone

# serve the result
cd ~/dev/cleo
python3 -m http.server 8000
```

The archive is saved to `~/dev/<username>/`.

## Supported sites

Any site in the Stack Exchange network:

- Stack Overflow
- Mathematics, MathOverflow
- Super User, Server Fault, Ask Ubuntu
- All other `*.stackexchange.com` sites (Physics, TeX, CS Theory, etc.)

Question pages are prefixed by site: `math-562694.html`, `so-16452383.html`, `mo-130500.html`, `physics-12345.html`, etc.

## Output structure

```
<username>/
  index.html          # top questions + top answers (with JS pagination)
  accounts.html       # network accounts
  reputation.html     # reputation history
  activity.html       # recent activity (with JS pagination)
  subscriptions.html  # filter subscriptions
  stitch.py           # generated stitcher (can be re-run)
  questions/
    math-562694.html   # individual question pages
    so-16452383.html
    mo-130500.html
    ...
```

## How it works

The script runs in 6 phases:

| Phase | Description |
|-------|-------------|
| 1 | Download the 5 profile tab pages |
| 2 | Detect and download question pagination pages |
| 3 | Detect and download activity pagination pages |
| 4 | Detect and download answer pagination pages |
| 5 | Extract all question URLs from every page, download each one |
| 6 | Generate `stitch.py`, run it to rewrite links and inline pagination |

The stitch script processes each profile page in order: rewrite tabs, inline pagination, rewrite question links, kill external links. This ordering ensures inlined pagination content gets its links rewritten before external link removal.

## Limitations

- Some question pages may be blocked by Cloudflare or rate limiting. The script warns when a downloaded page is suspiciously small (<5KB). Re-running the script skips already-downloaded pages.
- The "Hot Network Questions" sidebar on question pages contains links to unrelated questions that are not downloaded.
- Images and CSS are loaded from SE CDN servers; the archive is not fully offline for styling/images.
- JavaScript functionality beyond pagination (voting, commenting, etc.) is non-functional by design.

## License

Public domain. Do whatever you want with it.
