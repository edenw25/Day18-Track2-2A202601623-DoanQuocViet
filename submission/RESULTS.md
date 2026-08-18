# Day 18 Lakehouse Lab — measured results

Every number below was produced on this machine by the executed notebooks in
[`notebooks/`](../notebooks/); the raw output cells are preserved in the
`.ipynb` files. Environment: **Windows 11 · Python 3.11.8 · deltalake 1.6.2 ·
pyiceberg 0.11.1 · duckdb 1.5.5 · polars 1.43.2 · pyarrow 25.0.1**.

Gates: `smoke` 9/9 · `pytest` 24/24 · `run-all` 8/8 (42.4 s).

---

## Part A — Foundations

### NB1 — Delta basics

| Criterion | Measured |
|---|---|
| `_delta_log/` JSON commits | v0 `WRITE`, 1 file added, 3 rows |
| Bad-schema write blocked | `Cast error: Cannot cast string 'thirty' to value of Int64 type` |
| `schema_mode="merge"` adds `tier` | 4 rows, `tier` = `premium` on the new row, `null` on the three pre-existing |

The reading: enforcement is the **default** and evolution is **opt-in**. The
three older rows were backfilled with `null` by metadata alone — no data file
was rewritten. That asymmetry is the whole design: an accidental type change is
an error, a deliberate one is a keyword.

### NB2 — OPTIMIZE + Z-ORDER

| Criterion | Measured |
|---|---|
| Small-file problem reproduced | **200** files before OPTIMIZE |
| `numFiles` after | **55** (3.6× fewer) |
| Speedup | **5.8×** (143.6 ms → 24.5 ms median, n=3) |
| Files-pruned ratio | **55.0×** — 1 of 55 files covers `user_id=4242` |

Both alternatives of the criterion passed. The one to trust is **55×**, not
5.8×: wall-clock on a laptop measures page cache as much as layout, whereas
file-pruning is read straight out of the commit's `add.stats` min/max ranges
and is deterministic. Z-ORDER did not make the query faster by being clever —
it made the min/max ranges **non-overlapping**, which is what turns a statistic
into a skip. The per-file ranges printed in the notebook (`[3696, 5534] ←
contains target`) are that proof.

### NB3 — Time travel, MERGE, RESTORE

| Criterion | Measured |
|---|---|
| `history()` versions | **5**, and v4 is the `RESTORE` row |
| MERGE 100K | 0.08 s — 50,000 updated + 50,000 inserted, 150,000 output rows |
| RESTORE rolls back | `score < 0` count = **0** after restore (0.02 s) |

The reading: RESTORE took **0.02 s on 150K rows** because it copies no data. It
appends a new commit whose action list points back at the v2 files, which are
still on disk. That is also the warning — "rollback" is instant only while the
old files exist, i.e. only inside your VACUUM retention window. NB6 Job 3 is
where that window gets spent.

### NB4 — Medallion

| Criterion | Measured |
|---|---|
| Bronze / Silver / Gold on disk | all three under `_lakehouse/` |
| Silver < Bronze | 200,000 → **190,052** (9,948 duplicates dropped) |
| Gold ≥ 7 dates × 3 models | **8 dates × 3 models = 24 rows** |

The reading: the 9,948 dropped rows are ~5% of ingest. Bronze deliberately kept
them — Bronze's contract is *faithful to the source*, including its mistakes.
Dedup belongs in Silver so the raw record stays re-processable when the dedup
key turns out to be wrong. Gold shows the cost split the layer exists for:
`claude-opus-4-7` serves ~1/6 the tokens of `claude-sonnet-4-6` at nearly the
same daily spend (~$291 vs ~$345), which is a routing decision no dashboard
over Bronze could surface.

---

## Part B — Lakehouse 2026

### NB5 — Iceberg & the catalog as control plane

| Criterion | Measured |
|---|---|
| Created through the catalog, spec `day(ts)` | `1000: ts_day: day(2)` |
| Hidden-partition pruning | **10×** — 10 files → **1** filtering on `ts` |
| Metadata:data ratio | data 47.3 KB / metadata 134.3 KB → **284%** |
| Rename keeps `field_id` | `latency_ms` → `latency_millis`, both `field_id=4` |
| ≥ 2 partition specs coexist | `spec_id [1, 2]`, 5,500 rows readable across both |

The reading — **10× because the filter was on `ts`, never on `ts_day`.**
`ts_day` is not a column anyone inserts; Iceberg derived it from the stored
transform and applied it during planning. A Hive user who wrote the same query
without remembering the partition predicate reads all 10 files. At 512 MB/file
and $5/TB that is 4.5 GB wasted per query — **$220/day at 10K queries**. Hidden
partitioning does not make pruning better; it removes the opportunity to forget.

The 284% metadata ratio is an artefact of 10-row files and should be read as a
*second* penalty of small files: you pay in data GETs **and** in metadata to
plan over. At production file sizes the same tree is ~0.1%.

The rename is the part Hive genuinely cannot do: identity lives in the
**field-ID**, not the name or the position, so renaming rewrote zero bytes and
old Parquet files still resolve.

### NB6 — The four mandatory jobs

| Job | Measured |
|---|---|
| 1 — Compaction | 200 → **11** files (**18×**), avg file was 51.5 KB vs 128–512 MB target |
| 2 — Clustering | point query opens **1 of 10** files → **90% skipped** |
| 3 — Expiry (Delta) | vacuum reclaimed **16.1 MB**, 211 tombstoned files |
| 3 — Expiry (Iceberg) | snapshots **20 → 3**, avro **40 → 40**, metadata **grew** 335.2 → 342.8 KB |
| 4 — Orphans (Delta) | **3** planted orphans (21.2 KB) found + removed; 5 files were invisible to the table |
| 4 — Orphans (Iceberg) | **17** stranded manifest lists (37.0 KB) swept, avro 40 → 23 |
| 5 — Checkpoint | `...0099.checkpoint.parquet` + `_last_checkpoint` written |

Three readings worth the marks:

1. **Compaction cost bytes before it saved them.** Data went 10.1 MB → 16.1 MB
   *up* immediately after OPTIMIZE, because the new files are written before
   the old ones are reclaimed. You are briefly billed for both copies. A
   compaction job scheduled without headroom for that overlap is how
   maintenance itself causes the incident.

2. **`VACUUM` did not find the orphans** (a contradiction of the common
   belief). After vacuum: 15 parquet files on disk, **10** in the log — 5 paid
   for and invisible. `deltalake` reclaims files the log has *tombstoned*; a
   file a crashed writer left behind was never committed, so it was never
   tombstoned, and no retention setting will ever surface it. The fix in the
   notebook is a set difference between the directory listing and
   `file_uris()` — plus an age guard, because deleting a file a concurrent
   writer has written but not yet committed corrupts the table.

3. **`expire_snapshots` deleted zero files** (the second contradiction).
   20 → 3 snapshots and the metadata directory got *larger*, because expiry
   rewrites `metadata.json` and only unlinks snapshot **references**. The 17
   orphaned manifest lists were still on disk and still billed. Job 3 and Job 4
   are a **pair** — running expiry alone is the precise reason teams say "we
   expire snapshots but the S3 bill never moves".

At the FinOps end: managed compaction on 500 GB / 2M files costs $990/mo, of
which **$240 (24%) is the per-object component** — driven by file *count*, not
volume. Fixing the writer's trigger interval is cheaper than paying to clean up
after it.

### NB7 — Multimodal & vectors

| Criterion | Measured |
|---|---|
| Random-access amplification | **200×** — 12.5 MB row group read to fetch one 64 KB frame |
| Analytical scan cost | 1.2 KB read either layout — column pruning already protects it |
| int8 on disk | 2.6 MB → **451.9 KB** = **5.8× smaller** (83% saved) |
| recall@10 / topic fidelity | **0.904** / **1.000** |
| Semantic search as SQL | top-5 all on-topic |
| Lifecycle bug | **0** hits in-table, **8** hits in the stale index |

The reading: the common advice "never inline blobs" is **wrong for analytical
scans** — `SELECT topic, count(*)` touched 1.2 KB in both layouts, because
projection pushdown never opens the blob column. It is right for **random
single-row reads**, where Parquet's granularity is the row group: one 12.5 MB
group to serve one 64 KB frame. The trigger is not blob size, it is the access
pattern, and at 1,000 random fetches/sec that 200× *is* the GPU-starvation
problem.

On quantization, **exact-ID recall understates the trade**: 0.904 recall but
**1.000** topic fidelity means the ~10% of "misses" are swaps between
near-equivalent neighbours, which for RAG is not a miss at all. Reporting only
recall@10 would have rejected a 5.8× storage win for free.

Brute force scaled as advertised — 11.2 ms at 2K vectors, ~5.6 s extrapolated
at 1M — so the lakehouse is the **system-of-record** and the vector DB is a
**rebuildable derived index**, not the other way around. The lifecycle result
is what happens when a team gets that backwards: the erased subject's 8
documents were gone from the table and still served by the index, and with a
one-way upsert sync, *permanently*. CDF emitted exactly 8 delete events — the
index has to **subscribe to deletes**, not poll for inserts.

Also confirmed: Delta has no fixed-width vector type. `fixed_size_list<float>[256]`
came back as `list<element: float>` and needed a cast at query time — which is
why Hudi 1.2 added a first-class `VECTOR(dim, type)` column.

### NB8 — Agents & provenance

| Criterion | Measured |
|---|---|
| Trajectories through medallion | 1,578 steps; Silver partitions `agent_version=policy-v2 / -v3` |
| Gold covers both policies | v2 success 0.760 vs v3 0.753, cost $10.37 vs $10.39 |
| Version pin replays exactly | pinned v0 = 1,578 steps after v1 grew to 1,978 — **exact match** |
| MCP `tools/list` cacheable | 5 agent turns → **1** catalog round-trip (`ttlMs: 60000`) |
| Destructive call gated | `resultType: input_required` before `delete_rows`, `ok` only after approval |
| Task poll completes | `working → working → completed`, `{'rows': 300}` |
| 4 Art. 10 buckets as partitions | licensed / public_domain / synthetic / scraped_optout_checked, **+ UNCLASSIFIED 334** |
| UNCLASSIFIED excluded | trainable set **1,666 / 2,000** |

The reading: v2 and v3 differ by 0.7 points of success rate on 150 trajectories
each — that is **noise, not a win**, and the value of the Gold table is that it
says so instead of letting a screenshot of one good run decide the rollout.

The pinned `table_version: 0` is the entire difference between a reproducible
training run and a story about one. Rollouts kept landing (1,578 → 1,978
steps); replay at v0 still returned exactly what training saw.

On the MCP surface, the gate that matters is that `input_required` is enforced
by the **protocol**, not by the model's judgement — an agent cannot
self-approve its own destructive call. And the `submit → poll → completed`
shape is the same one Iceberg 1.11 server-side planning uses; two protocols
converging on one shape is not a coincidence.

Governance: **334 rows (16.7%) were UNCLASSIFIED** and excluded, leaving 1,666
defensible training rows. Under Art. 10 an unlabelled bucket mixing scraped and
licensed data is not a documentation gap, it is an audit failure. The honest
tension the notebook ends on: erasure dropped the subject's 8 rows and bumped
the table v0 → v1, but **v0 still contains them**. "We support time travel" and
"we honour erasure" conflict unless the retention window is a deliberate,
written decision.

---

## One deviation from the shipped repo

`scripts/lakehouse.py` — `reset_catalog()` called
`shutil.rmtree(..., ignore_errors=True)` while pyiceberg's SQLAlchemy engine
still held `catalog.db` open. POSIX allows unlinking an open file; Windows
returns `WinError 32`, and `ignore_errors=True` swallowed it, so the catalog
was never actually dropped. `test_reset_catalog_does_not_touch_siblings`
failed on Windows and passes everywhere else.

Fix: track the live engine per catalog name and `engine.dispose()` before the
`rmtree`. Four lines, no behavioural change on POSIX, and no measured result in
this lab is affected. Without it `pytest` is 23/24 on Windows.
