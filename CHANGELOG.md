# Changelog
All notable changes to this package will be documented in this file.

The format is based on [Keep a Changelog](http://keepachangelog.com/en/1.0.0/)
and this project adheres to [Semantic Versioning](http://semver.org/spec/v2.0.0.html).

## [2.0.5] - 2026-05-01

### Added
- README "Edge Cases" section covering unknown-feature warning behavior and `null` return when a Resources asset is missing.

### Changed
- README enum guidance softened from a hard rule ("0 is reserved") to a recommendation.
- README setup now explains `FeatureStateRepositoryBase` and the split between `IFeatureStateRepository<T>` (raw data) and `IFeatureStateRetriever<T>` (environment resolution).

### Fixed
- Package test suite now compiles and passes under Unity 6.

## [2.0.4] - 2026-04-08

### Changed
- Made packages signed.

## [2.0.3] - 2026-04-08

### Added
- Added license metafile.

## [2.0.2] - 2026-04-08

### Added
- Added metadata.

## [2.0.1] - 2026-04-07

### Added
- Added reference to GitHub repository.

## [2.0.0] - 2026-03-26

### Changed
- Feature identifiers changed from strings to a user-defined enum type.
  All API methods and interfaces are now generic (`IFeatureStateRetriever<TFeature>`,
  `IFeatureStateRepository<TFeature>`, `FeatureStateRetriever<TFeature>`).
- `FeatureStateRepository` is now an abstract generic base class; clients must provide
  a concrete non-generic subclass with `[CreateAssetMenu]`.
- `FeatureState` struct renamed to `FeatureStateEntry<TFeature>`.
- `_featureNamesSource` MonoScript field removed from the ScriptableObject inspector;
  feature names are now auto-derived from the enum type.

### Removed
- String-based `IFeatureStateRetriever.IsFeatureEnabled(string)` API.
- String-based `IFeatureStateRepository` interface methods.
- Non-generic `FeatureStateRepository`, `FeatureStateRetriever`,
  `ResourceFeatureStateRepositoryRetriever` classes.

## [1.0.0] - 2026-03-22

### This is the first release of *\<Feature States\>*.
