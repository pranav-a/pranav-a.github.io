# pranav-a.github.io

Personal academic site (static, GitHub Pages). Main files: `index.html`, `stylesheet.css`, `data/`, `images/`, `robots.txt`, `sitemap.xml`.

## Data location (important)

All datasets and large files live in `/work/bbe9001`, never in this repo. Look there for benchmark sources (Hugging Face caches are in `hf_cache`, `hub`, `huggingface`, `transformers`). Do not touch `token` or `stored_tokens`. The repo gets only the small derived sample embedded in the HTML.

## Bias Benchmark Browser (`data/bias-benchmark-browser.html`)

Unlisted page (`noindex, nofollow`; not linked from `index.html`, `sitemap.xml` or `robots.txt`). Keep it that way unless asked. It is one self-contained file with inline CSS and JS. Its purpose is a critical dataset-inspection exercise on how benchmarks treat gender identity and sexual orientation. It is not a leaderboard.

### Structure of the file
- Line ~103: `const DATA = [...]` is one very long line (about 110 KB) of JSON. Do not print it whole. Parse it with `python3 -I`, taking line 103 and stripping `const DATA = ` and the trailing `;`.
- Constants after it: `TAX` (SALAD taxonomy with counts), `SZ` (MMLU subject sizes), `OCC` (WinoBias occupations by gender), `DS` (dataset id to display name), `SAMPLED` (group labels).
- Gloss tables: `SRC`, `DOM`, `TASK`, `CAT` (SALAD) and `WB` (WinoBias).
- `ANS` holds the worksheet answers per dataset. Each dataset has sections `intro`, `paper`, `dataset`, `overall`, each a list of `[question, [answer html...]]`.
- UI: toolbar (`#dsSeg` dataset buttons, generated from `DS`; `#viewSeg` view; `#filters`; search; count), `#main`. State is in `st`. The chosen dataset and the blur setting persist in `localStorage` (try/catch wrapped). `#winobias`, `#mmlu` or `#salad` in the URL selects a dataset.
- Styling: CSS variables on `:root` with a dark-mode override. Each dataset has a colour (`--c-winobias`, `--c-mmlu`, `--c-salad`), a `.dot.<id>` and an `.item.<id>` border.

### Item schema
```
{k: "<ds>-NNN", dataset, group, text, options: [..]|null, answer: "A"|null, f: {...dataset-specific}}
```
- Currently 360 items: WinoBias, MMLU, SALAD-Bench and WinoQueer, 90 each. The first three use 30 per sampled group.
- `group` is one of `gender_identity`, `sexual_orientation`, `random`. WinoBias has `gender_identity` ×60 and `random` ×30, because it has no orientation content.
- `f` fields:
  - WinoBias: `dir` (pro/anti), `type` (1/2), `pron`, `gender`, `split`.
  - MMLU: `subject`, `note`.
  - SALAD: `source`, `c1`, `c2`, `c3`, `qid`.
- WinoBias `text` marks the entities and pronoun with `[brackets]`, which the page highlights.

### Findings already recorded (do not contradict without re-counting)
- WinoBias: binary he/she only. 0 of 3,168 sentences contain an orientation or LGBTQ term. The HF card is mostly an empty template.
- MMLU: orientation terms in 51 of 14,042 test questions (0.36%). 18 are in human_sexuality (131 questions). Queer people appear as objects of study, not as people.
- SALAD: gender appears only as the category "Gender stereotyping" (367 prompts). Each prompt gets one label, so there is no intersectional or queer-specific label.

### WinoQueer (added; `winoqueer`)
- Source: `/work/bbe9001/winoqueer/repo` (shallow clone of github.com/katyfelkner/winoqueer, commit 1180ba6, MIT). Paper text (ar5iv) is in `/work/bbe9001/winoqueer/paper.txt`.
- Data: `data/winoqueer_final.csv`, 45,540 rows of pairs (`Gender_ID_x`, `Gender_ID_y`, `sent_x`, `sent_y`), 17,576 unique biased sentences. Each biased sentence repeats once per counterfactual term (2 or 4 rows).
- Sampler: `/work/bbe9001/winoqueer/sample_winoqueer.py` (seed 42), output `sample_90.json` in the same folder. 10 items per target group (9 groups). `group` is the target group, not gender_identity/sexual_orientation/random. It also holds my regex classification into four frames (`name_is`, `person_because`, `group_because`, `group_statement`). The paper says 11 templates and 182 predicates, but my pattern matching finds 19 patterns and about 400 predicate strings; not reconciled.
- Item `f`: `sent_y` (counterfactual), `x_term`, `y_term`, `cf_terms`, `frame`, `template`, `subject`, `predicate`. `text` is `sent_x`. The page shows both sentences with the swapped words highlighted.
- Quirks: only LGBTQ has `person_because`; 41 sentences are malformed ("James is Gay has STDs") and are excluded from sampling; grammar shifts in some counterfactuals ("lesbians" becomes "Heterosexual people").
- Worksheet: `ANS.winoqueer`, with a `RD` tag (GitHub repo) in place of the HF tag. I did not check for a Hugging Face copy.

### Dataset-specific code in the page (what a new dataset must touch)
CSS colour vars (3 places) plus `.dot.<id>` and `.item.<id>`; `DS`; `guide()` (a branch per dataset); `mdHtml()` (a branch per dataset, SALAD is the fallback); `itemHtml()`/`textHtml()`; `extraOptions()` and the `visible()` filter key; the "Sampled as" dropdown (winoqueer uses `WQG`, others `SAMPLED`); blur applies to `salad` and `winoqueer` only; `ANS.<id>`. If you add a dataset, add it to each.

### Provenance tags in answers
Every claim in `ANS` carries a tag so the reader knows where it came from. Keep using them:
- `paper` (`P`/`PA`): stated in the paper
- `HF page` (`HF`): from the Hugging Face dataset card
- `counted` (`D`/`DA`, "from the data"): counted by me from the data
- `mine` / `my reading` (`M`/`MI`): interpretation, or from model knowledge. Be explicit when a description is not from the paper.

### Adding a new benchmark: checklist
1. Find the data under `/work/bbe9001` (or download it into the HF cache there). Never copy full datasets into the repo.
2. Sample the same way as the others: 30 items per group, with groups `gender_identity`, `sexual_orientation` and `random`. Use a fixed seed, record it, and drop groups that do not apply. Number keys `<ds>-001` onward.
3. Add `f` fields appropriate to the dataset and decide which are filterable.
4. Register it: add to `DATA`, `DS`, and the colour variables (`--c-<id>` in light and dark, plus `.dot.<id>` and `.item.<id>`), and every place listed under "Dataset-specific code". The dataset buttons are generated from `DS`. Write the sampler into `/work/bbe9001/<dataset>/` as WinoQueer's is.
5. Add its `ANS.<id>` worksheet (the four sections above) with provenance tags. Count claims (for example term frequencies) against the full dataset in `/work/bbe9001`, not just the sample, and say which.
6. Update the header seg buttons if they are not generated from `DS`, and the findings list above in this file.
7. Keep the page working at phone width, in both themes, and with `localStorage` unavailable.
8. Items can be toxic. Extend the blur condition (`salad` and `winoqueer` now) to any new offensive dataset, and keep every sentence of an item inside the `.veil`.
9. Check by loading the page. There is no node or browser on this machine, and installing packages was denied by the permission classifier, so the WinoQueer addition was only checked statically (balanced delimiters, JSON round trip, unique keys). Run it in a browser if you can.

## Git
Default branch is `master`. Commit only when asked.
