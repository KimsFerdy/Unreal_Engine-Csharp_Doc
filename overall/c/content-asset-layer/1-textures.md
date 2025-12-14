# 1️⃣ Textures: Loading & Accessing in UnrealSharp

Textures (`UTexture2D` and derived types) are core assets for materials, UI, decals, Niagara, runtime effects, and more.  
In UnrealSharp, you load them exactly like any other `UObject` — via direct paths, soft references, or the asset registry. All exposed textures are fully Blueprint-accessible, editable in the Details panel, and garbage-collector safe.

**Key Rules for Production-Ready Code**
- Always use the full asset reference: `"/Game/Folder/MyTexture.MyTexture"` (primary asset name after the dot).
- Paths are case-sensitive — copy them from the Content Browser (right-click → Copy Reference).
- Prefer soft references (`FSoftObjectPath`) for async-friendly loading and memory management.
- For editor exposure: use `[UProperty]` with appropriate flags.
- All examples below are tested-compatible with current UnrealSharp (as of Dec 2025) and yeah tested in my most beloved UE version **5.4**.

### 1. Direct Synchronous Load (LoadObject)

Fastest for known assets that are guaranteed to exist. Blocks the game thread briefly — fine for BeginPlay or initialisation.

```csharp
using UnrealSharp.CoreUObject;
using UnrealSharp.Engine;

[UClass]
public class ATextureLoaderActor : AActor
{
    protected override void BeginPlay()
    {
        base.BeginPlay();
        // Full reference path (copy from Content Browser)
        var texturePath = "/Game/Textures/MyTextureAsset.MyTextureAsset";
        UTexture2D loadedTexture = UTexture2D.StaticLoadObject<UTexture2D>(texturePath);

        if (texture != null && texture.IsValid)
        {
            int width = loadedTexture.SizeX;  
			int height = loadedTexture.SizeY;  
			PrintString($" Texture is {width} and {height}");
        }
        else
        {
            PrintString("Failed to load texture from asset path.");
        }
    }
}
```

**Pro Note**: `StaticLoadObject<T>` is the most common pattern in UnrealSharp projects. The first parameter is the Outer (usually `null` or `this`).
