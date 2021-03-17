# DeltaSource &mdash; Streaming Source of Delta Data Source

`DeltaSource` is the streaming source of <<DeltaDataSource.md#, delta data source>> for streaming queries in Spark Structured Streaming.

TIP: Read up on https://jaceklaskowski.gitbooks.io/spark-structured-streaming/spark-sql-streaming-Source.html[Streaming Source] in https://bit.ly/spark-structured-streaming[The Internals of Spark Structured Streaming] online book.

`DeltaSource` is <<creating-instance, created>> when `DeltaDataSource` is requested for a <<DeltaDataSource.md#createSource, streaming source>>.

```
val q = spark
  .readStream               // Creating a streaming query
  .format("delta")          // Using delta data source
  .load("/tmp/delta/users") // Over data in a delta table
  .writeStream
  .format("memory")
  .option("queryName", "demo")
  .start
import org.apache.spark.sql.execution.streaming.{MicroBatchExecution, StreamingQueryWrapper}
val plan = q.asInstanceOf[StreamingQueryWrapper]
  .streamingQuery
  .asInstanceOf[MicroBatchExecution]
  .logicalPlan
import org.apache.spark.sql.execution.streaming.StreamingExecutionRelation
val relation = plan.collect { case r: StreamingExecutionRelation => r }.head

import org.apache.spark.sql.delta.sources.DeltaSource
assert(relation.source.asInstanceOf[DeltaSource])

scala> println(relation.source)
DeltaSource[file:/tmp/delta/users]
```

[[maxFilesPerTrigger]]
`DeltaSource` uses [maxFilesPerTrigger](options.md#maxFilesPerTrigger) option to limit the number of files to process when requested for the [file additions (with rate limit)](#getChangesWithRateLimit).

[[logging]]
[TIP]
====
Enable `ALL` logging level for `org.apache.spark.sql.delta.sources.DeltaSource` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

```
log4j.logger.org.apache.spark.sql.delta.sources.DeltaSource=ALL
```

Refer to [Logging](spark-logging.md).
====

== [[creating-instance]] Creating DeltaSource Instance

`DeltaSource` takes the following to be created:

* [[spark]] `SparkSession`
* [[deltaLog]] <<DeltaLog.md#, DeltaLog>> of the delta table to read data (as <<getBatch, micro-batches>>) from
* [[options]] [DeltaOptions](DeltaOptions.md)
* [[filters]] Filter expressions (default: no filters)

`DeltaSource` initializes the <<internal-properties, internal properties>>.

== [[getBatch]] Micro-Batch With Data Between Start And End Offsets (Streaming DataFrame) -- `getBatch` Method

[source, scala]
----
getBatch(
  start: Option[Offset],
  end: Offset): DataFrame
----

NOTE: `getBatch` is part of the `Source` contract (https://jaceklaskowski.gitbooks.io/spark-structured-streaming/spark-sql-streaming-Source.html[Spark Structured Streaming]) for a streaming `DataFrame` with data between the start and end offsets.

`getBatch` creates an <<DeltaSourceOffset.md#, DeltaSourceOffset>> for the <<tableId, tableId>> (aka <<DeltaSourceOffset.md#reservoirId, reservoirId>>) and the given `end` offset.

`getBatch` <<getChanges, gets the changes>> as follows...FIXME

== [[getOffset]] Latest Available Offset -- `getOffset` Method

[source, scala]
----
getOffset: Option[Offset]
----

NOTE: `getOffset` is part of the `Source` abstraction (https://jaceklaskowski.gitbooks.io/spark-structured-streaming/spark-sql-streaming-Source.html[Spark Structured Streaming]) for the latest available offset of this streaming source.

[[getOffset-currentOffset]]
`getOffset` calculates the latest offset (that a streaming query can use for the data of the next micro-batch) based on the <<previousOffset, previousOffset>> internal registry.

For no <<previousOffset, ending offset of the latest micro-batch>>, `getOffset` simply <<getStartingOffset, retrieves the starting offset>> (based on the latest version of the delta table).

When the <<previousOffset, ending offset of the latest micro-batch>> is defined (which means that the `DeltaSource` is requested for another micro-batch), `getOffset` takes the <<iteratorLast, last indexed AddFile>> from <<getChangesWithRateLimit, getChangesWithRateLimit>> for the <<previousOffset, previous ending offset>>. `getOffset` returns the <<previousOffset, previous ending offset>> when the last element was not available.

With the <<previousOffset, ending offset of the latest micro-batch>> and the <<iteratorLast, last indexed AddFile>> both available, `getOffset` creates a new <<DeltaSourceOffset.md#, DeltaSourceOffset>> for the version, index, and `isLast` flag from the last indexed <<AddFile.md#, AddFile>>.

[NOTE]
====
`isStartingVersion` local value is enabled (`true`) when the following holds:

* Version of the last indexed <<AddFile.md#, AddFile>> is equal to the <<DeltaSourceOffset.md#reservoirVersion, reservoirVersion>> of the <<previousOffset, previous ending offset>>

* <<DeltaSourceOffset.md#isStartingVersion, isStartingVersion>> flag of the <<previousOffset, previous ending offset>> is enabled (`true`)
====

In the end, `getOffset` prints out the following DEBUG message to the logs (using the <<previousOffset, previousOffset>> internal registry):

```
previousOffset -> currentOffset: [previousOffset] -> [currentOffset]
```

== [[stop]] Stopping -- `stop` Method

[source, scala]
----
stop(): Unit
----

NOTE: `stop` is part of the streaming `Source` contract (https://jaceklaskowski.gitbooks.io/spark-structured-streaming/spark-sql-streaming-Source.html[Spark Structured Streaming]) to stop this source and free up any resources allocated.

`stop` simply <<cleanUpSnapshotResources, cleanUpSnapshotResources>>.

== [[getStartingOffset]] Retrieving Starting Offset -- `getStartingOffset` Internal Method

[source, scala]
----
getStartingOffset(): Option[Offset]
----

`getStartingOffset` requests the <<deltaLog, DeltaLog>> for the version of the delta table (by requesting for the <<DeltaLog.md#snapshot, current state (snapshot)>> and then for the <<Snapshot.md#version, version>>).

`getStartingOffset` <<iteratorLast, takes the last file>> from the <<getChangesWithRateLimit, files added (with rate limit)>> for the version of the delta table, `-1L` as the `fromIndex`, and the `isStartingVersion` flag enabled (`true`).

`getStartingOffset` returns a new <<DeltaSourceOffset.md#, DeltaSourceOffset>> for the <<tableId, tableId>>, the version and the index of the last file added, and whether the last file added is the last file of its version.

`getStartingOffset` returns `None` (_offset not available_) when either happens:

* the version of the delta table is negative (below `0`)

* no files were added in the version

`getStartingOffset` throws an `AssertionError` when the version of the last file added is smaller than the delta table's version:

```
assertion failed: getChangesWithRateLimit returns an invalid version: [v] (expected: >= [version])
```

NOTE: `getStartingOffset` is used exclusively when `DeltaSource` is requested for the <<getOffset, latest available offset>>.

== [[getChanges]] `getChanges` Internal Method

[source, scala]
----
getChanges(
  fromVersion: Long,
  fromIndex: Long,
  isStartingVersion: Boolean): Iterator[IndexedFile]
----

`getChanges` branches per the given `isStartingVersion` flag (enabled or not):

* For `isStartingVersion` flag enabled (`true`), `getChanges` <<getSnapshotAt, gets the state (snapshot)>> for the given `fromVersion` followed by <<getChanges-filterAndIndexDeltaLogs, (filtered out) indexed AddFiles>> for the next version after the given `fromVersion`

* For `isStartingVersion` flag disabled (`false`), `getChanges` simply gives <<getChanges-filterAndIndexDeltaLogs, (filtered out) indexed AddFiles>> for the given `fromVersion`

[NOTE]
====
`isStartingVersion` flag simply adds <<getSnapshotAt, the state (snapshot)>> before <<getChanges-filterAndIndexDeltaLogs, (filtered out) indexed AddFiles>> when enabled (`true`).

`isStartingVersion` flag is enabled when `DeltaSource` is requested for the following:

* <<getBatch, Micro-batch with data between start and end offsets>> and the start offset is not given or is for the <<DeltaSourceOffset.md#isStartingVersion, starting version>>

* <<getOffset, Latest available offset>> with no <<previousOffset, end offset of the latest micro-batch>> or the <<previousOffset, end offset of the latest micro-batch>> for the <<DeltaSourceOffset.md#isStartingVersion, starting version>>
====

In the end, `getChanges` filters out (_excludes_) indexed <<AddFile.md#, AddFiles>> that are not with the version later than the given `fromVersion` or the index greater than the given `fromIndex`.

NOTE: `getChanges` is used when `DeltaSource` is requested for the <<getOffset, latest available offset>> (when requested for the <<getChangesWithRateLimit, files added (with rate limit)>>) and <<getBatch, getBatch>>.

=== [[getChanges-filterAndIndexDeltaLogs]] `filterAndIndexDeltaLogs` Internal Method

[source, scala]
----
filterAndIndexDeltaLogs(
  startVersion: Long): Iterator[IndexedFile]
----

`filterAndIndexDeltaLogs`...FIXME

== [[getChangesWithRateLimit]] Retrieving File Additions (With Rate Limit) -- `getChangesWithRateLimit` Internal Method

[source, scala]
----
getChangesWithRateLimit(
  fromVersion: Long,
  fromIndex: Long,
  isStartingVersion: Boolean): Iterator[IndexedFile]
----

`getChangesWithRateLimit` <<getChanges, get the changes>> (as indexed <<AddFile.md#, AddFiles>>) for the given `fromVersion`, `fromIndex`, and `isStartingVersion` flag.

`getChangesWithRateLimit` takes the configured number of `AddFiles` (up to the <<maxFilesPerTrigger, maxFilesPerTrigger>> option (if defined) or [1000](options.md#MAX_FILES_PER_TRIGGER_OPTION_DEFAULT)).

NOTE: `getChangesWithRateLimit` is used when `DeltaSource` is requested for the <<getOffset, latest available offset>>.

== [[getSnapshotAt]] Retrieving State Of Delta Table At Given Version -- `getSnapshotAt` Internal Method

[source, scala]
----
getSnapshotAt(
  version: Long): Iterator[IndexedFile]
----

`getSnapshotAt` requests the <<initialState, DeltaSourceSnapshot>> for the <<SnapshotIterator.md#iterator, data files>> (as indexed <<AddFile.md#, AddFiles>>).

In case the <<initialState, DeltaSourceSnapshot>> hasn't been initialized yet (`null`) or the requested version is different from the <<initialStateVersion, initialStateVersion>>, `getSnapshotAt` does the following:

. <<cleanUpSnapshotResources, cleanUpSnapshotResources>>

. Requests the <<deltaLog, DeltaLog>> for the <<DeltaLog.md#getSnapshotAt, state (snapshot) of the delta table>> at the version

. Creates a new <<DeltaSourceSnapshot.md#, DeltaSourceSnapshot>> for the state (snapshot) as the current <<initialState, DeltaSourceSnapshot>>

. Changes the <<initialStateVersion, initialStateVersion>> internal registry to the requested version

NOTE: `getSnapshotAt` is used when `DeltaSource` is requested to <<getChanges, getChanges>> (with `isStartingVersion` flag enabled).

== [[verifyStreamHygieneAndFilterAddFiles]] `verifyStreamHygieneAndFilterAddFiles` Internal Method

[source, scala]
----
verifyStreamHygieneAndFilterAddFiles(
  actions: Seq[Action]): Seq[Action]
----

`verifyStreamHygieneAndFilterAddFiles`...FIXME

NOTE: `verifyStreamHygieneAndFilterAddFiles` is used when `DeltaSource` is requested to <<getChanges, getChanges>>.

== [[cleanUpSnapshotResources]] `cleanUpSnapshotResources` Internal Method

[source, scala]
----
cleanUpSnapshotResources(): Unit
----

`cleanUpSnapshotResources`...FIXME

NOTE: `cleanUpSnapshotResources` is used when `DeltaSource` is requested to <<getSnapshotAt, getSnapshotAt>>, <<getBatch, getBatch>> and <<stop, stop>>.

== [[iteratorLast]] Retrieving Last Element From Iterator -- `iteratorLast` Internal Method

[source, scala]
----
iteratorLast[T](
  iter: Iterator[T]): Option[T]
----

`iteratorLast` simply returns the last element in the given `Iterator` or `None`.

NOTE: `iteratorLast` is used when `DeltaSource` is requested to <<getStartingOffset, getStartingOffset>> and <<getOffset, getOffset>>.

== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| initialState
a| [[initialState]] <<DeltaSourceSnapshot.md#, DeltaSourceSnapshot>>

Initially uninitialized (`null`).

Changes (along with the <<initialStateVersion, initialStateVersion>>) when `DeltaSource` is requested for the <<getSnapshotAt, snapshot at a given version>> (only when the versions are different)

Used when `DeltaSource` is requested for the <<getSnapshotAt, snapshot at a given version>>

Closed and dereferenced (`null`) when `DeltaSource` is requested to <<cleanUpSnapshotResources, cleanUpSnapshotResources>>

| initialStateVersion
a| [[initialStateVersion]] Version of the <<deltaLog, delta table>>

Initially `-1L` and changes (along with the <<initialState, initialState>>) to the version requested when `DeltaSource` is requested for the <<getSnapshotAt, snapshot at a given version>> (only when the versions are different)

Used when `DeltaSource` is requested to <<cleanUpSnapshotResources, cleanUpSnapshotResources>> (and unpersist the current snapshot)

| previousOffset
a| [[previousOffset]] Ending <<DeltaSourceOffset.md#, DeltaSourceOffset>> of the latest <<getBatch, micro-batch>>

Starts uninitialized (`null`).

Used when `DeltaSource` is requested for the <<getOffset, latest available offset>>.

| tableId
a| [[tableId]] Table ID

Used when...FIXME

|===
