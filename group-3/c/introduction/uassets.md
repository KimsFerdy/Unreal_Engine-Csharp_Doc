# UAssets

## 20 Most used UAssets in Unreal Engine

***

### 1. **Texture2D**

**What**: 2D image data (albedo, normal, masks, UI).\
**Use**: Materials, UI, decals, runtime swaps.\
**Avoid**: Logic or data storage.\
**C# / BP**: Load via path or expose as property; often passed into Materials.

***

### 2. **Material**

**What**: Shader graph defining surface appearance.\
**Use**: Static visual definitions.\
**Avoid**: Runtime parameter changes (use instances).\
**C# / BP**: Rarely modified directly at runtime.

***

### 3. **Material Instance**

**What**: Parameterized version of a Material.\
**Use**: Runtime changes (colors, textures, scalars).\
**Avoid**: Complex logic inside materials.\
**C# / BP**: Create MID in C#, tweak params in BP or code.

***

### 4. **Static Mesh**

**What**: Non-animated geometry.\
**Use**: Props, environments, architecture.\
**Avoid**: Characters or deforming objects.\
**C# / BP**: Referenced by components, not manipulated directly.

***

### 5. **Skeletal Mesh**

**What**: Mesh with bones and skinning.\
**Use**: Characters, animated creatures.\
**Avoid**: Static props.\
**C# / BP**: Animation control via components and Anim Blueprints.

***

### 6. **Animation Sequence**

**What**: Keyframed skeletal animation.\
**Use**: Character motion.\
**Avoid**: State logic.\
**C# / BP**: Triggered via animation systems or state machines.

***

### 7. **Animation Blueprint**

**What**: Logic graph controlling animations.\
**Use**: Complex animation behavior.\
**Avoid**: Game logic or state ownership.\
**C# / BP**: C# feeds data, BP animates.

***

### 8. **Blueprint Class**

**What**: Visual class definition.\
**Use**: High-level orchestration and data binding.\
**Avoid**: Heavy logic or systems.\
**C# / BP**: BP extends or configures C# logic.

***

### 9. **Level (Map)**

**What**: World layout and placed actors.\
**Use**: Scene composition.\
**Avoid**: Game rules or persistent logic.\
**C# / BP**: Loaded, streamed, or queried at runtime.

***

### 10. **World Partition Data**

**What**: Large-world streaming system.\
**Use**: Open worlds, large maps.\
**Avoid**: Small levels.\
**C# / BP**: Mostly engine-managed, referenced indirectly.

***

### 11. **Data Asset**

**What**: Structured data container.\
**Use**: Configs, item definitions, tuning values.\
**Avoid**: Runtime mutable state.\
**C# / BP**: Read-only data source for systems.

***

### 12. **Data Table**

**What**: Row-based structured data (CSV-like).\
**Use**: Large sets of similar data.\
**Avoid**: Deep hierarchies.\
**C# / BP**: Queried by key or iterated.

***

### 13. **Curve (Float / Vector / Color)**

**What**: Value over time.\
**Use**: Smooth transitions, scaling, animation helpers.\
**Avoid**: Logic branching.\
**C# / BP**: Sampled during ticks or timelines.

***

### 14. **Sound Wave**

**What**: Raw audio asset.\
**Use**: Effects, voice, music.\
**Avoid**: Playback logic.\
**C# / BP**: Played via audio components.

***

### 15. **Sound Cue**

**What**: Audio logic graph.\
**Use**: Randomization, layering, variation.\
**Avoid**: Game logic.\
**C# / BP**: Triggered, not controlled.

***

### 16. **Niagara System**

**What**: Visual effects system.\
**Use**: Particles, VFX.\
**Avoid**: Gameplay state.\
**C# / BP**: Spawned and parameter-driven.

***

### 17. **Input Action**

**What**: Abstract input definition.\
**Use**: Player controls (Enhanced Input).\
**Avoid**: Direct key handling.\
**C# / BP**: Bound to gameplay code.

***

### 18. **Input Mapping Context**

**What**: Input bindings collection.\
**Use**: Contextual controls (menus, gameplay).\
**Avoid**: Hard-coded input switching.\
**C# / BP**: Activated/deactivated at runtime.

***

### 19. **Widget Blueprint**

**What**: UI layout and logic.\
**Use**: HUDs, menus, overlays.\
**Avoid**: Game rules or data ownership.\
**C# / BP**: C# drives data, BP displays.

***

### 20. **Physics Asset**

**What**: Collision bodies for skeletal meshes.\
**Use**: Ragdolls, hit detection.\
**Avoid**: Precise gameplay logic.\
**C# / BP**: Triggered, not authored, in code.
