# MATACS-TH-2026-02: AI Proposal Triage Design

- **Name:** Halima Sadia
- **Reference:** MATACS-TH-2026-02
- **Time actually spent:** About 2 hours

## Files

- [WRITEUP.md](WRITEUP.md): answers to the five questions
- [PROMPTS.md](PROMPTS.md): the extraction prompt
- [ranking.json](ranking.json): expected ranking of proposals A to E
- [diagram/](diagram/): process flow from upload to ranked output (AI steps, rule steps, human checkpoints)

## Design in one paragraph

AI only reads proposals and extracts facts with a proof quote. Plain code checks the quotes, applies the admin's pass/fail rules, calculates scores and ranks. A person reviews flagged items, can override, and makes the final award. Everything is logged.

## How I used AI tools

I used Claude to help plan the process flow, draft the write-up, the extraction prompt and the sample ranking. I then read the output and checked the scoring myself. Proof read it again , found glitch in the flow where it skipped the flow of code check in diagram. Prompted it to add up.

