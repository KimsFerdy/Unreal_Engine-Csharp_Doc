# Materials

Materials (`UMaterial` and `UMaterialInterface`) are the heart of visual fidelity in Unreal Engine.\
In UnrealSharp, you load them like any `UObject`, create **dynamic instances** (`UMaterialInstanceDynamic`) for safe runtime parameter changes, and apply them to components. This enables everything from team color swaps and damage effects to day/night cycles and procedural variations—all in pure C#.

**Production Rules**

* Always use the full primary asset reference: `"/Game/Materials/M_MyMat.M_MyMat"` (copy from Content Browser → Copy Reference).
* Prefer `FSoftObjectPath` + `LoadSynchronous<T>` for reliable loading.
* Create `UMaterialInstanceDynamic` (MID) for runtime tweaks—never modify base `UMaterial`.
* Cache MIDs when animating in `Tick` (recreating every frame is wasteful).
* Parameter names are `FName` (case-sensitive, match Material Editor exactly).
* All examples below are fully compliant with current UnrealSharp (Dec 2025), use proper attributes, and are ready to test.

### 1. Load Base Material from Path & Apply to Mesh

Swap an entire material on a component (e.g., change a building's look).

```csharp
using UnrealSharp.CoreUObject;
using UnrealSharp.Engine;

[UClass]
public class AMaterialLoader : AActor
{
    protected override void BeginPlay()
    {
        base.BeginPlay();  
  
		UMaterial baseMat = StaticLoadObject<UMaterial>("/Game/Materials/M_MyBaseMat.M_MyBaseMat");  
		if (baseMat != null)  
		{  
		    PrintString(baseMat.ObjectName + " Loaded");  
		    UStaticMeshComponent meshComponent = GetComponentByClass<UStaticMeshComponent>();  
		    if (meshComponent)  
		    {  
		        meshComponent.SetMaterial(0, baseMat);  
		    }  
		}
    }
}
```

**Pro Note**: Works identically on `USkeletalMeshComponent`.

### 2. Create Dynamic Material Instance from Base

Non-destructive per-actor variations—ideal for unique props or characters.

```csharp
using UnrealSharp.Engine;

[UClass]
public class AMaterialInstancer : AActor
{
    protected override void BeginPlay()
    {
        base.BeginPlay();

        UMaterial baseMat = LoadObject<UMaterial>(null, "/Game/Materials/M_MyBaseMat.M_MyBaseMat");

        if (baseMat != null)
        {
            UMaterialInstanceDynamic mid = UMaterialInstanceDynamic.Create(baseMat, this);

            UPrimitiveComponent primComp = GetComponentByClass<UPrimitiveComponent>();
            if (primComp != null)
            {
                primComp.SetMaterial(0, mid);
                SystemLibrary.Log(ELogLevel.Display, "Created and applied dynamic material instance!");
            }
        }
    }
}
```

**Pro Note**: Outer (`this`) keeps the MID rooted and alive with the actor.

### 3. Set Scalar Parameter (e.g., Roughness)

Runtime surface changes—perfect for wear/tear or environmental effects.

```csharp
using UnrealSharp.Engine;

[UClass]
public class AMaterialScalarTweaker : AActor
{
    protected override void BeginPlay()
    {
        base.BeginPlay();

        UPrimitiveComponent primComp = GetComponentByClass<UPrimitiveComponent>();
        UMaterialInstanceDynamic mid = primComp.CreateDynamicMaterialInstance(0);

        if (mid != null)
        {
            mid.SetScalarParameterValue(new FName("Roughness"), 0.8f);
            SystemLibrary.Log(ELogLevel.Display, "Roughness set to 0.8 — surface now gritty!");
        }
    }
}
```

**Pro Note**: `CreateDynamicMaterialInstance(ElementIndex)` is a convenient shortcut that auto-creates MID from current material.

### 4. Set Vector Parameter (e.g., Emissive Color)

Dynamic glows, health indicators, or mood lighting.

```csharp
using UnrealSharp.CoreUObject;
using UnrealSharp.Engine;

[UClass]
public class AMaterialEmissiveTweaker : AActor
{
    protected override void BeginPlay()
    {
        base.BeginPlay();

        UPrimitiveComponent primComp = GetComponentByClass<UPrimitiveComponent>();
        UMaterialInstanceDynamic mid = primComp.CreateDynamicMaterialInstance(0);

        if (mid != null)
        {
            mid.SetVectorParameterValue(new FName("EmissiveColor"), FLinearColor.Red * 5.0f);
            SystemLibrary.Log(ELogLevel.Display, "Emissive glow activated — intense red!");
        }
    }
}
```

**Pro Note**: Multiply intensity >1 for HDR bloom.

### 5. Set Texture Parameter (Runtime Skin/Decal Swap)

Character customization, seasonal foliage, etc.

```csharp
using UnrealSharp.CoreUObject;
using UnrealSharp.Engine;

[UClass]
public class AMaterialTextureSwapper : AActor
{
    protected override void BeginPlay()
    {
        base.BeginPlay();

        UTexture2D newTex = LoadObject<UTexture2D>(null, "/Game/Textures/T_NewSkin.T_NewSkin");

        UPrimitiveComponent primComp = GetComponentByClass<UPrimitiveComponent>();
        UMaterialInstanceDynamic mid = primComp.CreateDynamicMaterialInstance(0);

        if (mid != null && newTex != null)
        {
            mid.SetTextureParameterValue(new FName("BaseColor"), newTex);
            SystemLibrary.Log(ELogLevel.Display, "Base color texture swapped — fresh look!");
        }
    }
}
```

**Pro Note**: Combine with soft references for async loading.

### 6. Copy Parameters Between Instances

Sync settings across multiple actors (team uniforms, global effects).

```csharp
using UnrealSharp.Engine;

[UClass]
public class AMaterialCopier : AActor
{
    protected override void BeginPlay()
    {
        base.BeginPlay();

        UMaterialInstanceDynamic sourceMid = LoadObject<UMaterialInstanceDynamic>(null, "/Game/Materials/MI_SourceInstance.MI_SourceInstance");

        UPrimitiveComponent primComp = GetComponentByClass<UPrimitiveComponent>();
        UMaterialInstanceDynamic targetMid = primComp.CreateDynamicMaterialInstance(0);

        if (sourceMid != null && targetMid != null)
        {
            targetMid.CopyParameterValues(sourceMid); // Copies all compatible parameters
            SystemLibrary.Log(ELogLevel.Display, "Parameters copied — perfect clones!");
        }
    }
}
```

**Pro Note**: Use `CopyScalarParameters`, `CopyVectorParameters` for selective copying.

### 7. Animate Parameter Over Time (Opacity Pulse)

Smooth fades, ghost effects, or breathing glows.

```csharp
using UnrealSharp.CoreUObject;
using UnrealSharp.Engine;

[UClass]
public class AMaterialAnimator : AActor
{
    private UMaterialInstanceDynamic CachedMid;

    protected override void BeginPlay()
    {
        base.BeginPlay();

        UPrimitiveComponent primComp = GetComponentByClass<UPrimitiveComponent>();
        CachedMid = primComp.CreateDynamicMaterialInstance(0);
    }

    public override void Tick(float deltaSeconds)
    {
        base.Tick(deltaSeconds);

        if (CachedMid != null)
        {
            float opacity = (float)(Math.Sin(TimeSeconds * 2.0) * 0.5 + 0.5); // 0–1 pulse
            CachedMid.SetScalarParameterValue(new FName("Opacity"), opacity);
        }
    }
}
```

**Pro Note**: Always cache the MID—never recreate in Tick!

### 8. Query All Scalar Parameters (Debug/Introspection)

Log available parameters for documentation or runtime UI.

```csharp
using UnrealSharp.Engine;

[UClass]
public class AMaterialParameterLister : AActor
{
    protected override void BeginPlay()
    {
        base.BeginPlay();

        UMaterial baseMat = LoadObject<UMaterial>(null, "/Game/Materials/M_MyMat.M_MyMat");

        if (baseMat != null)
        {
            TArray<FMaterialParameterInfo> scalarParams = new TArray<FMaterialParameterInfo>();
            TArray<float> defaultValues = new TArray<float>();

            baseMat.GetAllScalarParameterInfo(scalarParams, defaultValues);

            for (int i = 0; i < scalarParams.Num; i++)
            {
                SystemLibrary.Log(ELogLevel.Display, $"Scalar: {scalarParams[i].Name} = {defaultValues[i]}");
            }
        }
    }
}
```

**Pro Note**: Also available: `GetAllVectorParameterInfo`, `GetAllTextureParameterInfo`.

### 9. Global Parameter Collections (Level-Wide Effects)

For rain wetness, time-of-day, etc., affecting many materials at once.

```csharp
using UnrealSharp.Engine;

[UClass]
public class AGlobalWetnessController : AActor
{
    protected override void BeginPlay()
    {
        base.BeginPlay();

        UMaterialParameterCollection collection = LoadObject<UMaterialParameterCollection>(null, "/Game/Materials/MPC_GlobalEffects.MPC_GlobalEffects");

        if (collection != null)
        {
            UMaterialParameterCollectionInstance instance = GetWorld().GetParameterCollectionInstance(collection);
            instance.SetScalarParameterValue(new FName("GlobalWetness"), 0.7f);
            SystemLibrary.Log(ELogLevel.Display, "Global wetness applied — everything slick!");
        }
    }
}
```

**Pro Note**: Use Material Parameter Collections in editor for truly global params (seen in your GhostMaterialParams).

### 10. Batch Apply to Multiple Actors

Level-wide events (night mode, damage overlay).

```csharp
using UnrealSharp.Engine;

[UClass]
public class ABatchMaterialApplier : AActor
{
    protected override void BeginPlay()
    {
        base.BeginPlay();

        TArray<AActor> actors = new TArray<AActor>();
        GetAllActorsOfClass(typeof(AStaticMeshActor), actors);

        foreach (AActor actor in actors)
        {
            UStaticMeshComponent mesh = actor.GetComponentByClass<UStaticMeshComponent>();
            if (mesh != null)
            {
                UMaterialInstanceDynamic mid = mesh.CreateDynamicMaterialInstance(0);
                mid.SetScalarParameterValue(new FName("DamageOverlay"), 0.5f);
            }
        }

        SystemLibrary.Log(ELogLevel.Display, $"Applied damage overlay to {actors.Num} actors!");
    }
}
```

**Pro Note**: Filter with tags: `if (actor.ActorHasTag("AffectedByWeather"))`.
