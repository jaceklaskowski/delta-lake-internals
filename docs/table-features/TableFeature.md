# TableFeature

`TableFeature` is an abstraction of [table features](#implementations).

`TableFeature` is `Serializable` ([Java]({{ java.api }}/java/io/Serializable.html)).

## Creating Instance

`TableFeature` takes the following to be created:

* [Name](#name)
* <span id="minReaderVersion"> Minimum reader protocol version required
* <span id="minWriterVersion"> Minimum writer protocol version required

!!! note "Abstract Class"
    `TableFeature` is an abstract class and cannot be created directly. It is created indirectly for the [concrete TableFeatures](#implementations).

### Name { #name }

```scala
name: String
```

The name of this table feature

The name can only be a combination of letters, `-`s (dashes) and `_`s (underscores).

---

Used when:

* `DeltaLog` is requested to [assertTableFeaturesMatchMetadata](../DeltaLog.md#assertTableFeaturesMatchMetadata) (for reporting purposes)
* `OptimisticTransactionImpl` is requested to [updateMetadataInternal](../OptimisticTransactionImpl.md#updateMetadataInternal) and [recordProtocolChanges](../OptimisticTransactionImpl.md#recordProtocolChanges)
* `TableFeature` is requested to [allSupportedFeaturesMap](TableFeature.md#allSupportedFeaturesMap)
* `Protocol` is requested to [forTableFeature](../Protocol.md#forTableFeature)
* `TableFeatureSupport` is requested to [withFeature](TableFeatureSupport.md#withFeature), [canUpgradeTo](TableFeatureSupport.md#canUpgradeTo), [isFeatureSupported](TableFeatureSupport.md#isFeatureSupported), [propertyKey](TableFeatureSupport.md#propertyKey), [defaultPropertyKey](TableFeatureSupport.md#defaultPropertyKey)
* `DescribeDeltaDetailCommand` is executed (and requested to [describeDeltaTable](../commands/describe-detail/DescribeDeltaDetailCommand.md#describeDeltaTable))

### Required Features { #requiredFeatures }

```scala
requiredFeatures: Set[TableFeature]
```

The required [TableFeature](TableFeature.md)s that this `TableFeature` depends on

Default: (empty)

See:

* [RowTrackingFeature](../row-tracking/RowTrackingFeature.md#requiredFeatures)

Used when:

* `Protocol` is requested to [getDependencyClosure](../Protocol.md#getDependencyClosure)
* `TableFeatureSupport` is requested to [withFeature](TableFeatureSupport.md#withFeature)

## Implementations

* [LegacyWriterFeature](LegacyWriterFeature.md)
* [WriterFeature](WriterFeature.md)

??? note "Sealed Abstract Class"
    `TableFeature` is a Scala **sealed abstract class** which means that all of the implementations are in the same compilation unit (a single file).

## allSupportedFeaturesMap { #allSupportedFeaturesMap }

```scala
allSupportedFeaturesMap: Map[String, TableFeature]
```

`allSupportedFeaturesMap` is a collection of [TableFeature](TableFeature.md)s by their lower-case [name](#name):

* [AppendOnlyTableFeature](../append-only-tables/AppendOnlyTableFeature.md)
* [ChangeDataFeedTableFeature](../change-data-feed/ChangeDataFeedTableFeature.md)
* [CheckConstraintsTableFeature](../check-constraints/CheckConstraintsTableFeature.md)
* [ClusteringTableFeature](../liquid-clustering/ClusteringTableFeature.md)
* [ColumnMappingTableFeature](../column-mapping/ColumnMappingTableFeature.md)
* [DeletionVectorsTableFeature](../deletion-vectors/DeletionVectorsTableFeature.md)
* [DomainMetadataTableFeature](DomainMetadataTableFeature.md)
* [GeneratedColumnsTableFeature](../generated-columns/GeneratedColumnsTableFeature.md)
* [IdentityColumnsTableFeature](../identity-columns/IdentityColumnsTableFeature.md)
* [InvariantsTableFeature](../column-invariants/InvariantsTableFeature.md)
* _others_

---

`allSupportedFeaturesMap` is used when:

* `TableFeature` is requested to [look up the table feature](#featureNameToFeature)
* `Action` is requested to [supportedProtocolVersion](../Action.md#supportedProtocolVersion)
* `Protocol` is requested to [extractAutomaticallyEnabledFeatures](../Protocol.md#extractAutomaticallyEnabledFeatures)
* `TableFeatureSupport` is requested for the [implicitlySupportedFeatures](TableFeatureSupport.md#implicitlySupportedFeatures)

## Looking Up Table Feature { #featureNameToFeature }

```scala
featureNameToFeature(
  featureName: String): Option[TableFeature]
```

`featureNameToFeature` tries to find the [TableFeature](TableFeature.md) by the given `featureName` in the [allSupportedFeaturesMap](#allSupportedFeaturesMap).

---

`featureNameToFeature` is used when:

* `DeltaLog` is requested to [assertTableFeaturesMatchMetadata](../DeltaLog.md#assertTableFeaturesMatchMetadata)
* `OptimisticTransactionImpl` is requested to [updateMetadataInternal](../OptimisticTransactionImpl.md#updateMetadataInternal)
* `Protocol` is requested to [extractAutomaticallyEnabledFeatures](../Protocol.md#extractAutomaticallyEnabledFeatures)
* `TableFeatureSupport` is requested for the [implicitlyAndExplicitlySupportedFeatures](TableFeatureSupport.md#implicitlyAndExplicitlySupportedFeatures)
* `TableFeatureProtocolUtils` is requested to [getSupportedFeaturesFromTableConfigs](TableFeatureProtocolUtils.md#getSupportedFeaturesFromTableConfigs)
