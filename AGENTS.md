# AGENTS.md

## Project purpose

- This repository is a Chinese-language cryptocurrency learning project, not a trading bot or investment product.
- Preserve `QA.txt` and `crypto_beginner_learning_roadmap.png` as migration archives. Put reviewed learning material in `docs/`.
- Keep explanations accessible to a beginner, but do not trade away technical accuracy for slogans.

## Content rules

- Prefer primary sources: protocol documentation and specifications, original whitepapers, issuer disclosures, and regulator publications.
- For facts that can change (protocol parameters, token supply, reserve composition, regulation, product availability, or fees), verify them at task time and record the verification date.
- Clearly label issuer claims as claims. Distinguish protocol facts, observations, and inference.
- Do not provide personalized investment recommendations, return promises, price targets, or instructions to evade regulation.
- Default learning exercises to read-only explorers, local simulations, or testnets. Never require real-money transactions to complete a lesson.

## Security rules

- Never request, print, store, or commit seed phrases, private keys, wallet backup files, exchange credentials, API secrets, identity documents, or unredacted account screenshots.
- Use obvious placeholders such as `YOUR_API_KEY`; never invent realistic secrets.
- Do not ask the learner to connect or sign with a wallet before the wallet-safety stage is complete.
- If a task could move funds, create an approval, sign a transaction, or change an external account, stop and obtain explicit user authorization after explaining the exact action and risk.

## Repository workflow

- Write project documentation in Simplified Chinese; English technical terms may follow in parentheses on first use.
- Update `docs/roadmap.md` and `docs/learning-log.md` when a learning milestone is actually completed. Do not infer completion from file creation.
- Keep Markdown links relative for repository files and use stable canonical URLs for external sources.
- Make focused changes, preserve user-authored material, and do not commit unless the user asks.
- Before handing off documentation changes, check `git diff --check`, inspect links and headings, and report anything that could not be verified automatically.

## Definition of done for a learning stage

- The stage has explicit learning goals, a lesson sequence, safe exercises, primary sources, common misconceptions, an assessment, and exit criteria.
- The learner has produced the required outputs and passed the assessment; merely reading the material is not enough.
