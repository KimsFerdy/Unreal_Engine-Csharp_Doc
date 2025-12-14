# Variables and Attributes

So guys, This is the first page that you need to kinda master, it is the foundation of everything you'll write in UnrealSharp.\
Variables in UnrealSharp are standard C# at their core, but to be **production-ready** and fully integrated with Unreal Engine (editor exposure, Blueprint access, replication, garbage collection, serialization, components, etc.), you must understand how UnrealSharp maps C# declarations to UE’s reflection system using **attributes** and **specific types**.

The goal here is to show you exactly how to declare variables that are:

* Visible and editable in the Details panel
* Accessible from Blueprints
* Safe with UE’s garbage collector
* Correctly serialized when saving/loading
* Properly attached in actor hierarchies
* Performant and idiomatic in a real game (thanks to the small project that helped me write all this documentation [Sample Project](https://github.com/UnrealSharp/UnrealSharp-SampleDefenseGame))

We’ll cover every common case with real, working examples taken or adapted from [Sample Project](https://github.com/UnrealSharp/UnrealSharp-SampleDefenseGame)

## 1. Basic Primitive Types

These map directly to UE equivalents and are safe to use anywhere.

```csharp
[UClass]
public class AMyActor : AActor
{
    [UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite, Category = "Stats")]
    public int Health { get; set; } = 100;

    [UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
    public float Damage { get; set; } = 25.0f;

    [UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
    public bool bIsInvulnerable { get; set; } = false;

    [UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
    public byte TeamId { get; set; } = 0;
}
```

**Best Practices**

* Always use auto-properties (`{ get; set; }`) when exposing to UE – UnrealSharp generates the necessary reflection data.
* Use `float` for gameplay values (UE’s default). Use `double` only for temporary high-precision math.
* Prefix booleans with `b` (UE convention) for clarity.

## 2. Unreal-Specific Core Types

These are the structs you will use constantly.

```csharp
using UnrealSharp.CoreUObject;

[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
public FVector SpawnLocation { get; set; }

[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
public FRotator SpawnRotation { get; set; }

[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
public FTransform SpawnTransform { get; set; }

[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
public FLinearColor TeamColor { get; set; }

[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
public FName ActorName { get; set; }

[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
public FText DisplayText { get; set; }
```

**Warning on some Variables** Sometimes you'll be seeing something like :

```
Assembly processing failed (Property initializer for UProperty PARAMETERR_NAME is not a supported constant type)
```

cause it comes from UnrealSharp's source generator.\
UnrealSharp only allows **compile-time constants** as default values for `[UProperty]`.\
Primitive types (`int`, `float`, `bool`, `enum`, `string` literals, etc.) are fine, but **structs** like `FVector`, `FRotator`, `FQuat`, `FTransform`, etc. are **not** supported as initializers, even if you write `= FVector.Zero` or `= new FVector(0,0,0)`.

### Why it fails

* `FVector.Zero` is a `static readonly` field → not a compile-time constant.
* `new FVector(0,0,0)` is an object allocation → also not a constant.
* The source generator needs to emit the default value into the reflected metadata at compile time, so only true constants are allowed.

### Correct way in UnrealSharp

1.  **Remove the initializer completely**\
    Struct properties default to their zero value anyway (`FVector` → (0,0,0), `FRotator` → (0,0,0), etc.).

    ```csharp
    [UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
    public FVector SpawnLocation { get; set; }   // ← no "= ..."

    [UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
    public FRotator SpawnRotation { get; set; } // ← no "= ..."
    ```
2.  **If you need a non-zero default that is visible in the editor**, set it in `ConstructionScript` (for Actors) or in a `[UFunction(BlueprintEvent)]` that the Blueprint subclass can override.\
    Example in an Actor class:

    ```csharp
    public override void ConstructionScript()
    {
        base.ConstructionScript();

        // These values will appear as defaults in the Details panel
        SpawnLocation = new FVector(1000, 0, 200);
        SpawnRotation = new FRotator(0, 45, 0);
    }
    ```

    Or, if the class is meant to be subclassed in Blueprint:

    ```csharp
    [UFunction(FunctionFlags.BlueprintEvent)]
    protected virtual void SetupDefaults()
    {
        SpawnLocation = new FVector(1000, 0, 200);
        SpawnRotation = new FRotator(0, 45, 0);
    }

    protected override void BeginPlay()
    {
        base.BeginPlay();
        SetupDefaults(); // Blueprint can override this
    }
    ```
3.  **Alternative: expose simple primitive properties and compose the vector in code**\
    If you only need a few common defaults, you can do:

    ```csharp
    [UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
    public float SpawnX { get; set; } = 1000f;

    [UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
    public float SpawnY { get; set; } = 0f;

    [UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
    public float SpawnZ { get; set; } = 200f;

    // Computed property (not reflected as UProperty)
    public FVector SpawnLocation => new FVector(SpawnX, SpawnY, SpawnZ);
    ```

    This gives designers editable fields while keeping the real `FVector` clean.

**Anyway**

* Allowed: `int MyInt { get; set; } = 100;`, `float MyFloat { get; set; } = 50f;`, `bool MyBool { get; set; } = true;`, enums, string literals.
* Not allowed: `FVector`, `FRotator`, `FTransform`, `FLinearColor`, any `new ...` or static readonly fields.
* Recommended pattern: remove initializer → use `ConstructionScript` or a BlueprintEvent to set non-zero defaults.

## 3. Strings

UnrealSharp maps `string` → `FString` automatically in most contexts, but for full editor support use `FString` explicitly.

```csharp
[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
public string HelloString { get; set; }
```

## 4. Enums

```csharp
[UEnum]
public enum EChoices : byte
{
    First,
    Second,
    Third
}

// Usage
[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
public EChoices ChoicesList { get; set; }
```

* Use `byte` as the underlying type (UE convention).
* Enum will appear as a dropdown in the editor.

### **Warning:**

Enum should always be outside of Classes, in Native C# we could place Enum wherever the place wanted, but here, always place them "OUTSIDE" a class

## 5. Collections (TArray, TMap, TSet)

Use UE collections for anything exposed to Blueprints or serialized.

```csharp
[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
public TArray<int> Scores { get; set; }

[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
public TArray<AActor> Targets { get; set; }

[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
public TMap<FName, float> StatModifiers { get; set; }

// Private C# collections for internal logic (faster, no reflection overhead)
private List<C# Class Here> selectedUnits = new();
private HashSet<AActor> processedActors = new();
```

**Real example from** [**Sample Project**](https://github.com/UnrealSharp/UnrealSharp-SampleDefenseGame)

```csharp
[UProperty(PropertyFlags.EditAnywhere)]
public TArray<USoundWave> UnitConfirmationSounds { get; set; }
```

## 6. Class References (TSubclassOf)

Used everywhere for spawning/selecting classes in editor.

```csharp
[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
public TSubclassOf<C# Class Here> CSharpClassName { get; set; }

[UProperty(PropertyFlags.EditAnywhere)]
public TSubclassOf<UMyAITask> DefaultTask { get; set; } = typeof(UIdleTask);
```

* Appears as a class picker in the editor.
* Check validity with `UnitToBuild.Valid`

## 7. Object & Actor References

```csharp
[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite)]
public AActor TargetActor { get; set; }

[UProperty(PropertyFlags.BlueprintReadOnly)]
public AHQStructure HQ { get; private set; }

// Weak reference (does not prevent GC)
public TWeakObjectPtr<AActor> WeakTarget { get; set; }
```

**Important**: Strong references (normal UProperty) keep the object alive. Use weak references when you only want to observe.

## 8. Components – The Most Important Case

Components are the building blocks of actors.

```csharp
[UClass]
public class AStructure : AActor
{
    [UProperty(PropertyFlags.EditAnywhere, DefaultComponent = true, RootComponent = true)]
    public UStaticMeshComponent StaticMeshComponent { get; set; }

    [UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite, 
               DefaultComponent = true, AttachmentComponent = nameof(StaticMeshComponent))]
    public UWidgetComponent HealthBarComponent { get; set; }

    [UProperty(PropertyFlags.EditAnywhere, DefaultComponent = true, AttachmentComponent = nameof(StaticMeshComponent))]
    public UNiagaraComponent HarvestBeamComponent { get; private set; }
}
```

**Flags Explained**

* `DefaultComponent = true` → automatically created when actor is constructed
* `RootComponent = true` → this is the root (required for every actor)
* `AttachmentComponent = nameof(...)` → parent component name (string)

**Manual creation (when you need more control)**

```csharp
protected override void BeginPlay()
{
    base.BeginPlay();

    var meshComp = NewObject<UStaticMeshComponent>(this);
    meshComp.RegisterComponent();
    RootComponent = meshComp;
}
```

## 9. Property Flags Cheat Sheet

| Flag                                   | Meaning                              |
| -------------------------------------- | ------------------------------------ |
| EditAnywhere                           | Editable in editor (any context)     |
| EditDefaultsOnly                       | Only in archetype/defaults           |
| EditInstanceOnly                       | Only on placed instances             |
| BlueprintReadWrite                     | Accessible from Blueprints (get/set) |
| BlueprintReadOnly                      | Blueprint can read but not write     |
| Category = "MyCategory"                | Groups properties in Details panel   |
| meta = (DisplayName = "Health Points") | Custom editor label                  |

Example:

```csharp
[UProperty(PropertyFlags.EditAnywhere | PropertyFlags.BlueprintReadWrite, 
           Category = "Combat", meta = (DisplayName = "Max Health"))]
public float MaxHealth { get; set; } = 100.0f;
```

## 10. Function Attributes

```csharp
[UFunction(FunctionFlags.BlueprintCallable)]
public void Heal(float Amount) { ... }

[UFunction(FunctionFlags.BlueprintEvent)]
public virtual void OnRep_Health() { }   // BlueprintImplementableEvent

[UFunction(FunctionFlags.BlueprintNativeEvent)]
public virtual void OnTakeDamage(AActor Instigator) 
{ 
    // Default C# implementation
    // Blueprint can override
}
```

## Summary – Production Checklist

When declaring a variable in UnrealSharp:

1. Is it exposed to editor/Blueprint? → Add `[UProperty(...)]`
2. Is it a component? → Add `DefaultComponent/RootComponent/AttachmentComponent` as needed
3. Is it a collection exposed to Blueprint? → Use `TArray`/`TMap`
4. Is it a class reference? → Use `TSubclassOf<T>`
5. Is it internal only? → Plain C# field/property (no attribute)
6. Does it need a custom editor name/category? → Use `Category` and `meta`
