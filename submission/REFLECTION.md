# REFLECTION — Top 5 Lakehouse Anti-Patterns

**Our biggest risk: treating the vector index as a system-of-record.**

Our RAG corpus lives in Postgres+pgvector, synced nightly by a one-way upsert.
NB7 reproduced our exact failure mode: after erasing 8 documents for one
subject, the lakehouse returned **0** hits and the external index still
returned **8**. An upsert-only sync has no opcode for "gone", so that is not a
lag window — it is permanent. A deletion request we already reported as
fulfilled would keep feeding that content into prompts.

Why us specifically: the index was built first and the table added later as
"the archive", so serving trusts the index. That inverts the rule — the index
should be a *rebuildable derived* artifact.

Two fixes NB7 made concrete. Short term, drive the sync from **Change Data
Feed** rather than polling for new rows; the CDF read emitted 8 delete events
carrying exactly which documents to evict. Long term, keep the embedding in
the row, where the table itself enforces the lifecycle.

The catch NB8 named: time travel means v0 still holds the erased rows. Erasure
completes only once retention expires them — so our retention window must
become a written decision, not a default.
