# Publishing — repo, Pages, archives

---

## Repo layout

```
course-repo/
├── index.html                     meta-refresh redirect to the manual
├── manual-name.html               the deliverable
├── companion.html                 optional
├── README.md                      repo front page
├── CAPTURE-GUIDE.md               maintainer-only: figures to take, front/back boundary
├── .gitignore
├── images/                        tracked — this is where figures land
├── templates/                     optional downloadable starter folder
│   ├── PROJECT_2026_project-name/
│   ├── project-folder.zip
│   ├── build-zip.py
│   └── README.md
├── tools/audit.py
└── .github/workflows/audit.yml
```

`index.html` is a redirect, not a landing page — one destination, no choice to make:

```html
<!DOCTYPE html><html lang="fr"><head><meta charset="UTF-8">
<meta http-equiv="refresh" content="0; url=manual-name.html">
<title>Course title</title></head>
<body><p><a href="manual-name.html">Course title →</a></p></body></html>
```

Name the manual so it is identifiable in a downloads folder in three years. Not `index.html`.

## .gitignore

Student material must never land in the repo. Films, generations, weights and source documents are all things someone will `git add -A` by accident:

```
*.pdf  *.docx  *.pptx  *.xlsx        # source documents — the manual supersedes them
*.mp4  *.mov  *.wav  *.mp3           # student media
*.psd  *.ply  *.safetensors  *.ckpt  # working files and weights
__pycache__/  *.pyc
output/
# images/ is deliberately NOT ignored — figures are tracked
```

Add that last comment. Someone will otherwise "tidy" `images/` into the ignore list and silently break every figure.

## Pages

Settings → Pages → Deploy from a branch → `main` / root. Live at `https://<user>.github.io/<repo>/` within a minute.

No Jekyll config needed. Add `.nojekyll` only if you introduce `_`-prefixed paths, which Jekyll drops.

With `gh`:

```bash
gh repo create <user>/<repo> --public --source=. --remote=origin --push
gh api -X POST repos/<user>/<repo>/pages -f 'source[branch]=main' -f 'source[path]=/'
gh run list --limit 3                     # both the audit and the Pages build
curl -s -o /dev/null -w "%{http_code}\n" -L https://<user>.github.io/<repo>/
```

Always verify the live URL, not just the green check. A push can succeed, CI can pass, and the page still 404 because Pages was never enabled.

## CI

Copy `assets/audit.yml` to `.github/workflows/`. Two steps: run the audit, and verify the downloadable archive is not stale. Both are cheap; neither needs dependencies.

---

## Deterministic archives

If the course ships a starter folder, students get a `.zip`. The problem: the tree and the archive drift apart silently, and nobody notices until a student unzips last term's folder.

The fix is a CI check that rebuilds the archive and compares hashes — which only works if the build is **byte-reproducible**. Three things break that, and all three were found the hard way:

**1 · Timestamps.** `ZipInfo` defaults to the file's mtime, which a fresh `git clone` rewrites. Use a fixed stamp:

```python
STAMP = (2026, 1, 1, 0, 0, 0)     # any fixed date; never datetime.now()
```

**2 · Line endings.** A Windows working tree holds CRLF; the CI checkout holds LF. Same content, different bytes. Normalise text entries inside the archive:

```python
TEXT = {".md", ".csv", ".txt", ".json"}
if os.path.splitext(arcname)[1].lower() in TEXT:
    data = data.replace(b"\r\n", b"\n").replace(b"\r", b"\n")
```

**3 · `create_system`.** `ZipInfo` sets it to 0 on Windows and 3 elsewhere. That one byte per entry is enough to make the hash differ between a laptop and CI. Pin everything platform-dependent:

```python
info = zipfile.ZipInfo(name, STAMP)
info.create_system = 3        # Unix, on every host
info.create_version = 20
info.extract_version = 20
```

Also: track empty directories with `.gitkeep`, exclude those files from the archive, and re-create the empty directories as real zip directory entries — a student's folder should not contain your git plumbing.

Make the CI failure message print **both hashes and the differing entries**. "The zip is stale" sends you looking in the wrong place; "content differs: 03_breakdown/shot_sheet.csv" does not.

Name the archive after the folder inside it. A `PROJECT_2026_x/` tree inside `starter.zip` confuses every student exactly once, which is once too often.

---

## The README

The repo front page is read by colleagues and by students who land on GitHub rather than the manual. It should carry:

- **The live link first**, stated as the link to give students.
- A link per companion page, each with one line on what it is for.
- The offline instruction: download the `.html`, open it in any browser, works from a USB key.
- A chapter table — the fastest way for a colleague to judge the manual.
- What is in it: chapter count, word count, figure count.
- The course structure: deliverables, the year table.
- The template download.
- Credits for the tools used, with links.
- A note on the audit, so a contributor knows to run it.

Emoji are fine in the README. They are not fine in the manual.

**README numbers go stale silently.** Word count, file size, chapter count, "ten figures" — every substantial edit invalidates them, and no check catches it. Re-derive them whenever you reframe anything.

---

## Mistakes that have actually cost time

Recorded because each was expensive and none was obvious.

**A destructive test in the repo.** A line-ending experiment ran `shutil.rmtree` on the template tree; the restore step then crashed on a non-ASCII character in the home path; the next `git add -A` staged 24 deletions and they were committed. CI caught it, and recovery was `git checkout <sha> -- templates/PROJECT`.
→ **Copy to a scratch directory before any destructive test. Never mutate the repo in place to test a hypothesis.**

**A filter that filtered nothing.** The band CSS and JS were written before any content carried `data-level`. The buttons highlighted correctly, so it looked like it worked, and nothing was hidden for weeks.
→ **After wiring a filter, set it to the most restrictive value and confirm content actually disappears.**

**Content trapped in one language wrapper.** Ten figures lived inside the FR block. The English edition simply did not have them, and no reader of the French edition could tell.
→ **Read both editions end to end at least once.**

**`main()` at import.** `tools/audit.py` ran the whole audit on import, so reusing `view()` from another script ran the audit as a side effect.
→ **`if __name__ == "__main__": main()`.** Cheap, and makes the passes reusable.

**Windows path and encoding traps.** Bash `/tmp` is not visible to Windows Python; non-ASCII characters in the home path break paths passed through argv; printing `│` or `·` dies in `cp1252`.
→ Use the scratchpad, derive paths from `__file__` or `os.path.expanduser`, and set `PYTHONIOENCODING=utf-8` before printing anything non-ASCII. A crash at a `print` looks like a logic bug and is not.

**Stale README numbers after a reframe.** Word count, file size, a percentage, "ten figures", and a sentence describing a requirement that had been removed two commits earlier — all still in the README, all wrong.
→ Re-derive after every reframe.

---

## Commit messages

These repos use full-prose commit messages, and it pays off: the commit log is the only record of *why* a decision was made. State what changed, why, and what was verified.

Bad: `update manual`. Good: names the sections touched, the reason, and "audit passes 7/7, 24 external links verified 200".

Say what you did **not** do, too. A commit that fixes two of three findings should say which one is outstanding.
