# Query Sort Tool

A moderated tool for testing whether the doTERRA storefront's Search Query Taxonomy v2 can be applied consistently. Built for in-person research sessions: a moderator and one rater pass a single device back and forth while each rater, in turn and alone, sorts the same set of real search queries into the taxonomy's 14 buckets.

It is the validation step for the search work. The [Card Sort Tool](https://github.com/AntWQA/Card-Sorting-Tool) and [Tree Testing Tool](https://github.com/AntWQA/Tree-Testing-Tool) test the navigation; this one tests whether two people reading the same bucket definitions put the same queries in the same buckets. It tests the buckets, not the people: low agreement on a bucket means its definition is being read two ways, which is the thing to fix before the Algolia configuration, golden query set and evaluation gates are built on it.

## Hosted version

The live, working copy of this tool is a published Claude Artifact:

**[Open the Query Sort Tool](https://claude.ai/artifact/AgkSPq9fYxdvsWZDQ3Cvip)**

Private to the owner's account until shared from the page's share menu.

That is the version to run sessions with. It has a real shared database behind it, so:

- Every answer saves to a shared results ledger as the session runs, not just at the end.
- The ledger and the content (buckets and queries) sync live across every device that opens the link.
- Results export as CSV, JSON, or Markdown from the Ledger tab.

## This repo

`query-sorting-study.html` is the same tool as a single, self-contained HTML file: the source of truth for the code, and a way to run or inspect the tool outside of claude.ai. The published Artifact is this file with the page skeleton (doctype, `html`, `head`, `body`) removed and the `db` and `downloads` capabilities declared.

Opened directly it still runs the full moderator and rater flow, but without the Artifact's shared database:

- Results save to that browser's own local storage instead of a shared ledger. Not synced across devices, and cleared if browsing data is cleared.
- Export works the same either way, and is the reliable way to get a copy out.

### Running locally

No build step or dependencies, just one HTML file with inline CSS and JS.

```bash
open query-sorting-study.html
```

Or serve it, which some browser features need:

```bash
python3 -m http.server 8080
```

Then visit `http://localhost:8080/query-sorting-study.html`.

## How the tool works

1. **Setup**. The moderator names a rater, records how familiar they are with the doTERRA range (1 to 5), and optionally flags a practice run, which is kept in the ledger but excluded from the analysis. Names must be unique.
2. **Brief**. The rater reads a short brief: sort by what the query means, use only the bucket descriptions on screen, do not look anything up, do not discuss answers until everyone has finished.
3. **Sort**. One query at a time. The rater chooses a bucket by tapping it or pressing its letter, or "None of these fit" (0), which needs a reason. They can flag a term as unrecognised or themselves as unsure, add a note, and go back to change an answer. Queries in other languages show a literal English translation beneath them. The query order is shuffled per rater.
4. **Review**. A list of every answer. Tapping a row reopens that query, and "Back to review" returns to the list.
5. **Debrief**. The moderator records which buckets were hardest to tell apart, overall confidence (1 to 5), and notes. The rater's reasons are worth more than the percentages.
6. **Done**. Results save to the ledger and the moderator starts the next rater.

An interrupted session stays in the ledger as in-progress and can be resumed or discarded from the Setup screen. "End early" keeps the answers but excludes the sort from the analysis. There is no retake, because a second pass would not be blind.

## The test design

| Choice | Setting | Why |
|---|---|---|
| Raters | Two, sorting independently, one after the other | Agreement (Cohen's kappa) is only valid if neither sees the other's answers |
| Materials | Bucket names and definitions only | Tests whether the definitions stand alone. Examples, success criteria and precedence rules are withheld |
| Sample | 50 queries, stratified | Every bucket is represented. Not proportional to real volume |
| Order | Shuffled per rater | Order effects do not line up between raters |

The 50 queries come from the v2 classified-queries workbook: 38 from the head of the GA4 export, 5 from the zero-result list and 7 from the long tail. Buckets K, M and N have little or no head volume, so their items come from the zero-result list and the tail. They are there to test whether the definitions can be applied, not to say how common those queries are.

Language is a tag, not a bucket. Each of the 8 non-English queries is shown with a literal English translation, with no hint of the bucket. The translations were written by Claude and should be checked by a native speaker before the first rater. Edit them in the Content tab (`language` and `gloss` on each item).

### Who to use as raters

Two dōTERRA-side people who would apply the taxonomy day to day, from different functions: one in e-commerce or merchandising (familiarity 4 to 5), one from a neighbouring function such as support or content (familiarity 3 to 4). Avoid anyone who has seen the workbook or the draft classification, and customers, who do not apply a taxonomy. WQA colleagues suit a practice run rather than the main pair, because calls on buckets B, C and D depend on knowing the range.

## Reading the results

The Ledger tab shows, for any two completed sorts:

- **Cohen's kappa** with a bootstrap 95% interval, against a target of 0.61 (where "substantial" agreement starts). The verdict is deliberately cautious: it only says "Holds" if the low end of the interval meets the target.
- **Each rater against the draft**, and **queries where both raters agree but the draft differs**. If both raters land on the same bucket and the draft disagrees, the draft classification is the thing to revisit, not the definition.
- **By bucket**: specific agreement (how often both chose a bucket, out of all the times either did), with a "too few to say" read where counts are thin.
- **Where the buckets blur** and a **confusion matrix** show which pairs of buckets are being swapped.
- **Weighted by volume**: agreement across head queries weighted by search events, the closest read on real traffic.
- **Term familiarity**: agreement split by whether either rater flagged a term as unrecognised, to separate "does not know the product" from "reads the definition differently".

### What the numbers are worth

Two raters and 50 queries gives a wide interval and only a few queries per bucket, so treat everything as directional. Two raters cannot tell one person's quirk from a flawed definition; a third rater would, and the comparison picker accepts any two completed sorts. Kappa also falls when one bucket dominates a sample even if raters mostly agree.

Three further caveats worth holding on to:

- **The draft is one analyst's reading from query text alone.** "Agrees with the draft" is not the same as "correct".
- **Definitions are verbatim from the workbook**, and some carry internal vocabulary a rater has never seen: bucket A says "tagged brand_prefix", D says "tagged by language", K says "Not yet identified". Leave them (the test then includes that confusion) or reword them in the Content tab before the first rater. Never change them between raters.
- **Bucket N has almost no evidence.** Its two sampled queries ("Shinrin-Yoku", "Ingredients") are weak analyst calls from the long tail.

## Content

The Content tab edits the buckets and queries as JSON, validates before saving, and versions every save so each session records which version it ran against. Edit before the first rater. Changing it between the two raters breaks the comparison.

## What this does not cover

It tests whether the taxonomy can be applied consistently, not whether it is the right taxonomy, and not whether search results for any bucket are good. Those need the golden query set and evaluation gates this work is meant to feed.

## Data

The queries and event counts in the default content come from the GA4 baseline supplied by dōTERRA. That is why this repository is private. Do not make it public without checking that the data can be shared.

## Project context

This tool lives under the broader **Storefront** project folder, which covers other WQA/doTERRA storefront UX work.
