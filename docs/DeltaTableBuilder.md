# DeltaTableBuilder

`DeltaTableBuilder` is a [builder interface](#operators) to create [DeltaTable](DeltaTable.md)s programmatically.

`DeltaTableBuilder` is [created](#creating-instance) using the following [DeltaTable](DeltaTable.md) utilities:

* [DeltaTable.create](DeltaTable.md#create)
* [DeltaTable.createIfNotExists](DeltaTable.md#createIfNotExists)
* [DeltaTable.replace](DeltaTable.md#replace)
* [DeltaTable.createOrReplace](DeltaTable.md#createOrReplace)

In the end, `DeltaTableBuilder` is supposed to be [executed](#execute) to take action.

## io.delta.tables Package

`DeltaTableBuilder` belongs to `io.delta.tables` package.

```scala
import io.delta.tables.DeltaTableBuilder
```

## Creating Instance

`DeltaTableBuilder` takes the following to be created:

* <span id="spark"> `SparkSession` ([Spark SQL]({{ book.spark_sql }}/SparkSession))
* <span id="builderOption"> `DeltaTableBuilderOptions`

## Operators

### <span id="addColumn"> addColumn

```scala
addColumn(
  colName: String,
  dataType: DataType): DeltaTableBuilder
addColumn(
  colName: String,
  dataType: DataType,
  nullable: Boolean): DeltaTableBuilder
addColumn(
  colName: String,
  dataType: String): DeltaTableBuilder
addColumn(
  colName: String,
  dataType: String,
  nullable: Boolean): DeltaTableBuilder
addColumn(
  col: StructField): DeltaTableBuilder
addColumns(
  cols: StructType): DeltaTableBuilder
```

### <span id="comment"> comment

```scala
comment(
  comment: String): DeltaTableBuilder
```

### <span id="execute"> execute

```scala
execute(): DeltaTable
```

Creates a [DeltaTable](DeltaTable.md)

### <span id="location"> location

```scala
location(
  location: String): DeltaTableBuilder
```

### <span id="partitionedBy"> partitionedBy

```scala
partitionedBy(
  colNames: String*): DeltaTableBuilder
```

### <span id="property"> property

```scala
property(
  key: String,
  value: String): DeltaTableBuilder
```

### <span id="tableName"> tableName

```scala
tableName(
  identifier: String): DeltaTableBuilder
```