---
name: bib-verify
description: "Verify BibTeX entries against CrossRef API. Checks DOIs, titles, authors, journals, years for accuracy. Detects fabricated or incorrect entries. Use when auditing .bib files for a thesis or paper. Chinese triggers: 参考文献, bib, 引用检查, DOI, 论文引用, 文献校验."
---

## Instructions

Verify BibTeX references for: $ARGUMENTS

If no file path is given, ask the user for the .bib file path.

---

## Step 1: Extract entries from the bib file

Read the .bib file and extract all entries. For each entry, collect:
- Citation key
- DOI (if present)
- Title
- Authors
- Journal/Booktitle
- Year
- Volume, pages

## Step 2: Verify entries with DOI

For entries that have a DOI, verify via CrossRef API:

```bash
curl -s "https://api.crossref.org/works/{DOI}" --max-time 8 | python3 -c "
import sys,json
d=json.load(sys.stdin)['message']
print('Title:', d['title'][0][:100])
print('Authors:', ', '.join([a.get('family','') for a in d.get('author',[])][:5]))
print('Year:', d.get('published',{}).get('date-parts',[['']])[0][0])
print('Journal:', d.get('container-title',[''])[0][:60])
print('Volume:', d.get('volume',''))
print('Pages:', d.get('page',''))
"
```

Compare the returned metadata with the bib entry. Flag mismatches in:
- Title (significant difference, not just capitalization)
- Authors (different people, not just name format)
- Journal name (completely different journal)
- Year (off by more than 1 year)
- Volume or pages (significant difference)

## Step 3: Verify entries without DOI

For entries without DOI, search CrossRef by title:

```bash
curl -s "https://api.crossref.org/works?query={URL_ENCODED_TITLE}&rows=1" --max-time 8
```

If a match is found, report the real DOI so it can be added.
If no match is found, flag the entry as unverifiable (may need manual check).

## Step 4: Batch processing

Process DOIs in batches of 5-8 to avoid rate limiting. Use parallel curl calls where possible.

For large bib files (>50 entries), first extract all DOIs:
```bash
grep -oP 'doi\s*=\s*\{([^}]+)\}' refs.bib | sed 's/doi\s*=\s*{//;s/}//'
```

## Step 5: Report

Output a summary table:

| Key | Status | Issue |
|-----|--------|-------|
| key1 | OK | - |
| key2 | MISMATCH | Title differs: bib says "X", CrossRef says "Y" |
| key3 | DOI_NOT_FOUND | DOI does not exist in CrossRef |
| key4 | NO_DOI | No DOI provided, searched by title, found DOI: 10.xxx |
| key5 | FABRICATED | Journal/authors completely wrong |

Severity levels:
- **FABRICATED**: Journal, authors, or title entirely wrong (GPT hallucination)
- **MISMATCH**: Some fields incorrect but paper exists
- **DOI_NOT_FOUND**: DOI returns 404/no result
- **NO_DOI**: Entry has no DOI (suggest adding one)
- **OK**: All verified fields match

## Notes

- CrossRef API is free and doesn't require authentication
- Rate limit: ~50 requests/second for polite pool (add `mailto` param for higher limits)
- IEEE DOIs often return HTTP 418 (anti-bot) on doi.org but work fine on CrossRef API
- Books (@book) may not be in CrossRef; flag as "unverifiable" rather than "fabricated"
- Conference papers (@inproceedings) may have limited metadata in CrossRef
