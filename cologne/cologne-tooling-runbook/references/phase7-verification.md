
```python
import sys
sys.stdout.reconfigure(encoding='utf-8')
# `gh` and `fetch_issues` come from "Fetching issues correctly" (above).

TYPE = {'bug','regression','tech-debt','dependency','security','feature','enhancement',
        'performance','documentation','docs-api','infrastructure','data-pipeline',
        'cross-repo','build-tooling','question','proposal','discussion'}
SEV  = {'trivial','minor','major','critical'}

issues = fetch_issues(REPO)                        # paginated, PRs excluded
no_type=[]; multi_type=[]; no_sev=[]; multi_sev=[]; no_ms=[]
for i in issues:
    n=i['number']; lbls=set(i['labels'])
    t=lbls & TYPE; s=lbls & SEV
    if not t: no_type.append(n)
    elif len(t)>1: multi_type.append((n,sorted(t)))    # stale default not removed
    if not s: no_sev.append(n)
    elif len(s)>1: multi_sev.append((n,sorted(s)))
    if i['milestone'] is None and 'discussion' not in lbls: no_ms.append(n)  # discussion exempt

bad = {k:v for k,v in {'no_type':no_type,'multi_type':multi_type,'no_sev':no_sev,
       'multi_sev':multi_sev,'no_ms':no_ms}.items() if v}
print(f"{len(issues)} issues — " + ("all complete ✓" if not bad else f"GAPS {bad}"))
```

**Hard gate**: every list must be empty before proceeding (`no_ms` lists only non-`discussion` issues, which are milestone-exempt). This paginated, PR-free, per-issue check is the authoritative "done?" gate. A non-empty `multi_type` means a stale GitHub default (`bug`, `enhancement`, `question`, `documentation`) was not removed in Phase 3.

---

