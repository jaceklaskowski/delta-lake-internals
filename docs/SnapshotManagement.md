# SnapshotManagement

`SnapshotManagement` is an extension for [DeltaLog](DeltaLog.md) to manage [Snapshot](#currentSnapshot)s.

## <span id="currentSnapshot"><span id="snapshot"> Current Snapshot

`SnapshotManagement` manages `currentSnapshot` registry with the recently-loaded [Snapshot](Snapshot.md) (of a Delta table).

`currentSnapshot` is initialized as the [latest available Snapshot](#getSnapshotAtInit) right when [DeltaLog](DeltaLog.md) is created and [updated](#update) on demand.

`currentSnapshot`...FIXME

`currentSnapshot` is used when:

* `SnapshotManagement` is requested to...FIXME

### <span id="getSnapshotAtInit"> Loading Latest Snapshot at Initialization

```scala
getSnapshotAtInit: Snapshot
```

`getSnapshotAtInit` [getLogSegmentFrom](#getLogSegmentFrom) for the [last checkpoint](Checkpoints.md#lastCheckpoint).

`getSnapshotAtInit` prints out the following INFO message to the logs:

```text
Loading version [version][startCheckpoint]
```

`getSnapshotAtInit` [creates a Snapshot](#createSnapshot) for the log segment.

`getSnapshotAtInit` records the current time in [lastUpdateTimestamp](#lastUpdateTimestamp) registry.

`getSnapshotAtInit` prints out the following INFO message to the logs:

```text
Returning initial snapshot [snapshot]
```

### <span id="getLogSegmentFrom"> Fetching Log Files for Version Checkpointed

```scala
getLogSegmentFrom(
  startingCheckpoint: Option[CheckpointMetaData]): LogSegment
```

`getLogSegmentFrom` [fetches log files for the version](#getLogSegmentForVersion) (based on the optional `CheckpointMetaData` as the starting checkpoint version to start listing log files from).

## <span id="getLogSegmentForVersion"> Fetching Latest Checkpoint and Delta Log Files for Version

```scala
getLogSegmentForVersion(
  startCheckpoint: Option[Long],
  versionToLoad: Option[Long] = None): LogSegment
```

`getLogSegmentForVersion` [list all the files](#listFrom) (in a transaction log) from the given `startCheckpoint` (or defaults to `0`).

`getLogSegmentForVersion` filters out unnecessary files and leaves [checkpoint](#isCheckpointFile) and [delta](#isDeltaFile) files only.

`getLogSegmentForVersion` filters out checkpoint files of size `0`.

`getLogSegmentForVersion` takes all the files that are [older than](#getFileVersion) the requested `versionToLoad`.

`getLogSegmentForVersion` splits the files into [checkpoint](#isCheckpointFile) and [delta](#isDeltaFile) files.

`getLogSegmentForVersion` finds the latest checkpoint from the list.

In the end, `getLogSegmentForVersion` creates a [LogSegment](LogSegment.md) with the (checkpoint and delta) files.

`getLogSegmentForVersion` is used when:

* `SnapshotManagement` is requested for [getLogSegmentFrom](#getLogSegmentFrom), [updateInternal](#updateInternal) and [getSnapshotAt](#getSnapshotAt)

### <span id="listFrom"> Listing Files from Version Upwards

```scala
listFrom(
  startVersion: Long): Iterator[FileStatus]
```

`listFrom`...FIXME

## <span id="createSnapshot"> Creating Snapshot

```scala
createSnapshot(
  segment: LogSegment,
  minFileRetentionTimestamp: Long,
  timestamp: Long): Snapshot
```

`createSnapshot` [readChecksum](ReadChecksum.md#readChecksum) (for the version of the given [LogSegment](LogSegment.md)) and creates a [Snapshot](Snapshot.md).

`createSnapshot` is used when:

* `SnapshotManagement` is requested for [getSnapshotAtInit](#getSnapshotAtInit), [updateInternal](#updateInternal) and [getSnapshotAt](#getSnapshotAt)

## <span id="lastUpdateTimestamp"> Last Successful Update Timestamp

`SnapshotManagement` uses `lastUpdateTimestamp` internal registry for the timestamp of the last successful update.

## <span id="update"> Updating Current Snapshot

```scala
update(
  stalenessAcceptable: Boolean = false): Snapshot
```

`update` determines whether to do update asynchronously or not based on the input `stalenessAcceptable` flag and [isSnapshotStale](#isSnapshotStale).

With `stalenessAcceptable` flag turned off (the default value) and the state snapshot is not [stale](#isSnapshotStale), `update` [updates](#updateInternal) (with `isAsync` flag turned off).

`update`...FIXME

### <span id="update-usage"> Usage

`update` is used when:

* `DeltaHistoryManager` is requested to [getHistory](DeltaHistoryManager.md#getHistory), [getActiveCommitAtTime](DeltaHistoryManager.md#getActiveCommitAtTime), [checkVersionExists](DeltaHistoryManager.md#checkVersionExists)
* `DeltaLog` is requested to [start a transaction](DeltaLog.md#startTransaction)
* `OptimisticTransactionImpl` is requested to [doCommit](OptimisticTransactionImpl.md#doCommit) and [getNextAttemptVersion](OptimisticTransactionImpl.md#getNextAttemptVersion)
* `DeltaTableV2` is requested for a [Snapshot](DeltaTableV2.md#snapshot)
* `TahoeLogFileIndex` is requested for a [Snapshot](TahoeLogFileIndex.md#getSnapshot)
* `DeltaSource` is requested for the [getStartingVersion](DeltaSource.md#getStartingVersion)
* In [Delta commands](commands/DeltaCommand.md)...

### <span id="isSnapshotStale"> isSnapshotStale

```scala
isSnapshotStale: Boolean
```

`isSnapshotStale` reads [spark.databricks.delta.stalenessLimit](DeltaSQLConf.md#DELTA_ASYNC_UPDATE_STALENESS_TIME_LIMIT) configuration property.

`isSnapshotStale` is enabled (`true`) when any of the following holds:

1. [spark.databricks.delta.stalenessLimit](DeltaSQLConf.md#DELTA_ASYNC_UPDATE_STALENESS_TIME_LIMIT) configuration property is `0` (the default)
1. Internal [lastUpdateTimestamp](#lastUpdateTimestamp) has never been updated (and is below `0`) or is at least [spark.databricks.delta.stalenessLimit](DeltaSQLConf.md#DELTA_ASYNC_UPDATE_STALENESS_TIME_LIMIT) configuration property old

### <span id="tryUpdate"> tryUpdate

```scala
tryUpdate(
  isAsync: Boolean = false): Snapshot
```

`tryUpdate`...FIXME

### <span id="updateInternal"> updateInternal

```scala
updateInternal(
  isAsync: Boolean): Snapshot // (1)
```

1. `isAsync` flag is not used

`updateInternal` requests the [current Snapshot](#currentSnapshot) for the [LogSegment](Snapshot.md#logSegment) that is in turn requested for the [checkpointVersion](LogSegment.md#checkpointVersion). `updateInternal` [gets the LogSegment](#getLogSegmentForVersion) for the `checkpointVersion`.

If the `LogSegment`s are equal (and so no new files have been added), `updateInternal` updates the [lastUpdateTimestamp](#lastUpdateTimestamp) registry to the current timestamp and returns the [currentSnapshot](#currentSnapshot).

Otherwise, if the fetched `LogSegment` is different than the [current Snapshot's](Snapshot.md#logSegment), `updateInternal` prints out the following INFO message to the logs:

```text
Loading version [version][ starting from checkpoint version [v]]
```

`updateInternal` [creates a new Snapshot](#createSnapshot) with the fetched `LogSegment`.

`updateInternal` [replaces Snapshots](#replaceSnapshot) and prints out the following INFO message to the logs:

```text
Updated snapshot to [newSnapshot]
```

### <span id="replaceSnapshot"> Replacing Snapshots

```scala
replaceSnapshot(
  newSnapshot: Snapshot): Unit
```

`replaceSnapshot` requests the [currentSnapshot](#currentSnapshot) to [uncache](StateCache.md#uncache) (and drop any cached data) and makes the given `newSnapshot` the [current one](#currentSnapshot).

## Demo

```scala
import org.apache.spark.sql.delta.DeltaLog
val log = DeltaLog.forTable(spark, dataPath)

import org.apache.spark.sql.delta.SnapshotManagement
assert(log.isInstanceOf[SnapshotManagement], "DeltaLog is a SnapshotManagement")
```

```scala
val snapshot = log.update(stalenessAcceptable = false)
```

```text
scala> :type snapshot
org.apache.spark.sql.delta.Snapshot

assert(snapshot.version == 0)
```

## Logging

As an extension of [DeltaLog](DeltaLog.md), use [DeltaLog](DeltaLog.md#logging) logging to see what happens inside.
