# MetadataCleanup

`MetadataCleanup` is an abstraction of [metadata cleaners](#implementations) that can [clean up](#doLogCleanup) expired checkpoints and delta logs of a [delta table](#self).

<span id="self">
`MetadataCleanup` requires to be used with [DeltaLog](DeltaLog.md) (or subtypes) only.

## Implementations

* [DeltaLog](DeltaLog.md)

## Table Properties

### <span id="enableExpiredLogCleanup"> enableExpiredLogCleanup

`MetadataCleanup` uses [enableExpiredLogCleanup](DeltaConfigs.md#ENABLE_EXPIRED_LOG_CLEANUP) table configuration to enable [log cleanup](#doLogCleanup).

### <span id="deltaRetentionMillis"> logRetentionDuration

`MetadataCleanup` uses [logRetentionDuration](DeltaConfigs.md#LOG_RETENTION) table configuration for [cleanUpExpiredLogs](#cleanUpExpiredLogs) (to determine `fileCutOffTime`).

## <span id="doLogCleanup"> Cleaning Up Expired Logs

```scala
doLogCleanup(): Unit
```

`doLogCleanup` is part of the [Checkpoints](Checkpoints.md#doLogCleanup) abstraction.

---

`doLogCleanup` [cleanUpExpiredLogs](#cleanUpExpiredLogs) when [enabled](#enableExpiredLogCleanup).

### <span id="cleanUpExpiredLogs"> cleanUpExpiredLogs

```scala
cleanUpExpiredLogs(): Unit
```

`cleanUpExpiredLogs` calculates a `fileCutOffTime` based on the [current time](DeltaLog.md#clock) and the [logRetentionDuration](#deltaRetentionMillis) table property.

`cleanUpExpiredLogs` prints out the following INFO message to the logs:

```text
Starting the deletion of log files older than [date]
```

`cleanUpExpiredLogs` [finds the expired delta logs](#listExpiredDeltaLogs) (based on the `fileCutOffTime`) and deletes the files (using Hadoop's [FileSystem.delete]({{ hadoop.api }}/org/apache/hadoop/fs/FileSystem.html#delete(org.apache.hadoop.fs.Path,%20boolean)) non-recursively). `cleanUpExpiredLogs` counts the files deleted (and uses it in the summary INFO message).

In the end, `cleanUpExpiredLogs` prints out the following INFO message to the logs:

```text
Deleted [numDeleted] log files older than [date]
```

### <span id="listExpiredDeltaLogs"> Finding Expired Log Files

```scala
listExpiredDeltaLogs(
  fileCutOffTime: Long): Iterator[FileStatus]
```

`listExpiredDeltaLogs` [loads the most recent checkpoint](Checkpoints.md#lastCheckpoint) if available.

If the last checkpoint is not available, `listExpiredDeltaLogs` returns an empty iterator.

`listExpiredDeltaLogs` requests the [LogStore](DeltaLog.md#store) for the [paths](storage/LogStore.md#listFrom) (in the same directory) that are (lexicographically) greater or equal to the ``0``th checkpoint file (per [checkpointPrefix](FileNames.md#checkpointPrefix) format) of the [checkpoint](FileNames.md#isCheckpointFile) and [delta](FileNames.md#isDeltaFile) files in the [log directory](DeltaLog.md#logPath).

In the end, `listExpiredDeltaLogs` creates a `BufferingLogDeletionIterator` that...FIXME

## Logging

Enable `ALL` logging level for the [Implementations](#implementations) logger to see what happens inside.
