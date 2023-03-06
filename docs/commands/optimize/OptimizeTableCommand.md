# OptimizeTableCommand

`OptimizeTableCommand` is a `LeafRunnableCommand` ([Spark SQL]({{ book.spark_sql }}/logical-operators/LeafRunnableCommand)) for the following high-level operators:

* [OPTIMIZE](../../sql/index.md#OPTIMIZE) SQL command
* [DeltaOptimizeBuilder.executeCompaction](../../DeltaOptimizeBuilder.md#executeCompaction)
* [DeltaOptimizeBuilder.executeZOrderBy](../../DeltaOptimizeBuilder.md#executeZOrderBy)

`OptimizeTableCommand` is an [OptimizeTableCommandBase](OptimizeTableCommandBase.md).

Once [executed](#run), `OptimizeTableCommand` returns a `DataFrame` with the following:

* [dataPath](../../DeltaLog.md#dataPath)
* [OptimizeMetrics](OptimizeMetrics.md)

## Creating Instance

`OptimizeTableCommand` takes the following to be created:

* <span id="path"> Table Path
* <span id="tableId"> `TableIdentifier`
* <span id="userPartitionPredicates"> Partition Predicates
* <span id="options"> Options
* <span id="zOrderBy"> `zOrderBy` attributes (aka _interleaveBy_ attributes)

`OptimizeTableCommand` is created when:

* `DeltaSqlAstBuilder` is requested to [parse OPTIMIZE SQL statement](../../sql/DeltaSqlAstBuilder.md#visitOptimizeTable)
* `DeltaOptimizeBuilder` is requested to [execute](../../DeltaOptimizeBuilder.md#execute)

## <span id="run"> Executing Command

??? note "Signature"
    ```scala
    run(
      sparkSession: SparkSession): Seq[Row]
    ```

    `run` is part of the `RunnableCommand` ([Spark SQL]({{ book.spark_sql }}/logical-operators/RunnableCommand#run)) abstraction.

`run` [gets the DeltaLog](../DeltaCommand.md#getDeltaLog) of the delta table (by the given [path](#path) or [TableIdentifier](#tableId)).

`run` [validates the zOrderBy columns](OptimizeTableCommandBase.md#validateZorderByColumns) (that may throw `DeltaIllegalArgumentException` or `DeltaAnalysisException` exceptions and so break the command execution).

In the end, `run` creates an [OptimizeExecutor](OptimizeExecutor.md) that is in turn requested to [optimize](OptimizeExecutor.md#optimize).
