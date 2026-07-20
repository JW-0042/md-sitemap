# Transfer this repo to JW-0042

The protocol was authored for the **JW-0042** GitHub namespace. The first publish used the authenticated CLI account `vtf587-boop` because write access to `JW-0042` was not available in that session.

## Option A — GitHub UI (easiest)

1. Log in as the owner of **vtf587-boop** (or an admin of this repo).
2. Open **Settings → General → Danger Zone → Transfer ownership**.
3. Transfer to user **JW-0042**.
4. Log in as **JW-0042** and **accept** the transfer if GitHub requires confirmation.
5. Update topics/homepage if needed; create a new release note pointing to the new URL.

## Option B — CLI as JW-0042

```bash
gh auth login   # as JW-0042
# From a machine that still has admin on vtf587-boop/md-sitemap:
gh api -X POST /repos/vtf587-boop/md-sitemap/transfer -f new_owner=JW-0042
```

## After transfer

1. Clone / re-point remote: `git remote set-url origin https://github.com/JW-0042/md-sitemap.git`
2. Optionally add a redirect README on any leftover fork.
3. Update production `llms.txt` lines that cite the spec URL.
4. Edit this file to note “transferred”.
