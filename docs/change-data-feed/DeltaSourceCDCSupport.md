# DeltaSourceCDCSupport

`DeltaSourceCDCSupport` is an abstraction to bring [CDC support](index.md) to [DeltaSource](../delta/DeltaSource.md).

## <span id="getCDCFileChangesAndCreateDataFrame"> getCDCFileChangesAndCreateDataFrame

```scala
getCDCFileChangesAndCreateDataFrame(
  startVersion: Long,
  startIndex: Long,
  isStartingVersion: Boolean,
  endOffset: DeltaSourceOffset): DataFrame
```

`getCDCFileChangesAndCreateDataFrame` [changesToDF](CDCReader.md#changesToDF) with the following:

* [getFileChangesForCDC](#getFileChangesForCDC) (with no `AdmissionLimits`) for the versions and their [FileAction](../FileAction.md)s
* `isStreaming` enabled

In the end, `getCDCFileChangesAndCreateDataFrame` returns the `DataFrame` with the file changes (out of the `CDCVersionDiffInfo`).

---

`getCDCFileChangesAndCreateDataFrame` is used when:

* `DeltaSourceBase` is requested to [createDataFrameBetweenOffsets](../delta/DeltaSourceBase.md#createDataFrameBetweenOffsets) (and to [getFileChangesAndCreateDataFrame](../delta/DeltaSourceBase.md#getFileChangesAndCreateDataFrame)) for `DeltaSource` for a [streaming DataFrame (with data between the start and end offsets)](../delta/DeltaSource.md#getBatch) with the [readChangeFeed](../delta/options.md#readChangeFeed) option enabled

## <span id="getFileChangesForCDC"> getFileChangesForCDC

```scala
getFileChangesForCDC(
  fromVersion: Long,
  fromIndex: Long,
  isStartingVersion: Boolean,
  limits: Option[AdmissionLimits],
  endOffset: Option[DeltaSourceOffset]): Iterator[(Long, Iterator[IndexedFile])]
```

With [isStartingVersion](#getFileChangesForCDC-isStartingVersion) on (`true`), `getFileChangesForCDC` [gets the snapshot](../delta/DeltaSource.md#getSnapshotAt) at the `fromVersion` version and turns`dataChange` on for all [AddFile](../AddFile.md)s. `getFileChangesForCDC` creates a [IndexedChangeFileSeq](IndexedChangeFileSeq.md) (with the snapshot and `isInitialSnapshot` flag enabled). `getFileChangesForCDC`...FIXME

With [isStartingVersion](#getFileChangesForCDC-isStartingVersion) off (`false`), `getFileChangesForCDC` [filterAndIndexDeltaLogs](#filterAndIndexDeltaLogs) for the `fromVersion` version.

That gives a collection of a version and [IndexedChangeFileSeq](IndexedChangeFileSeq.md) pairs.

In the end, `getFileChangesForCDC` requests all the `IndexedChangeFileSeq`s to [filterFiles](IndexedChangeFileSeq.md#filterFiles) (with `fromVersion`, `fromIndex`, `limits` and `endOffset` arguments).

---

`getFileChangesForCDC` is used when:

* `DeltaSourceBase` is requested to [getFileChangesWithRateLimit](../delta/DeltaSourceBase.md#getFileChangesWithRateLimit)
* `DeltaSourceCDCSupport` is requested to [getCDCFileChangesAndCreateDataFrame](#getCDCFileChangesAndCreateDataFrame)

### <span id="getFileChangesForCDC-isStartingVersion"> isStartingVersion

`getFileChangesForCDC` is given `isStartingVersion` flag when executed:

* `true` for the following:
    * `DeltaSource` when [getStartingVersion](../delta/DeltaSource.md#getStartingVersion) is undefined (returns `None`)
    * `DeltaSource` when [getBatch](../delta/DeltaSource.md#getBatch) with `startOffsetOption` and [getStartingVersion](../delta/DeltaSource.md#getStartingVersion) both undefined (`None`s)

* `false` for the following:
    * `DeltaSource` when [getBatch](../delta/DeltaSource.md#getBatch) with `startOffsetOption` undefined but [getStartingVersion](../delta/DeltaSource.md#getStartingVersion) specified

* `true` or `false` for the following:
    * `DeltaSourceBase` when [getNextOffsetFromPreviousOffset](../delta/DeltaSourceBase.md#getNextOffsetFromPreviousOffset) based on [isStartingVersion](../delta/DeltaSourceOffset.md#isStartingVersion) (of the [previous offset](../delta/DeltaSourceOffset.md))
    * `DeltaSource` when [getBatch](../delta/DeltaSource.md#getBatch) with `startOffsetOption` specified and based on the [isStartingVersion](../delta/DeltaSourceOffset.md#isStartingVersion) (of the [start offset](../delta/DeltaSourceOffset.md))

### <span id="filterAndIndexDeltaLogs"> filterAndIndexDeltaLogs

```scala
filterAndIndexDeltaLogs(
  startVersion: Long): Iterator[(Long, IndexedChangeFileSeq)]
```

`filterAndIndexDeltaLogs` requests the [DeltaLog](../delta/DeltaSource.md#deltaLog) to [get the changes](../DeltaLog.md#getChanges) at the given `startVersion` version and on (`Iterator[(Long, Seq[Action])]`).

`filterAndIndexDeltaLogs` uses [failOnDataLoss](../delta/options.md#failOnDataLoss) option to get the changes.

`filterAndIndexDeltaLogs` [filterCDCActions](#filterCDCActions) (across the actions across all the versions) and converts the [AddFile](../AddFile.md)s, [AddCDCFile](../AddCDCFile.md)s and [RemoveFile](../RemoveFile.md)s to `IndexedFile`s.

In the end, for every version, `filterAndIndexDeltaLogs` creates a [IndexedChangeFileSeq](IndexedChangeFileSeq.md) with the `IndexedFile`s (and the [isInitialSnapshot](IndexedChangeFileSeq.md#isInitialSnapshot) flag off).

### <span id="filterCDCActions"> filterCDCActions

```scala
filterCDCActions(
  actions: Seq[Action],
  version: Long): Seq[FileAction]
```

!!! note
    `version` argument is ignored.

`filterCDCActions` collects the [AddCDCFile](../AddCDCFile.md) actions from the given `actions` (if there are any).

Otherwise, `filterCDCActions` collects [AddFile](../AddFile.md)s and [RemoveFile](../RemoveFile.md)s with `dataChange` enabled.