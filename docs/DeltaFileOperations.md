# DeltaFileOperations Utilities

=== [[recursiveListDirs]] recursiveListDirs

[source, scala]
----
recursiveListDirs(
  spark: SparkSession,
  subDirs: Seq[String],
  hadoopConf: Broadcast[SerializableConfiguration],
  hiddenFileNameFilter: String => Boolean = defaultHiddenFileFilter,
  fileListingParallelism: Option[Int] = None): Dataset[SerializableFileStatus]
----

recursiveListDirs...FIXME

recursiveListDirs is used when:

* ManualListingFileManifest (of ConvertToDeltaCommandBase) is requested to doList

* VacuumCommand utility is used to VacuumCommand.md#gc[gc]

=== [[tryDeleteNonRecursive]] tryDeleteNonRecursive

[source,scala]
----
tryDeleteNonRecursive(
  fs: FileSystem,
  path: Path,
  tries: Int = 3): Boolean
----

tryDeleteNonRecursive...FIXME

tryDeleteNonRecursive is used when VacuumCommandImpl is requested to VacuumCommandImpl.md#delete[delete]

== [[internal-methods]] Internal Methods

=== [[recurseDirectories]] recurseDirectories

[source,scala]
----
recurseDirectories(
  logStore: LogStore,
  filesAndDirs: Iterator[SerializableFileStatus],
  hiddenFileNameFilter: String => Boolean): Iterator[SerializableFileStatus]
----

recurseDirectories...FIXME

recurseDirectories is used when DeltaFileOperations is requested to <<recursiveListDirs, recursiveListDirs>> and <<listUsingLogStore, listUsingLogStore>>.

=== [[listUsingLogStore]] listUsingLogStore

[source,scala]
----
listUsingLogStore(
  logStore: LogStore,
  subDirs: Iterator[String],
  recurse: Boolean,
  hiddenFileNameFilter: String => Boolean): Iterator[SerializableFileStatus]
----

listUsingLogStore...FIXME

listUsingLogStore is used when DeltaFileOperations is requested to <<recursiveListDirs, recursiveListDirs>> and  <<recurseDirectories, recurseDirectories>>.

=== [[isThrottlingError]] isThrottlingError

[source,scala]
----
isThrottlingError(
  t: Throwable): Boolean
----

isThrottlingError returns `true` when the Throwable contains `slow down`.

isThrottlingError is used when DeltaFileOperations is requested to <<listUsingLogStore, listUsingLogStore>> and <<tryDeleteNonRecursive, tryDeleteNonRecursive>>.
