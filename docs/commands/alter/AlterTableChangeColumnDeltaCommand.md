# AlterTableChangeColumnDeltaCommand

`AlterTableChangeColumnDeltaCommand` is an [AlterDeltaTableCommand](AlterDeltaTableCommand.md) to change (_alter_) the name, the comment, the nullability, the position and the data type of a [column](#columnName) (of a [DeltaTableV2](#table)).

`AlterTableChangeColumnDeltaCommand` is used when [DeltaCatalog.alterTable](../../DeltaCatalog.md#alterTable) is requested to execute `ColumnChange`s ([Spark SQL]({{ book.spark_sql }}/connector/catalog/TableChange/#ColumnChange)).

ColumnChange | SQL
-------------|----------
 `RenameColumn` ([Spark SQL]({{ book.spark_sql }}/connector/catalog/TableChange/#RenameColumn)) | `ALTER TABLE RENAME COLUMN` ([Spark SQL]({{ book.spark_sql }}/sql/AstBuilder#visitRenameTableColumn))
 `UpdateColumnComment` | `ALTER TABLE CHANGE COLUMN COMMENT`
 `UpdateColumnNullability` | `ALTER TABLE CHANGE COLUMN (SET | DROP) NOT NULL`
 `UpdateColumnPosition` | `ALTER TABLE CHANGE COLUMN (FIRST | AFTER)`
 `UpdateColumnType` | `ALTER TABLE CHANGE COLUMN TYPE`

## Creating Instance

`AlterTableChangeColumnDeltaCommand` takes the following to be created:

* <span id="table"> [DeltaTableV2](../../DeltaTableV2.md)
* <span id="columnPath"> Column Path
* <span id="columnName"> Column Name
* <span id="newColumn"> New Column (as [StructField]({{ book.spark_sql }}/types/StructField))
* <span id="colPosition"> `ColumnPosition` (optional)
* <span id="syncIdentity"> (_unused_) `syncIdentity` flag

`AlterTableChangeColumnDeltaCommand` is created when:

* `DeltaCatalog` is requested to [alter a table](../../DeltaCatalog.md#alterTable)

## <span id="run"> Executing Command

```scala
run(
  sparkSession: SparkSession): Seq[Row]
```

`run` [starts a transaction](AlterDeltaTableCommand.md#startTransaction).

`run`...FIXME

`run` transforms the current schema to a new one based on the column changes (given by [newColumn](#newColumn)). As part of the changes, for every column, `run` checks whether [column mapping](../../column-mapping/DeltaColumnMappingBase.md#getPhysicalName) is used to determine the column name.

`run`...FIXME

### <span id="run-update"> Updating Metadata

In the end, `run` requests the `OptimisticTransaction` to [update the Metadata](../../OptimisticTransactionImpl.md#updateMetadata) (with the new `newMetadata`) and [commit](../../OptimisticTransactionImpl.md#commit) (with no [Action](../../Action.md)s and [ChangeColumn](../../Operation.md#ChangeColumn) operation).

---

`run` is part of the `RunnableCommand` ([Spark SQL]({{ book.spark_sql }}/logical-operators/RunnableCommand/#run)) abstraction.

## <span id="LeafRunnableCommand"> LeafRunnableCommand

`AlterTableChangeColumnDeltaCommand` is a `LeafRunnableCommand` ([Spark SQL]({{ book.spark_sql }}/logical-operators/LeafRunnableCommand)).

## <span id="IgnoreCachedData"> IgnoreCachedData

`AlterTableChangeColumnDeltaCommand` is a `IgnoreCachedData` ([Spark SQL]({{ book.spark_sql }}/logical-operators/IgnoreCachedData)).