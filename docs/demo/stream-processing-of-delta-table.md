---
hide:
  - navigation
---

# Demo: Stream Processing of Delta Table

```text
/*
spark-shell \
  --packages io.delta:delta-core_2.12:{{ delta.version }} \
  --conf spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension \
  --conf spark.databricks.delta.snapshotPartitions=1
*/
assert(spark.version.matches("2.4.[2-4]"), "Delta Lake supports Spark 2.4.2+")

import org.apache.spark.sql.SparkSession
assert(spark.isInstanceOf[SparkSession])

val deltaTableDir = "/tmp/delta/users"
val checkpointLocation = "/tmp/checkpointLocation"

// Initialize the delta table
// - No data
// - Schema only
case class Person(id: Long, name: String, city: String)
spark.emptyDataset[Person].write.format("delta").save(deltaTableDir)

import org.apache.spark.sql.streaming.Trigger
import scala.concurrent.duration._
val sq = spark
  .readStream
  .format("delta")
  .option("maxFilesPerTrigger", 1) // Maximum number of files to scan per micro-batch
  .load(deltaTableDir)
  .writeStream
  .format("console")
  .option("truncate", false)
  .option("checkpointLocation", checkpointLocation)
  .trigger(Trigger.ProcessingTime(10.seconds)) // Useful for debugging
  .start

// The streaming query over delta table
// should display the 0th version as Batch 0
-------------------------------------------
Batch: 0
-------------------------------------------
+---+----+----+
|id |name|city|
+---+----+----+
+---+----+----+

// Let's write to the delta table
val users = Seq(
    Person(0, "Jacek", "Warsaw"),
    Person(1, "Agata", "Warsaw"),
    Person(2, "Jacek", "Paris"),
    Person(3, "Domas", "Vilnius")).toDF

// More partitions are more file added
// And per maxFilesPerTrigger as 1 file addition per micro-batch
// You should see more micro-batches
scala> println(users.rdd.getNumPartitions)
4

// Change the default SaveMode.ErrorIfExists to more meaningful save mode
import org.apache.spark.sql.SaveMode
users
  .write
  .format("delta")
  .mode(SaveMode.Append) // Appending rows
  .save(deltaTableDir)

// Immediately after the above write finishes
// New batches should be printed out to the console
// Per the number of partitions of users dataset
// And per maxFilesPerTrigger as 1 file addition
// You should see as many micro-batches as files
-------------------------------------------
Batch: 1
-------------------------------------------
+---+-----+------+
|id |name |city  |
+---+-----+------+
|0  |Jacek|Warsaw|
+---+-----+------+

-------------------------------------------
Batch: 2
-------------------------------------------
+---+-----+------+
|id |name |city  |
+---+-----+------+
|1  |Agata|Warsaw|
+---+-----+------+

-------------------------------------------
Batch: 3
-------------------------------------------
+---+-----+-----+
|id |name |city |
+---+-----+-----+
|2  |Jacek|Paris|
+---+-----+-----+

-------------------------------------------
Batch: 4
-------------------------------------------
+---+-----+-------+
|id |name |city   |
+---+-----+-------+
|3  |Domas|Vilnius|
+---+-----+-------+
```
