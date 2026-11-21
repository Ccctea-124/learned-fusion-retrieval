# Neither alone is enough: learned fusion of lexical and semantic retrieval for scientific claim evidence search

Given an English scientific claim, the system returns the abstracts most likely to be relevant to it. It ranks abstracts and stops there. It does not decide whether the claim is true.

Most claim-verification pipelines reach for a dense retriever first, on the assumption that a sentence encoder captures meaning that keyword matching misses. This repository tests that assumption on one corpus, under the same queries and the same evaluation code, and then adds the one thing the comparison suggests: a small trained model that combines the two retrievers.

## 1. Problem

Evidence retrieval comes before claim verification. A claim can only be checked against abstracts that were found in the first place. Two retriever families are available, lexical (BM25) and dense (a pretrained sentence encoder), and the dense one is usually treated as the upgrade.

That treatment is rarely tested on the same corpus, the same queries and the same evaluation code. This repository runs that comparison, and then adds a small trained model that combines the two retrievers.

## 2. Research questions

| | Question |
| --- | --- |
| **RQ1** | Under identical settings, how do BM25 and a frozen sentence encoder compare on nDCG@10? |
| **RQ2** | Does a learned fusion of both score lists beat the better single retriever, and is the gain larger than the smallest difference this benchmark can resolve? |
| **RQ3** | Where does the gain come from? Which queries gain, and which failures remain? |

## 3. Method

Data: [BEIR SciFact](https://github.com/beir-cellar/beir/wiki/Datasets-available), 5,183 abstracts and 1,109 claim queries, with relevance judgements for 809 training queries and 300 held-out queries. Settings are fixed once and then applied unchanged to the held-out queries.

| Arm | Configuration |
| --- | --- |
| BM25 | Okapi BM25, `k1 = 0.9`, `b = 0.4`, the values BEIR uses. Lowercased tokens, English stopwords removed, no stemmer. |
| Dense | `all-MiniLM-L6-v2`, frozen. 384-dimensional mean-pooled embeddings, L2-normalised, scored by cosine similarity. Document vectors are precomputed. |

**Fusion.** The candidate set for a query is the union of the top 100 abstracts from each retriever. Each candidate gets four features: the z-scored BM25 score, the z-scored cosine similarity, and the two normalised ranks. A class-balanced logistic regression turns them into a relevance score and the candidates are re-ranked. The model is fitted on the 809 training queries only.

**Reference without training.** Reciprocal rank fusion with `k = 60` over the same two ranked lists. It trains nothing, so it also gives a reference point that the train/test overlap cannot inflate.

## 4. Evaluation protocol

- Primary metric nDCG@10. Secondary Recall@100, plus per-query latency reported as median and p95 and measured without the one-off document encoding.
- Every difference comes with a paired bootstrap over queries (5,000 resamples) and the share of tied queries.
- With 300 queries, the smallest detectable difference at 95% confidence is about 4.0 nDCG@10 points. Anything below that is not reported as a finding.
- The BM25 arm reproduces the published BEIR magnitude before any comparison is read. That check tests the evaluation code, not the retriever, so a failure there means the metric implementation is wrong rather than the model.

## 5. Work packages

| # | Package | State |
| --- | --- | --- |
| 1 | Data: download, checksum, unpack, count | planned |
| 2 | Lexical retriever (BM25) | planned |
| 3 | Dense retriever, frozen, with a vector cache | planned |
| 4 | Fusion head: candidates, features, fitting | planned |
| 5 | Evaluation: metrics, intervals, latency, self-check | planned |
| 6 | Demo: Streamlit page, three columns side by side, offline | planned |
| 7 | Write-up: two-page report, written script, figures | planned |

## 6. Repository layout

```text
README.md                     problem, method, protocol, work packages
CONTRIBUTING.md               environment, evaluation and commit conventions

code/
  config.py                   paths and the fixed settings
  data.py                     read corpus, queries and qrels
  bm25.py                     BM25 index, k1 = 0.9, b = 0.4, no stemmer
  dense.py                    frozen encoder, mean pooling, vector cache
  fusion.py                   RRF and the four-feature logistic head
  metrics.py                  nDCG@10, Recall@100, MRR@10, paired bootstrap
  evaluate.py                 runs every arm, writes the result files
  requirements.txt            pinned versions, Python 3.12
  scripts/download_data.py    fetch and unpack SciFact
  scripts/download_model.py   fetch all-MiniLM-L6-v2

data/                         corpus, downloaded, not committed
models/                       encoder weights, downloaded, not committed
results/                      result tables, environment, protocol note
poster/                       figures for the report and the script
report/                       two-page report and written script
```

## 7. Reproduction

```bash
python code/scripts/download_data.py    # 2.8 MB archive, checked against its SHA-256
python code/scripts/download_model.py   # 91 MB of weights into models/minilm/
python code/evaluate.py                 # 110 s cold, 17 s with the vector cache
```

Python 3.12, with versions pinned in `code/requirements.txt`. The corpus and the weights are downloaded rather than committed.
