<div align="center">
    <img src="https://contest.faio.kz/problems/01a173fe-4a2e-7747-a8a8-98a7edf0c9ca/public/image.png" alt="hid-dim-v3" border="0" width="500">
</div>

## Background

Darkhan and Adilkhan are two young data scientists who just joined a big tech company.
Their first assignment is a big one: build a search engine, like Google, Yahoo or Bing,
but only for sports. Given a query such as "Kylian Mbappe", "San Antonio Spurs" or
"Serena Williams", it should return the web pages that are about that athlete or team,
with the most relevant pages first.

Crawling the whole internet is far too much for a first attempt, so they start small, with
a sample of about 100,000 web pages, and build the ranker on this "mini-internet" first.
Your job is to help these two young specialists pull it off.

You are given this fixed collection of pages and a list of query entities. For each query
you return a ranked list of page IDs, from the most relevant to the least.

## Your task

For every query entity in `queries_test.csv`, output an ordered list of `url_id`s from the
corpus that are about that entity, most relevant first.

## Data files

All files are available as a single archive: [Download data.zip (136.1 MB)](https://contest.faio.kz/problems/01a173fe-4a2e-7747-a8a8-98a7edf0c9ca/public/data.zip)

```bash
wget https://contest.faio.kz/problems/01a173fe-4a2e-7747-a8a8-98a7edf0c9ca/public/data.zip
```

| File | Columns | Description |
|---|---|---|
| `corpus.csv.gz` | `url_id, url, title, description, body` | The pages (the "mini-internet"), about 100,000 rows. Contains sports pages and unrelated pages. |
| `queries_test.csv` | `query_id, query_name, entity_type` | The entities you rank pages for. `entity_type` is `athlete` or `team`. |
| `queries_train.csv` | `query_id, query_name, entity_type` | Example entities, with answers, to develop and test your approach. |
| `train_gold.csv` | `query_id, url_ids` | For each training entity, its relevant pages in order, best first. |
| `sample_submission.csv` | `query_id, url_ids` | A valid submission in the required format. |
| `score.py` | | The scoring code (the same metric as the server). Run it locally to check yourself. |

`url_id`s are opaque identifiers, and the same page can be relevant to more than one
entity. In `train_gold.csv` the pages are sorted by relevance, most relevant first; pages
next to each other may be equally relevant.

## Submission format

A CSV with a header and one row per test query:

```
query_id,url_ids
q00001,u0004521 u0000093 u0012034 ...
q00002,u0008810 u0000142 ...
```

- `url_ids` is a space-separated list of `url_id`s, ordered from most relevant to least.
- Do not return completely irrelevant pages.
- Include every `query_id` from `queries_test.csv`. A missing or empty query scores 0.

## Scoring

The metric for this task is NDCG (Normalized Discounted Cumulative Gain). Your final score
is the average NDCG over all test queries, computed over your full ranking (no cutoff).

Each page has a relevance grade $g$ (an integer, with $g = 0$ if the page is not relevant).
A page's gain and the discounted cumulative gain of your ranking are

$$\mathrm{gain}(g) = 2^{g} - 1, \qquad \mathrm{DCG} = \sum_{r \ge 1} \frac{\mathrm{gain}(g_r)}{r},$$

where $g_r$ is the grade of the page you placed at rank $r$ (ranks start at 1). Dividing by
the best possible ordering gives a value between 0 and 1:

$$\mathrm{NDCG} = \frac{\mathrm{DCG}}{\mathrm{IDCG}}, \qquad \mathrm{IDCG} = \mathrm{DCG}\ \text{of the ideal ordering}.$$

The final score is the mean NDCG over all test queries, on a 0 to 100 scale ($100 \times$
mean NDCG, so 100 is a perfect ordering). The shipped `score.py` computes exactly this.

### Example

Suppose one query has three relevant pages $u_1, u_2, u_3$ with grades 3, 2 and 1, so their
gains are $2^3 - 1 = 7$, $2^2 - 1 = 3$ and $2^1 - 1 = 1$. You submit them in the order
$u_2, u_1, u_3$. Then

$$\mathrm{DCG} = \frac{3}{1} + \frac{7}{2} + \frac{1}{3} \approx 6.833,$$

and the ideal ordering $u_1, u_2, u_3$ gives

$$\mathrm{IDCG} = \frac{7}{1} + \frac{3}{2} + \frac{1}{3} \approx 8.833,$$

so $\mathrm{NDCG} = 6.833 / 8.833 \approx 0.774$. If this were the only query, your score
would be $100 \times 0.774 = 77.4$.

## Leaderboard

- **Public and private split.** During the contest you see your score on a public 35% of
  the test queries. Your final result is computed on the remaining 65% (the private part),
  which you do not see a score for, and is announced after the contest ends. Do not overfit
  the public leaderboard.

**Submission limit.** For each problem, you can submit at most once per minute.