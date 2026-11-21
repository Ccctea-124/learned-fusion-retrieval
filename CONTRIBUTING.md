# Working conventions

## Scope

Compare BM25, frozen MiniLM embeddings and a four-feature logistic-regression fusion head for finding SciFact abstracts related to an English scientific claim. Untrained RRF is the reference. The output is a ranked list of abstracts, not a truth verdict.

Keep it small: data loading, two retrievers, the fusion head, one evaluator, one Streamlit demo. No stance classification, no sentence-level evidence selection, no language-model fine-tuning.

## Environment

- Python 3.12. Install from `code/requirements.txt`; `python` on PATH may point to a different version with nothing installed.
- Install from `code/requirements.txt`.
- Dataset and encoder weights stay local and are never committed.
- Every command quoted in `README.md` must point at a file that exists, and must have been run before it is written down.
- A six-sentence MiniLM encode was verified on 2026-10-29 against weights downloaded by hand. The `huggingface_hub` downloader failed here, so the model is loaded from a local directory with `TRANSFORMERS_OFFLINE=1`.
- `transformers` 5.x needs `torch>=2.6`; `torch 2.14.1+cpu` is what was checked. Streamlit 1.37.1 needs `rich<14`.

## Evaluation

- Fit the fusion head on the 809 training queries. Use train-only cross-validation when checking stability.
- The 300 held-out queries were used for an early TF-IDF smoke test and the fusion design was fixed after seeing them. Do not call that split untouched or blind.
- Report nDCG@10, Recall@100, paired bootstrap intervals, query latency, candidate recall, and a few concrete failure cases. Check the document overlap between the two splits.
- Keep every script and setting needed to rerun a reported number.

## Git

- One commit per stage: data, baseline, neural retrieval, fusion and evaluation, demo, write-up.
- Read the staged file list before committing. `readme.txt`, credentials, `data/` and model weights never go in.
- Commit under your own name and email.
- No rewriting authorship and no backdated commits.
