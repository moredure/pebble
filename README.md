# Pebble [![Build Status](https://travis-ci.com/cockroachdb/pebble.svg?branch=master)](https://travis-ci.com/cockroachdb/pebble) [![GoDoc](https://godoc.org/github.com/cockroachdb/pebble?status.svg)](https://godoc.org/github.com/cockroachdb/pebble)

#### [Nightly benchmarks](https://cockroachdb.github.io/pebble/)

Pebble is a LevelDB/RocksDB inspired key-value store focused on
performance and internal usage by CockroachDB. Pebble inherits the
RocksDB file formats and a few extensions such as range deletion
tombstones, table-level bloom filters, and updates to the MANIFEST
format.

Pebble intentionally does not aspire to include every feature in
RocksDB and is specifically targetting the use case and feature set
needed by CockroachDB:

* Block-based tables
* Checkpoints
* Indexed batches
* Iterator options (lower/upper bound, table filter)
* Level-based compaction
* Manual compaction
* Merge operator
* Prefix bloom filters
* Prefix iteration
* Range deletion tombstones
* Reverse iteration
* SSTable ingestion
* Single delete
* Snapshots
* Table-level bloom filters

RocksDB has a large number of features that are not implemented in
Pebble:

* Backups
* Column families
* Delete files in range
* FIFO compaction style
* Forward iterator / tailing iterator
* Hash table format
* Memtable bloom filter
* Persistent cache
* Pin iterator key / value
* Plain table format
* SSTable ingest-behind
* Sub-compactions
* Transactions
* Universal compaction style

***WARNING***: Pebble may silently corrupt data or behave incorrectly if
used with a RocksDB database that uses a feature Pebble doesn't
support. Caveat emptor!

## Advantages

Pebble offers several improvements over RocksDB:

* Faster reverse iteration via backwards links in the memtable's
  skiplist.
* Faster commit pipeline that achieves better concurrency.
* Seamless merged iteration of indexed batches. The mutations in the
  batch conceptually occupy another memtable level.
* Smaller, more approachable code base.

See the [Pebble vs RocksDB: Implementation
Differences](docs/rocksdb.md) doc for more details on implementation
differences.

## RocksDB Compatibility

Pebble strives for forward compatibility with RocksDB 6.2.1 (the
latest version of RocksDB used by CockroachDB). Forward compatibility
means that a DB generated by RocksDB can be used by Pebble. Currently,
Pebble provides bidirectional compatibility with RocksDB (a Pebble
generated DB can be used by RocksDB), but that will change in the
future as new functionality is introduced to Pebble. In general,
Pebble only provides compatibility with the subset of functionality
and configuration used by CockroachDB. The scope of RocksDB
functionality and configuration is too large to adequately test and
document all the incompatibilities. The list below contains known
incompatibilities.

* Pebble's use of WAL recycling is only compatible with RocksDB's
  `kTolerateCorruptedTailRecords` WAL recovery mode. Older versions of
  RocksDB would automatically map incompatible WAL recovery modes to
  `kTolerateCorruptedTailRecords`. New versions of RocksDB will
  disable WAL recycling.
* Column families. Pebble does not support column families, nor does
  it attempt to detect their usage when opening a DB that may contain
  them.
* Hash table format. Pebble does not support the hash table sstable
  format.
* Plain table format. Pebble does not support the plain table sstable
  format.
* SSTable format version 3 and 4. Pebble does not currently support
  version 3 and version 4 format sstables. The sstable format version
  is controlled by the `BlockBasedTableOptions::format_version`
  option. See [#97](https://github.com/cockroachdb/pebble/issues/97).

## Pedigree

Pebble is based on the incomplete Go version of LevelDB:

https://github.com/golang/leveldb

The Go version of LevelDB is based on the C++ original:

https://github.com/google/leveldb

Optimizations and inspiration were drawn from RocksDB:

https://github.com/facebook/rocksdb

## Getting Started

### Example Code

```go
package main

import (
	"fmt"
	"log"

	"github.com/cockroachdb/pebble"
)

func main() {
	db, err := pebble.Open("demo", &pebble.Options{})
	if err != nil {
		log.Fatal(err)
	}
	key := []byte("hello")
	if err := db.Set(key, []byte("world"), pebble.Sync); err != nil {
		log.Fatal(err)
	}
	value, closer, err := db.Get(key)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%s %s\n", key, value)
	if err := closer.Close(); err != nil {
		log.Fatal(err)
	}
	if err := db.Close(); err != nil {
		log.Fatal(err)
	}
}
```
