# Simple Stack Exchange Archiver

Archives any Stack Exchange user profile into a self-contained static website. One command, one directory, everything works offline.

## What it does

1. Downloads all 5 profile tab pages (top, accounts, reputation, activity, subscriptions)
2. Detects and captures paginated content (questions, answers, activity)
3. Downloads every linked question page across all SE network sites
4. Runs a generated stitch script that rewrites navigation to local files, inlines pagination with JS switchers, localizes question/answer links, and removes dead external links
5. Cleans up temp files

The output is a directory you can serve with any static file server or open directly in a browser.

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

# optionally symlink into PATH
ln -s "$PWD/se-archive" ~/.local/bin/se-archive
```

## Usage

```sh
se-archive <stackexchange_user_url>
```

The URL should be the network profile page on stackexchange.com:

```sh
# archive a math.SE user
se-archive https://stackexchange.com/users/3364210/cleo

# archive a stackoverflow user
se-archive https://stackexchange.com/users/12345/someone

# serve the result
cd ~/dev/cleo
python3 -m http.server 8000
```

Archives are saved to `~/dev/<username>/`.

## Supported sites

Works with any site in the Stack Exchange network:

- Stack Overflow
- Mathematics, MathOverflow
- Super User, Server Fault, Ask Ubuntu
- All `*.stackexchange.com` sites (Physics, TeX, CS Theory, etc.)

Question pages are prefixed by site: `math-562694.html`, `so-16452383.html`, `mo-130500.html`, `physics-12345.html`, etc.

## Output structure

```
<username>/
  index.html          # top questions + top answers with JS pagination
  accounts.html       # network accounts
  reputation.html     # reputation history
  activity.html       # recent activity with JS pagination
  subscriptions.html  # filter subscriptions
  stitch.py           # generated stitcher, can be re-run
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

The stitch script processes each profile page in order: rewrite tabs, inline pagination, rewrite question links, kill external links. This ordering matters because inlined pagination content needs its links rewritten before external link removal runs.

## Limitations

- Cloudflare or rate limiting may block some question page downloads. The script warns when a page is suspiciously small (<5KB). Re-running skips pages already downloaded.
- The "Hot Network Questions" sidebar links to unrelated questions that aren't downloaded.
- CSS and images still load from SE CDN servers, so styling requires an internet connection.
- Only pagination works client-side. Voting, commenting, and other interactive features are dead by design.

## License

Public domain.
