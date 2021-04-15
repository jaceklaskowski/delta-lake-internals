# DeltaCommand

**DeltaCommand** is a marker interface for [commands](#implementations) to work with data in delta tables.

## Implementations

* [AlterDeltaTableCommand](AlterDeltaTableCommand.md)
* [ConvertToDeltaCommand](ConvertToDeltaCommand.md)
* [DeleteCommand](DeleteCommand.md)
* [MergeIntoCommand](MergeIntoCommand.md)
* [UpdateCommand](UpdateCommand.md)
* [VacuumCommandImpl](vacuum/VacuumCommandImpl.md)
* [WriteIntoDelta](WriteIntoDelta.md)

## <span id="parsePartitionPredicates"> parsePartitionPredicates Method

```scala
parsePartitionPredicates(
  spark: SparkSession,
  predicate: String): Seq[Expression]
```

`parsePartitionPredicates`...FIXME

`parsePartitionPredicates` is used when...FIXME

## <span id="verifyPartitionPredicates"> verifyPartitionPredicates Method

```scala
verifyPartitionPredicates(
  spark: SparkSession,
  partitionColumns: Seq[String],
  predicates: Seq[Expression]): Unit
```

`verifyPartitionPredicates`...FIXME

`verifyPartitionPredicates` is used when...FIXME

## <span id="generateCandidateFileMap"> generateCandidateFileMap Method

```scala
generateCandidateFileMap(
  basePath: Path,
  candidateFiles: Seq[AddFile]): Map[String, AddFile]
```

`generateCandidateFileMap`...FIXME

`generateCandidateFileMap` is used when...FIXME

## <span id="removeFilesFromPaths"> removeFilesFromPaths Method

```scala
removeFilesFromPaths(
  deltaLog: DeltaLog,
  nameToAddFileMap: Map[String, AddFile],
  filesToRewrite: Seq[String],
  operationTimestamp: Long): Seq[RemoveFile]
```

`removeFilesFromPaths`...FIXME

`removeFilesFromPaths` is used when [DeleteCommand](DeleteCommand.md) and [UpdateCommand](UpdateCommand.md) commands are executed.

## <span id="buildBaseRelation"> Creating HadoopFsRelation (with TahoeBatchFileIndex)

```scala
buildBaseRelation(
  spark: SparkSession,
  txn: OptimisticTransaction,
  actionType: String,
  rootPath: Path,
  inputLeafFiles: Seq[String],
  nameToAddFileMap: Map[String, AddFile]): HadoopFsRelation
```

<span id="buildBaseRelation-scannedFiles">
`buildBaseRelation` converts the given `inputLeafFiles` to [AddFiles](#getTouchedFile) (with the given `rootPath` and `nameToAddFileMap`).

`buildBaseRelation` creates a [TahoeBatchFileIndex](../TahoeBatchFileIndex.md) for the `actionType`, the [AddFiles](#buildBaseRelation-scannedFiles) and the `rootPath`.

In the end, `buildBaseRelation` creates a `HadoopFsRelation` with the `TahoeBatchFileIndex` (and the other properties based on the [metadata](../OptimisticTransactionImpl.md#metadata) of the given [OptimisticTransaction](../OptimisticTransaction.md)).

!!! note
    Learn more on [HadoopFsRelation]({{ book.spark_sql }}/HadoopFsRelation/) in [The Internals of Spark SQL]({{ book.spark_sql }}/) online book.

`buildBaseRelation` is used when [DeleteCommand](DeleteCommand.md) and [UpdateCommand](UpdateCommand.md) commands are executed (with `delete` and `update` action types, respectively).

## <span id="getTouchedFile"> getTouchedFile Method

```scala
getTouchedFile(
  basePath: Path,
  filePath: String,
  nameToAddFileMap: Map[String, AddFile]): AddFile
```

`getTouchedFile`...FIXME

`getTouchedFile` is used when:

* `DeltaCommand` is requested to [removeFilesFromPaths](#removeFilesFromPaths) and [create a HadoopFsRelation](#buildBaseRelation) (for [DeleteCommand](DeleteCommand.md) and [UpdateCommand](UpdateCommand.md) commands)

* [MergeIntoCommand](MergeIntoCommand.md) is executed

## <span id="isCatalogTable"> isCatalogTable Method

```scala
isCatalogTable(
  analyzer: Analyzer,
  tableIdent: TableIdentifier): Boolean
```

`isCatalogTable`...FIXME

`isCatalogTable` is used when...FIXME

## <span id="isPathIdentifier"> isPathIdentifier Method

```scala
isPathIdentifier(
  tableIdent: TableIdentifier): Boolean
```

`isPathIdentifier`...FIXME

`isPathIdentifier` is used when...FIXME

## <span id="commitLarge"> commitLarge

```scala
commitLarge(
  spark: SparkSession,
  txn: OptimisticTransaction,
  actions: Iterator[Action],
  op: DeltaOperations.Operation,
  context: Map[String, String],
  metrics: Map[String, String]): Long
```

`commitLarge`...FIXME

`commitLarge` is used when:

* [ConvertToDeltaCommand](ConvertToDeltaCommand.md) command is executed
