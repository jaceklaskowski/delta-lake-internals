# DeltaErrorsBase

## convertToDeltaRowTrackingEnabledWithoutStatsCollection { #convertToDeltaRowTrackingEnabledWithoutStatsCollection }

```scala
convertToDeltaRowTrackingEnabledWithoutStatsCollection: Throwable
```

`convertToDeltaRowTrackingEnabledWithoutStatsCollection` creates a `DeltaIllegalStateException` with the following:

* `errorClass`: `DELTA_CONVERT_TO_DELTA_ROW_TRACKING_WITHOUT_STATS`
* `messageParameters`:
    * [spark.databricks.delta.stats.collect](configuration-properties/index.md#DELTA_COLLECT_STATS)
    * The [default session config key](table-features/TableFeatureProtocolUtils.md#defaultPropertyKey) of [RowTrackingFeature](row-tracking/RowTrackingFeature.md)
    * The [default table property key](DeltaConfig.md#defaultTablePropertyKey) of [delta.enableRowTracking](DeltaConfigs.md#ROW_TRACKING_ENABLED)

---

`convertToDeltaRowTrackingEnabledWithoutStatsCollection` is used when:

* `RowId` is requested to [checkStatsCollectedIfRowTrackingSupported](row-tracking/RowId.md#checkStatsCollectedIfRowTrackingSupported)

## modifyAppendOnlyTableException { #modifyAppendOnlyTableException }

```scala
modifyAppendOnlyTableException(
  tableName: String): Throwable
```

`modifyAppendOnlyTableException` creates a `DeltaUnsupportedOperationException` with the following:

* `errorClass`: `DELTA_CANNOT_MODIFY_APPEND_ONLY`
* `messageParameters`:
    * The given `tableName`
    * [delta.appendOnly](DeltaConfigs.md#IS_APPEND_ONLY)

---

`modifyAppendOnlyTableException` is used when:

* `DeltaLog` is requested to [assertRemovable](DeltaLog.md#assertRemovable)