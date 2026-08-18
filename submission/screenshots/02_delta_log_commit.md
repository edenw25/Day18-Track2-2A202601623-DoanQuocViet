# Evidence 2 — contents of one `_delta_log/*.json`

Table: `_lakehouse/silver/llm_calls` (NB4 Silver layer)  
Commit: `_delta_log/00000000000000000000.json` — the v0 commit.

Each line is one **action**. This is the whole of Delta's ACID mechanism:
a commit is an atomic rename of this JSON file into `_delta_log/`. Readers
replay the actions in order; a half-written commit is simply a file that
never got renamed, so it is never seen.

| Action | Count | What it does |
|---|---:|---|
| `commitInfo` | 1 | Who/what/when — the audit row `history()` surfaces |
| `protocol` | 1 | Min reader/writer version a client must support |
| `metaData` | 1 | Schema (as JSON), partition columns, format |
| `add` | 8 | One per data file, **with min/max stats** — the basis of file skipping |

```json
{
  "commitInfo": {
    "timestamp": 1787028085492,
    "operation": "WRITE",
    "operationParameters": {
      "partitionBy": "[\"date\"]",
      "mode": "Overwrite"
    },
    "engineInfo": "delta-rs:py-1.6.2",
    "clientVersion": "delta-rs.py-1.6.2",
    "operationMetrics": {
      "execution_time_ms": 160,
      "num_added_files": 8,
      "num_added_rows": 190052,
      "num_partitions": 0,
      "num_removed_files": 0
    }
  }
}
{
  "protocol": {
    "minReaderVersion": 1,
    "minWriterVersion": 2
  }
}
{
  "metaData": {
    "id": "e918b8e6-fb13-4923-987c-8277f4082f1e",
    "name": null,
    "description": null,
    "format": {
      "provider": "parquet",
      "options": {}
    },
    "schemaString": "{\"type\":\"struct\",\"fields\":[{\"name\":\"request_id\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}},{\"name\":\"ts\",\"type\":\"timestamp\",\"nullable\":true,\"metadata\":{}},{\"name\":\"date\",\"type\":\"date\",\"nullable\":true,\"metadata\":{}},{\"name\":\"model\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}},{\"name\":\"user_id\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}},{\"name\":\"prompt_tokens\",\"type\":\"integer\",\"nullable\":true,\"metadata\":{}},{\"name\":\"completion_tokens\",\"type\":\"integer\",\"nullable\":true,\"metadata\":{}},{\"name\":\"latency_ms\",\"type\":\"integer\",\"nullable\":true,\"metadata\":{}},{\"name\":\"status\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}}]}",
    "partitionColumns": [
      "date"
    ],
    "createdTime": 1787028085332,
    "configuration": {}
  }
}
{
  "add": {
    "path": "date=2026-04-07/part-00000-97becc2c-3a67-4d16-89bc-df8d50546aee-c000.snappy.parquet",
    "partitionValues": {
      "date": "2026-04-07"
    },
    "size": 1487868,
    "modificationTime": 1787028085470,
    "dataChange": true,
    "stats": {
      "numRecords": 27112,
      "minValues": {
        "status": "error",
        "request_id": "000218e7-424c-457c-8cda-81558380b869",
        "prompt_tokens": 50,
        "latency_ms": 50,
        "ts": "2026-04-06T17:00:02Z",
        "model": "claude-haiku-4-5",
        "completion_tokens": 20,
        "user_id": "u_1"
      },
      "maxValues": {
        "request_id": "fffb21db-c921-4e48-9f14-3951a45e1eb4",
        "prompt_tokens": 4000,
        "completion_tokens": 2000,
        "model": "claude-sonnet-4-6",
        "ts": "2026-04-07T16:59:54Z",
        "status": "rate_limited",
        "latency_ms": 8458,
        "user_id": "u_999"
      },
      "nullCount": {
        "user_id": 0,
        "latency_ms": 0,
        "status": 0,
        "request_id": 0,
        "model": 0,
        "completion_tokens": 0,
        "prompt_tokens": 0,
        "ts": 0
      }
    },
    "tags": null,
    "baseRowId": null,
    "defaultRowCommitVersion": null,
    "clusteringProvider": null
  }
}
{
  "add": {
    "path": "date=2026-04-01/part-00000-b435b7b9-e9b5-4b72-87de-55e9a60a5f8e-c000.snappy.parquet",
    "partitionValues": {
      "date": "2026-04-01"
    },
    "size": 1076039,
    "modificationTime": 1787028085470,
    "dataChange": true,
    "stats": {
      "numRecords": 19271,
      "minValues": {
        "ts": "2026-04-01T00:00:00Z",
        "status": "error",
        "latency_ms": 50,
        "prompt_tokens": 50,
        "user_id": "u_1",
        "request_id": "00116344-5c8b-4376-8ad9-bbf820f67945",
        "completion_tokens": 20,
        "model": "claude-haiku-4-5"
      },
      "maxValues": {
        "ts": "2026-04-01T16:59:59Z",
        "latency_ms": 8103,
        "prompt_tokens": 4000,
        "status": "rate_limited",
        "completion_tokens": 2000,
        "model": "claude-sonnet-4-6",
        "user_id": "u_999",
        "request_id": "fffd990f-9321-4364-b6e1-b915dc5a2dfe"
      },
      "nullCount": {
        "ts": 0,
        "latency_ms": 0,
        "status": 0,
        "completion_tokens": 0,
        "model": 0,
        "user_id": 0,
        "prompt_tokens": 0,
        "request_id": 0
      }
    },
    "tags": null,
    "baseRowId": null,
    "defaultRowCommitVersion": null,
    "clusteringProvider": null
  }
}
{
  "add": {
    "path": "date=2026-04-06/part-00000-e137092e-0eb8-424a-a047-cedc9d5b4e63-c000.snappy.parquet",
    "partitionValues": {
      "date": "2026-04-06"
    },
    "size": 1490507,
    "modificationTime": 1787028085480,
    "dataChange": true,
    "stats": {
      "numRecords": 27152,
      "minValues": {
        "completion_tokens": 20,
        "user_id": "u_1",
        "model": "claude-haiku-4-5",
        "latency_ms": 50,
        "prompt_tokens": 50,
        "status": "error",
        "request_id": "0000097f-462f-4fec-8dd3-3723d5fb22c3",
        "ts": "2026-04-05T17:00:00Z"
      },
      "maxValues": {
        "model": "claude-sonnet-4-6",
        "status": "rate_limited",
        "user_id": "u_999",
        "request_id": "fffed799-ac18-42a3-96c0-dfaf0a83475e",
        "latency_ms": 8380,
        "prompt_tokens": 4000,
        "completion_tokens": 2000,
        "ts": "2026-04-06T16:59:59Z"
      },
      "nullCount": {
        "ts": 0,
        "completion_tokens": 0,
        "model": 0,
        "prompt_tokens": 0,
        "latency_ms": 0,
        "user_id": 0,
        "request_id": 0,
        "status": 0
      }
    },
    "tags": null,
    "baseRowId": null,
    "defaultRowCommitVersion": null,
    "clusteringProvider": null
  }
}
{
  "add": {
    "path": "date=2026-04-04/part-00000-8d33ba98-df44-4ff1-b9a8-8ece43227f82-c000.snappy.parquet",
    "partitionValues": {
      "date": "2026-04-04"
    },
    "size": 1491529,
    "modificationTime": 1787028085480,
    "dataChange": true,
    "stats": {
      "numRecords": 27176,
      "minValues": {
        "ts": "2026-04-03T17:00:00Z",
        "latency_ms": 50,
        "status": "error",
        "user_id": "u_1",
        "request_id": "000cff6c-81f6-4672-9e0a-7a26eb554ebf",
        "prompt_tokens": 50,
        "model": "claude-haiku-4-5",
        "completion_tokens": 20
      },
      "maxValues": {
        "user_id": "u_999",
        "latency_ms": 7430,
        "ts": "2026-04-04T16:59:58Z",
        "model": "claude-sonnet-4-6",
        "request_id": "fffc894c-0423-41ac-a6ae-3ccd2183051e",
        "prompt_tokens": 4000,
        "completion_tokens": 2000,
        "status": "rate_limited"
      },
      "nullCount": {
        "model": 0,
        "ts": 0,
        "prompt_tokens": 0,
        "completion_tokens": 0,
        "user_id": 0,
        "status": 0,
        "latency_ms": 0,
        "request_id": 0
      }
    },
    "tags": null,
    "baseRowId": null,
    "defaultRowCommitVersion": null,
    "clusteringProvider": null
  }
}
{
  "add": {
    "path": "date=2026-04-08/part-00000-823198c3-7c93-431b-8baf-949510b0e446-c000.snappy.parquet",
    "partitionValues": {
      "date": "2026-04-08"
    },
    "size": 467233,
    "modificationTime": 1787028085479,
    "dataChange": true,
    "stats": {
      "numRecords": 7915,
      "minValues": {
        "ts": "2026-04-07T17:00:01Z",
        "completion_tokens": 20,
        "prompt_tokens": 50,
        "status": "error",
        "latency_ms": 50,
        "request_id": "0005b243-f515-48de-8fe6-067709c10862",
        "model": "claude-haiku-4-5",
        "user_id": "u_1"
      },
      "maxValues": {
        "user_id": "u_999",
        "status": "rate_limited",
        "latency_ms": 7398,
        "model": "claude-sonnet-4-6",
        "request_id": "fffee95c-4a93-4f5b-bfd4-564a3be6562f",
        "ts": "2026-04-07T23:59:56Z",
        "prompt_tokens": 4000,
        "completion_tokens": 2000
      },
      "nullCount": {
        "request_id": 0,
        "model": 0,
        "completion_tokens": 0,
        "latency_ms": 0,
        "ts": 0,
        "user_id": 0,
        "prompt_tokens": 0,
        "status": 0
      }
    },
    "tags": null,
    "baseRowId": null,
    "defaultRowCommitVersion": null,
    "clusteringProvider": null
  }
}
{
  "add": {
    "path": "date=2026-04-02/part-00000-b3ba7fbc-45b5-4c72-9658-20bad41d4a78-c000.snappy.parquet",
    "partitionValues": {
      "date": "2026-04-02"
    },
    "size": 1492880,
    "modificationTime": 1787028085480,
    "dataChange": true,
    "stats": {
      "numRecords": 27199,
      "minValues": {
        "request_id": "000236fd-e89f-4cc5-9a9f-1b97133ae3de",
        "ts": "2026-04-01T17:00:02Z",
        "status": "error",
        "completion_tokens": 20,
        "prompt_tokens": 50,
        "model": "claude-haiku-4-5",
        "latency_ms": 50,
        "user_id": "u_1"
      },
      "maxValues": {
        "request_id": "fffe3d89-8203-4975-977a-8c4fc8f44e15",
        "status": "rate_limited",
        "completion_tokens": 2000,
        "model": "claude-sonnet-4-6",
        "user_id": "u_999",
        "ts": "2026-04-02T16:59:58Z",
        "prompt_tokens": 4000,
        "latency_ms": 8278
      },
      "nullCount": {
        "latency_ms": 0,
        "model": 0,
        "status": 0,
        "user_id": 0,
        "completion_tokens": 0,
        "request_id": 0,
        "prompt_tokens": 0,
        "ts": 0
      }
    },
    "tags": null,
    "baseRowId": null,
    "defaultRowCommitVersion": null,
    "clusteringProvider": null
  }
}
{
  "add": {
    "path": "date=2026-04-03/part-00000-f3f502a3-6af8-4e57-9a32-ae6d8ded92e9-c000.snappy.parquet",
    "partitionValues": {
      "date": "2026-04-03"
    },
    "size": 1487641,
    "modificationTime": 1787028085492,
    "dataChange": true,
    "stats": {
      "numRecords": 27096,
      "minValues": {
        "ts": "2026-04-02T17:00:01Z",
        "model": "claude-haiku-4-5",
        "user_id": "u_1",
        "latency_ms": 50,
        "request_id": "0006b400-4122-4080-8e67-649f59e8eb2d",
        "prompt_tokens": 50,
        "completion_tokens": 20,
        "status": "error"
      },
      "maxValues": {
        "request_id": "fffd0415-df8d-440a-9a90-6cbb845c0948",
        "model": "claude-sonnet-4-6",
        "latency_ms": 7595,
        "ts": "2026-04-03T16:59:54Z",
        "completion_tokens": 2000,
        "status": "rate_limited",
        "user_id": "u_999",
        "prompt_tokens": 4000
      },
      "nullCount": {
        "latency_ms": 0,
        "request_id": 0,
        "ts": 0,
        "completion_tokens": 0,
        "user_id": 0,
        "model": 0,
        "prompt_tokens": 0,
        "status": 0
      }
    },
    "tags": null,
    "baseRowId": null,
    "defaultRowCommitVersion": null,
    "clusteringProvider": null
  }
}
{
  "add": {
    "path": "date=2026-04-05/part-00000-7138c3b9-e4fe-47d6-912c-f58e283d2639-c000.snappy.parquet",
    "partitionValues": {
      "date": "2026-04-05"
    },
    "size": 1495589,
    "modificationTime": 1787028085492,
    "dataChange": true,
    "stats": {
      "numRecords": 27131,
      "minValues": {
        "user_id": "u_1",
        "completion_tokens": 20,
        "model": "claude-haiku-4-5",
        "latency_ms": 50,
        "ts": "2026-04-04T17:00:01Z",
        "request_id": "000143d9-e5f8-44b4-9f37-4884f1785014",
        "prompt_tokens": 50,
        "status": "error"
      },
      "maxValues": {
        "request_id": "ffffe1f2-94e3-419f-a925-54e3b890986d",
        "status": "rate_limited",
        "prompt_tokens": 4000,
        "completion_tokens": 2000,
        "user_id": "u_999",
        "latency_ms": 7632,
        "ts": "2026-04-05T16:59:57Z",
        "model": "claude-sonnet-4-6"
      },
      "nullCount": {
        "model": 0,
        "ts": 0,
        "user_id": 0,
        "prompt_tokens": 0,
        "completion_tokens": 0,
        "status": 0,
        "latency_ms": 0,
        "request_id": 0
      }
    },
    "tags": null,
    "baseRowId": null,
    "defaultRowCommitVersion": null,
    "clusteringProvider": null
  }
}
```
