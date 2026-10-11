## Phase 5b — Batch labeling script

Pattern (write to `<repo>_label.py`, run in background):

```python
import subprocess, json, sys
sys.stdout.reconfigure(encoding='utf-8')
sys.stderr.reconfigure(encoding='utf-8')

types = {
    'bug': [<issue numbers>],
    'feature': [<issue numbers>],
    # ...
}
ms_map = { 'bug': <ms_num>, 'feature': <ms_num>, ... }   # built from API
sev_overrides = { <num>: 'major', <num>: 'critical', ... }
domain_tags = { <num>: 'domain:api', ... }   # optional

total = sum(len(v) for v in types.values())
all_nums = set(n for v in types.values() for n in v)
assert total == len(all_nums) == <expected_count>

errors = []
done = 0
for t, nums in types.items():
    ms = ms_map[t]
    default_sev = {  # from Phase 4 table
        'bug':'minor','regression':'major','security':'major',
        'feature':'major','enhancement':'minor','performance':'minor',
        'tech-debt':'minor','dependency':'minor',
        'documentation':'trivial','docs-api':'trivial',
        'infrastructure':'minor','data-pipeline':'major',
        'cross-repo':'major','build-tooling':'minor',
        'question':'trivial','proposal':'trivial','discussion':'trivial',
    }.get(t, 'minor')

    for n in nums:
        sev = sev_overrides.get(n, default_sev)
        labels_to_add = [t, sev]
        if n in domain_tags:
            labels_to_add.append(domain_tags[n])

        for label in labels_to_add:
            r = subprocess.run(['gh','api',f'repos/$ORG/$REPO/issues/{n}/labels',
                '-X','POST','-f',f'labels[]={label}'], capture_output=True, encoding='utf-8')
            if r.returncode != 0 and '422' not in r.stderr:
                errors.append(f'label {label} #{n}: {r.stderr[:80]}')

        r = subprocess.run(['gh','api',f'repos/$ORG/$REPO/issues/{n}',
            '-X','PATCH','-f',f'milestone={ms}'], capture_output=True, encoding='utf-8')
        if r.returncode != 0:
            errors.append(f'milestone #{n}: {r.stderr[:80]}')
        done += 1
        print(f'{done}/{total} #{n} {t}/{sev}/ms{ms}', flush=True)

print('DONE no errors.' if not errors else f'ERRORS ({len(errors)}):')
for e in errors: print(' ', e)
```

---

