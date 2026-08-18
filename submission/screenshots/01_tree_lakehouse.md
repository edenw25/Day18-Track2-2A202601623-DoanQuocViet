# Evidence 1 — `tree _lakehouse/` (lightweight path)

Machine: Windows 11, Python 3.11.8, deltalake 1.6.2, pyiceberg 0.11.1, duckdb 1.5.5
Captured: 2026-08-18 04:43 UTC

```
_lakehouse
  blobs
  bronze
    agent_traces
      _delta_log
    docs_multimodal
      _delta_log
    llm_calls_raw
      _delta_log
  gold
    agent_performance
      _delta_log
    llm_daily_metrics
      _delta_log
      date=2026-04-01
      date=2026-04-02
      date=2026-04-03
      date=2026-04-04
      date=2026-04-05
      date=2026-04-06
      date=2026-04-07
      date=2026-04-08
  iceberg
    nb5
      warehouse
    nb6
      warehouse
    nb8
      warehouse
  scratch
    customers_tt
      _delta_log
    docs_cdf
      _change_data
      _delta_log
    docs_intable
      _delta_log
    emb_f32
      _delta_log
    emb_int8
      _delta_log
    events_smallfiles
      _delta_log
    maint_events
      _delta_log
    media_inline
      _delta_log
    media_pointer
      _delta_log
    users_delta
      _delta_log
    vector_index_external
      _delta_log
  silver
    agent_trajectories
      _delta_log
      agent_version=policy-v2
      agent_version=policy-v3
    llm_calls
      _delta_log
      date=2026-04-01
      date=2026-04-02
      date=2026-04-03
      date=2026-04-04
      date=2026-04-05
      date=2026-04-06
      date=2026-04-07
      date=2026-04-08
    training_corpus_governed
      _delta_log
      provenance_bucket=UNCLASSIFIED
      provenance_bucket=licensed
      provenance_bucket=public_domain
      provenance_bucket=scraped_optout_checked
      provenance_bucket=synthetic
```

## File counts per table (depth 2)

```
_lakehouse/bronze/agent_traces                           2 files (   1 parquet)       36K
_lakehouse/bronze/docs_multimodal                        2 files (   1 parquet)      2.6M
_lakehouse/bronze/llm_calls_raw                          2 files (   1 parquet)       14M
_lakehouse/gold/agent_performance                        2 files (   1 parquet)      8.0K
_lakehouse/gold/llm_daily_metrics                       18 files (  16 parquet)      124K
_lakehouse/iceberg/nb5                                  52 files (  13 parquet)      428K
_lakehouse/iceberg/nb6                                  66 files (  20 parquet)      568K
_lakehouse/iceberg/nb8                                   6 files (   1 parquet)       48K
_lakehouse/scratch/customers_tt                          9 files (   4 parquet)      2.1M
_lakehouse/scratch/docs_cdf                              5 files (   3 parquet)       40K
_lakehouse/scratch/docs_intable                          4 files (   2 parquet)      5.0M
_lakehouse/scratch/emb_f32                               2 files (   1 parquet)      2.6M
_lakehouse/scratch/emb_int8                              2 files (   1 parquet)      456K
_lakehouse/scratch/events_smallfiles                   527 files ( 324 parquet)       29M
_lakehouse/scratch/maint_events                        218 files (  13 parquet)      7.5M
_lakehouse/scratch/media_inline                          2 files (   1 parquet)       13M
_lakehouse/scratch/media_pointer                         2 files (   1 parquet)      8.0K
_lakehouse/scratch/users_delta                           4 files (   2 parquet)       20K
_lakehouse/scratch/vector_index_external                 2 files (   1 parquet)      2.6M
_lakehouse/silver/agent_trajectories                     5 files (   3 parquet)       80K
_lakehouse/silver/llm_calls                              9 files (   8 parquet)       11M
_lakehouse/silver/training_corpus_governed              11 files (   9 parquet)      132K
```
