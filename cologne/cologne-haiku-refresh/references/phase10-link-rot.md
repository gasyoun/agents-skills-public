Scan the prose docs for broken external links. Files in scope (all on `docs-pass`):

- [PAPER.md](https://github.com/sanskrit-lexicon/MWS/blob/docs-pass/papers/microanalysis/PAPER.md)
- [IJL_COVER_LETTER.md](https://github.com/sanskrit-lexicon/MWS/blob/docs-pass/papers/microanalysis/IJL_COVER_LETTER.md)
- [PAPER_RU.md](https://github.com/sanskrit-lexicon/MWS/blob/docs-pass/papers/microanalysis/PAPER_RU.md)
- [DOUBTS.md](https://github.com/sanskrit-lexicon/MWS/blob/docs-pass/papers/microanalysis/DOUBTS.md)
- [HANDOFF.md](https://github.com/sanskrit-lexicon/MWS/blob/docs-pass/papers/microanalysis/HANDOFF.md)
- `papers/microanalysis/decisions/*.md`
- `sanskrit-lexicon/csl-atlas` chapter `*.md` files (clone or `gh api` as needed)

**Counting URLs is NOT auditing them.** This phase must actually HEAD each unique URL with a 5s timeout. Write `$WORKSPACE/link_rot.py`:

```python
"""Real H10 link-rot scan — HEAD with 5s timeout + GET fallback for 403/405/501.
Parallel via ThreadPoolExecutor (16 workers). Audit-only — never edits any file."""
import re, sys, time
from pathlib import Path
from collections import defaultdict
from concurrent.futures import ThreadPoolExecutor, as_completed
import requests
sys.stdout.reconfigure(encoding='utf-8')

MA = Path(sys.argv[1])
SCOPED = [MA/'PAPER.md', MA/'IJL_COVER_LETTER.md', MA/'PAPER_RU.md',
          MA/'DOUBTS.md', MA/'HANDOFF.md'] + sorted(MA.glob('decisions/*.md'))
URL_RX = re.compile(r'https?://[^\s\)\]\>"`]+')
HEADERS = {'User-Agent': 'cologne-haiku-refresh/1.0 (link-rot audit)'}
TIMEOUT = 5

urls = defaultdict(set)
for f in SCOPED:
    if not f.exists(): continue
    for m in URL_RX.finditer(f.read_text(encoding='utf-8', errors='ignore')):
        urls[m.group(0).rstrip('.,;:')].add(f.name)

def check(u):
    try:
        r = requests.head(u, headers=HEADERS, timeout=TIMEOUT, allow_redirects=True)
        if r.status_code in (403, 405, 501):
            r = requests.get(u, headers=HEADERS, timeout=TIMEOUT,
                             allow_redirects=True, stream=True); r.close()
        return (u, r.status_code, None)
    except requests.exceptions.Timeout: return (u, None, 'timeout')
    except Exception as e: return (u, None, f'{type(e).__name__}: {str(e)[:60]}')

results = {}
with ThreadPoolExecutor(max_workers=16) as p:
    for fut in as_completed({p.submit(check, u): u for u in urls}):
        u, c, e = fut.result()
        results[u] = (c, e)

bad = [(u, c, e) for u, (c, e) in results.items()
       if e or (c and (c == 404 or c >= 500))]
print(f"Unique URLs: {len(urls)}; bad: {len(bad)}")
by_file = defaultdict(list)
for u, c, e in bad:
    label = str(c) if c else e
    for f in urls[u]: by_file[f].append((u, label))
for fname in sorted(by_file):
    print(f"\n{fname} ({len(by_file[fname])} bad):")
    for u, lbl in by_file[fname][:25]: print(f"  [{lbl}] {u[:100]}")
```

Run, save log to `$WORKSPACE/h10_link_rot.log`, copy to `mwsissues/haiku_refresh/$REFRESH_DATE/h10_link_rot.log` so it gets committed as audit evidence.

```sh
python $WORKSPACE/link_rot.py $MA 2>&1 | tee $WORKSPACE/h10_link_rot.log
```

Append `| H10 link-rot | ⚠️ | <UNIQUE> URLs checked, <K> broken — see h10_link_rot.log |` to the audit. **Audit-only — do not edit prose files.** Fixes for broken URLs belong to the Sonnet 5 / Opus 4.8 passes.

---

