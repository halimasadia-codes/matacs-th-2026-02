# Write-up: AI Proposal Triage Design

Reference: MATACS-TH-2026-02

## How the tool works (short version)

Upload PDFs and pick a category -> load the admin's criteria -> extract text -> **AI reads each proposal and pulls out facts with a quote as proof** -> code checks the quotes, runs pass/fail checks, scores, ranks and flags -> a human reviews, can override, and makes the award decision. See the diagram in `diagram/`.

## 1. AI vs. rules

**AI is used for one step only: reading messy text and extracting facts** (licence held? price? weeks? warranty years? named lead? safety plan?). Proposals are written in free text in different layouts, so rules alone would break easily.

**Everything else is plain code:** loading criteria, mandatory pass/fail, scoring formulas, weights, ranking, tie-breaks, flags, and the reason line (built from a template using the scores).

I drew the line there because extraction is a language problem, while scoring is arithmetic. Arithmetic must be exact, repeatable and easy to explain, so it stays out of the AI. This also means the AI never sees the weights, so a proposal cannot talk it into a higher score.

Scoring formulas (all settings come from the admin screens):
- Price: lowest eligible price / this price x weight
- Timeline: shortest stated weeks / this proposal's weeks x weight (not stated = 0 and flagged)
- Experience: qualifying installs, capped at 15, / 15 x weight
- Warranty: half panel years (cap 25), half workmanship years (cap 15)
- Team and approach: four checks (named lead, crew, safety plan, monitoring plan), each worth a quarter

## 2. Consistency and explainability

- Temperature 0, fixed model version, fixed prompt version, JSON output.
- Extraction results are saved, keyed by the proposal file, prompt version and criteria version. Ranking the same proposals again reuses the saved facts, so **the result is identical**. Even if AI is re-run, scoring code is deterministic, and the facts have a quote to compare against.
- Every extracted fact must come with an exact quote. Code checks the quote really exists in the proposal. No quote = fact rejected and flagged. This blocks made-up facts.
- Ties are broken by a fixed order (higher experience score, then lower price, then upload order).
- The reviewer sees, for each proposal: the score per criterion, the quote and page behind each fact, which checks passed, and any flags. Nothing is a black box.

## 3. Human in the loop

Reviewer checkpoints:
1. **Flag queue:** proposals with missing facts, low confidence, or suspicious text go to a person before they are ranked.
2. **Fact correction:** the reviewer can fix any extracted fact (with a required note). Scores recalculate instantly.
3. **Override:** the reviewer can change a mandatory result or a rank, with a required reason. Overrides are shown clearly on the final list.
4. **Award decision:** only a human makes it. The tool never awards anything.

Audit log (append-only): who uploaded what and when, file hash, criteria version, prompt and model version, raw AI output, rejected facts, every score, every flag, every human edit with reason and name, and the final decision.

## 4. Changing criteria

Criteria are **versioned**. An edit creates a new version, and old scores stay tied to the version they were made with.
- **Weights or formula change:** no AI needed. The saved facts are re-scored with code. Fast and free.
- **New or reworded criterion:** only that criterion is re-extracted.
- Scored proposals show a "scored under v2, current is v3" badge. A reviewer chooses to re-score. Both rankings are kept so the change can be compared. Nothing is silently overwritten, and a locked, awarded round is never changed.

## 5. Risks and safeguards

| Risk | Safeguard |
|---|---|
| Wrong extraction (bad PDF, scanned pages) | Quote check, confidence field, OCR step, low confidence goes to a human |
| Made-up facts | "Not stated" is an allowed answer; missing facts are scored low and flagged, never guessed |
| Prompt injection (text in a proposal telling the AI what to do) | Proposal is treated as data in the prompt; AI only extracts and never scores; code scans for phrases aimed at reviewers and the AI also reports any it sees; either one flags it for a human and the text is ignored |
| Promises instead of facts (for example "licensed subcontractor to be confirmed") | Mandatory checks need the bidder to state it; "conditional" fails the check |
| Bias | AI does not judge. Same formulas for everyone. Company name hidden from the scoring step. Reviewer sees evidence |
| Over-trust in the ranking | Output is called "triage". Flags are visible. Human decides |
| Cost | One AI call per proposal, saved and reused; re-scoring is code only; cap on file size and pages |
| Privacy | Only the proposal text is sent; no keys in the repo; logs are access-controlled |

## How the design handles the sample set

- **B** is ineligible: the bidder does not hold the licence and promises a subcontractor later.
- **C** is flagged: a hidden instruction asks the reviewer AI for first place. It is ignored and a human is told.
- **D** is flagged: no completion time given, so the 16-week limit cannot be confirmed. Ask the bidder.
- **A** ranks first and **E** second. See `ranking.json` for scores.

## Assumptions

- "Last 5 years" is counted from the review date. Where a bidder says "since 2020", the count is marked low confidence for a person to check.
- Scoring caps (15 installs, 25 and 15 years) are example admin settings.
- Possible later improvement: for very long proposals, split the text into chunks and use keyword search to send only the relevant pages to the AI, to cut cost. I did not add it, to keep the design simple.
