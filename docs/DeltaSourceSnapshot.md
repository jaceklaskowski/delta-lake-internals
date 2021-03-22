# DeltaSourceSnapshot

[[SnapshotIterator]][[StateCache]]
`DeltaSourceSnapshot` is a <<SnapshotIterator.md#, SnapshotIterator>> with <<StateCache.md#, StateCache>>

`DeltaSourceSnapshot` is <<creating-instance, created>> when `DeltaSource` is requested for the <<DeltaSource.md#getSnapshotAt, snapshot at a given version>>.

[[version]]
When <<creating-instance, created>>, `DeltaSourceSnapshot` requests the <<snapshot, Snapshot>> for the <<Snapshot.md#version, version>> that it uses for the <<initialFiles, initialFiles>> (a new column and the name of the cached RDD).

== [[creating-instance]] Creating DeltaSourceSnapshot Instance

`DeltaSourceSnapshot` takes the following to be created:

* [[spark]] `SparkSession`
* [[snapshot]] <<Snapshot.md#, Snapshot>>
* [[filters]] Filter expressions (`Seq[Expression]`)

== [[initialFiles]] Initial Files (Indexed AddFiles) -- `initialFiles` Method

[source, scala]
----
initialFiles: Dataset[IndexedFile]
----

`initialFiles` requests the <<snapshot, Snapshot>> for <<Snapshot.md#allFiles, all files>> (`Dataset[AddFile]`) and sorts them by <<AddFile.md#modificationTime, modificationTime>> and <<AddFile.md#path, path>> in ascending order.

`initialFiles` zips the <<AddFile.md#, AddFiles>> with indices (using `RDD.zipWithIndex` operator), adds two new columns with the <<version, version>> and `isLast` as `false`, and finally creates a `Dataset[IndexedFile]`.

In the end, `initialFiles` <<StateCache.md#cacheDS, caches the dataset>> with the following name (with the <<version, version>> and the <<Snapshot.md#redactedPath, redactedPath>> of the <<snapshot, Snapshot>>)

```
Delta Source Snapshot #[version] - [redactedPath]
```

NOTE: `initialFiles` is used exclusively when `SnapshotIterator` is requested for a <<SnapshotIterator.md#iterator, iterator (of IndexedFiles)>>.
