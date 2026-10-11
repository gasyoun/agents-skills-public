Stage only the artifact directories — leave `$WORKSPACE/*.log` and helper `.py` scripts out of the commit.

```sh
cd $MWS_ROOT
mkdir -p mwsissues/haiku_refresh/$REFRESH_DATE
cp $WORKSPACE/ISSUE_COMMENT.md       mwsissues/haiku_refresh/$REFRESH_DATE/
cp $WORKSPACE/comment_haiku_refresh.md mwsissues/haiku_refresh/$REFRESH_DATE/
cp $WORKSPACE/haiku_refresh_audit.md mwsissues/haiku_refresh/$REFRESH_DATE/

git add papers/microanalysis/figures/ \
        papers/microanalysis/analysis/SUPPLEMENT_MANIFEST.md \
        mwsissues/haiku_refresh/$REFRESH_DATE/
# Note: do NOT git-add the .zip — analysis/.gitignore excludes it by policy.
# Note: also copy h10_link_rot.log into the artifact dir so the audit evidence
#       is committed alongside the audit table:
cp $WORKSPACE/h10_link_rot.log mwsissues/haiku_refresh/$REFRESH_DATE/ 2>/dev/null || true

git commit -m "Round-2 Haiku refresh — $REFRESH_DATE

H5 palette + H1 renderers + H7 manifest regenerated. H2/H3/H4/H6/H8/H9/H10
audited (see mwsissues/haiku_refresh/$REFRESH_DATE/haiku_refresh_audit.md).

Co-Authored-By: Claude <executing tier + version, e.g. Haiku 4.5 (claude-haiku-4-5-20251001)> <noreply@anthropic.com>"

git push -u origin $WORK_BRANCH
```

If the `mwsissues/` directory doesn't exist yet, `ls` the repo and fall back to whatever existing `*issues/` directory the repo uses (mirroring how `cologne-markup-fix` handles per-repo naming).

---

