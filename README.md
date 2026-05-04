# Feature States

Generic feature flag system supporting three environments (Editor, Debug, Production) for Unity projects.

# Installation

## Via Git URL

Open **Window → Package Manager**, click **+**, and choose **Add package from git URL**.

To install the latest version:
```
https://github.com/Warlander/feature-states.git
```

To install a specific release, append the tag:
```
https://github.com/Warlander/feature-states.git#2.0.5
```

## Via Registry Browser

If you have [Registry Browser](https://github.com/Warlander/registry-browser) in the project, make sure you have tracked registry added:

```
Scope Prefix: com.warlogic
Registry URL: https://upm.maciejcyranowicz.com
```

Then open **Window > Warlander > Registry Browser** and add `com.warlogic.featurestates` to the project.

## Via Scoped Registry

Add the Warlogic registry to your `Packages/manifest.json`:

```json
{
  "scopedRegistries": [
    {
      "name": "Warlogic",
      "url": "https://upm.maciejcyranowicz.com",
      "scopes": ["com.warlogic"]
    }
  ],
  "dependencies": {
    "com.warlogic.featurestates": "2.0.5"
  }
}
```

Then open **Window > Package Manager** and look for `com.warlogic.featurestates`.

# Setup

1. Create a `Feature` enum somewhere in the project. We recommend assigning explicit integer values starting at 1 rather than 0. This avoids accidental confusion with default-initialized values and makes it safer to add or remove values later. Example:
```csharp
public enum Feature
{
    FeatureA = 1,
    FeatureB = 2,
    FeatureC = 3
}
```

2. Create a concrete ScriptableObject subclass that extends `FeatureStateRepository<Feature>`. `FeatureStateRepository<T>` inherits from `FeatureStateRepositoryBase`, which is what the custom inspector targets. Include a `[CreateAssetMenu]` attribute so Unity can create the asset from the menu. Example:
```csharp
[CreateAssetMenu(fileName = "FeatureStates", menuName = "MyProject/Feature States")]
public class MyFeatureStateRepository : FeatureStateRepository<Feature> { }
```

At runtime, two interfaces work together:
- `IFeatureStateRepository<TFeature>` — provides raw per-environment toggle values (Production, Debug, Editor).
- `IFeatureStateRetriever<TFeature>` — resolves the current runtime environment and returns the effective boolean for a feature.

3. Create a `FeatureStates` asset anywhere inside a `Resources/` folder using the menu item you declared above. Open the asset in the Inspector — the custom inspector automatically syncs all enum values as rows. Configure the Production, Debug, and Editor toggles for each feature.

4. Obtain an `IFeatureStateRetriever<Feature>` at runtime. Manual wiring (no DI framework required):
```csharp
IFeatureStateRepository<Feature> repository = new ResourceFeatureStateRepositoryRetriever<Feature>("FeatureStates").Get();
IFeatureStateRetriever<Feature> featureStateRetriever = new FeatureStateRetriever<Feature>(repository);
```
Example DI with VContainer:
```csharp
builder.RegisterInstance(new ResourceFeatureStateRepositoryRetriever<Feature>("FeatureStates").Get());
builder.Register<FeatureStateRetriever<Feature>>(Lifetime.Singleton).As<IFeatureStateRetriever<Feature>>();
```
> The string `"FeatureStates"` is the name of the ScriptableObject asset inside a `Resources/` folder (without extension).

# Usage

Inject `IFeatureStateRetriever<Feature>` and call `IsFeatureEnabled` with an enum value:
```csharp
public class MyClass
{
    private readonly IFeatureStateRetriever<Feature> _featureStateRetriever;

    public MyClass(IFeatureStateRetriever<Feature> featureStateRetriever)
    {
        _featureStateRetriever = featureStateRetriever;
    }

    public void DoSomething()
    {
        if (!_featureStateRetriever.IsFeatureEnabled(Feature.FeatureA))
            return;
        // feature-specific logic
    }
}
```

# Environments

The system supports three environments, resolved automatically at runtime:

| Environment    | When active |
|----------------|-------------|
| **Editor**     | Running inside the Unity Editor |
| **Debug**      | Development builds (`Debug.isDebugBuild == true`) |
| **Production** | Release builds |

Configure which features are enabled per environment in the Inspector on your `FeatureStates` ScriptableObject.

# Edge Cases

- **Unknown feature** — If you query a feature that has no entry in the repository, the system logs a warning and returns `false` for all environments.
- **Missing asset** — If `ResourceFeatureStateRepositoryRetriever` cannot find the ScriptableObject in `Resources`, `Get()` returns `null`. You should validate the repository reference before passing it to `FeatureStateRetriever`.

# Adding or Removing Features

Add or remove values from your `Feature` enum. The next time you open the `FeatureStates` asset in the Inspector, the custom editor automatically adds rows for new values and removes rows for deleted values, preserving the existing toggle settings for unchanged values.
