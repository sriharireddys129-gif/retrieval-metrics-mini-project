# Retrieval Metrics Mini Project

Build a tiny evaluation tool that measures how good a search system is.

In this mini project, we implement `eval.js` to evaluate two search systems: keyword search and semantic search. The program computes **Mean Reciprocal Rank (MRR)** and **Recall@10** for each system and prints a Markdown comparison table.

No database or API keys are needed because both systems' ranked results are already provided as data.

## What is in this project

- `eval.js`: JavaScript program that calculates retrieval metrics.
- `test-set.csv`: Contains 10 `(query, correct_post_ids)` pairs. A row may contain multiple correct answers separated by semicolons.
- `search-results.json`: Contains the top-10 ranked post IDs returned by keyword and semantic search for each query.
- `posts-dataset.csv`: Contains the 35 posts referenced by the IDs for sanity-checking.

## What was implemented

1. Loaded `test-set.csv` and `search-results.json`.
2. Implemented Reciprocal Rank to identify the position of the first correct answer.
3. Implemented the evaluation loop to calculate reciprocal ranks for both search systems.
4. Calculated MRR and Recall@10 for keyword and semantic search.
5. Generated a Markdown comparison table showing the ranking of correct answers for all 10 queries.

## Formulas

### Reciprocal Rank (RR)

RR = 1 / Rank of the first correct answer.

If the correct answer is not found in the top 10, RR = 0.

### Mean Reciprocal Rank (MRR)

MRR = Sum of all Reciprocal Ranks / Total number of queries.

### Recall@10

Recall@10 = Number of queries with correct answers in the top 10 / Total number of queries.

## How to run

Make sure Node.js is installed.

Run the following command in the project directory:

```bash
node eval.js
```

## Evaluation Results

The following table shows the comparison between keyword search and semantic search across all 10 test queries.

| Query | Keyword rank | Semantic rank |
|---|---:|---:|
| my redis cache keeps showing old data | 8 | 1 |
| how do I add semantic search to a table I already have | miss | 2 |
| redis or memcached for caching | 1 | 2 |
| why did my query get slower after I added an index | miss | 1 |
| what does ACID actually guarantee | 2 | 1 |
| how do I keep my postgres online while changing the schema | 6 | 3 |
| my background jobs stop after every deploy | miss | 2 |
| how do I let my database accept more writes per second | 4 | 1 |
| sql or nosql for a new product | 1 | 3 |
| why does my redis memory keep climbing | miss | 2 |
| **Summary** | **MRR 0.30 · Recall@10 0.60** | **MRR 0.67 · Recall@10 1.00** |

## Interpretation of Results

Semantic search performs better overall, achieving an MRR of 0.67 and Recall@10 of 1.00, compared with keyword search's MRR of 0.30 and Recall@10 of 0.60. Keyword search performs better on the queries "redis or memcached for caching" and "sql or nosql for a new product", where exact technical terms help retrieve relevant results. Semantic search performs better on queries involving symptoms, natural language descriptions, and paraphrased questions, such as Redis memory issues and database performance problems. This demonstrates that semantic search can retrieve relevant results based on the meaning and context of queries rather than relying primarily on exact word matches. Although keyword search has weaker overall results, a team would still keep it because it performs well on certain exact technical queries and can complement semantic search.

## Conclusion

This project demonstrates how MRR and Recall@10 can be used to evaluate and compare information retrieval systems. MRR measures ranking quality, while Recall@10 measures whether relevant results are retrieved within the top 10 positions.

## Take It Further (Optional)

If you have a working PostgreSQL database with posts loaded and embeddings generated, you can replace `search-results.json` with actual search results.

### Keyword Search (PostgreSQL Full-Text Search)

```sql
SELECT post_id
FROM posts
WHERE to_tsvector('english', title || ' ' || body)
      @@ plainto_tsquery('english', $1)
ORDER BY ts_rank(
    to_tsvector('english', title || ' ' || body),
    plainto_tsquery('english', $1)
) DESC
LIMIT 10;
```

### Semantic Search (pgvector)

```sql
SELECT post_id
FROM posts
ORDER BY embedding <=> $1
LIMIT 10;
```

Both search systems can be connected to the same evaluation logic to compare their retrieval performance using real search results.