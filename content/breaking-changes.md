---
title: Babylon.js Breaking Changes
image:
description: Breaking changes and how to adapt for them
keywords: backwards compatibility, breaking
further-reading:
video-overview:
video-content:
toc-levels: 2
---

## 8.10.1

### Fix target camera orientation issues when using right-handed scenes

When using right-handed scenes, setting and getting the `rotation` or `rotationQuaternion` properties of a target camera was 180-rotated on the Y-axis when compared to the view or world matrix of the camera. This is confusing and caused problems when exporting glTF. Any code that previously set the `rotation` or `rotationQuaternion` properties of a camera derived from `TargetCamera` (`FreeCamera` or `ArcRotateCamera` for example) when using right-handed scenes will now be wrong. The default orientation of a target camera still faces +Z as before to maintain backwards compatibility. Left-handed scenes are also unaffected. See [PR](https://github.com/BabylonJS/Babylon.js/pull/16691) for more information.

## 8.2.0

### PBR: Add flag to calculate legacy translucency

In previous PRs (https://github.com/BabylonJS/Babylon.js/pull/16337, https://github.com/BabylonJS/Babylon.js/pull/16214), we corrected the way translucency/transmission is calculated.
However, we failed to add a flag to allow you to render your scene exactly as it was before these changes. We have therefore added a property `mat.subSurface.legacyTranslucency` (default: false), in case your scene does not render as expected with the new code and you want an easy way to do it. You can also set `PBRSubSurfaceConfiguration.DEFAULT_LEGACY_TRANSLUCENCY` to true to automatically apply the back compat behavior to all materials.

PR: https://github.com/BabylonJS/Babylon.js/pull/16446

## 7.54.0

### PBR: Fix calculation of diffuse transmission in PBR materials

This is a bug fix, but it may have an impact on the rendering of your scene. Previously, we incorrectly applied the albedo color when outputting the subsurface block calculation.
However, if you need to restore the old behavior for some reason, you can do `mat.subSurface.applyAlbedoAfterSubSurface = true`. Note that from 8.0.2 and forward you can also set `PBRSubSurfaceConfiguration.DEFAULT_APPLY_ALBEDO_AFTERSUBSURFACE` to true to automatically apply the back compat behavior to all materials.
Note that you can probably make your assets compatible with the new code without needing to set `applyAlbedoAfterSubSurface = true` by setting the correct texture in the `translucencyColorTexture` property.

PR: https://github.com/BabylonJS/Babylon.js/pull/16337

## 7.52.0

### Deprecation of legacy audio engine

To pave the way for the new audio engine, the old audio engine is no longer created by default when the graphics engine is initialized. To use the old deprecated audio engine, set the `audioEngine` option to `true` in the graphics engine constructor, for example:

```javascript
const engine = BABYLON.Engine(canvas, true, { audioEngine: true }, true);
```

PR: https://github.com/BabylonJS/Babylon.js/pull/15839

## 7.51.0

### Fix calculation of diffuse transmittance in PBR materials

This is a bug fix, but it may have an impact on the rendering of your scene. It will manifest itself as lighter materials when you activate translucency on a PBR material.

PR: https://github.com/BabylonJS/Babylon.js/pull/16214

## 7.47.3

### Fix alpha support
It was not always easy to understand how alpha was applied, depending on the value of certain material properties, and the handling of certain glTF files with transparency was even broken.

Since version 7.47.3, if you give a non-null value to `transparencyMode`, it will always be respected, regardless of the values you give to the other properties (such as the mesh `visibility`, the material `alpha`, etc.). This makes it easy to apply alpha testing, alpha blending, or both at the same time, by setting the correct value for `transparencyMode`.

However, this change could break your code. If it does, most of the time, the solution is simply to set the correct value for `transparencyMode`.

PR: https://github.com/BabylonJS/Babylon.js/pull/16144

## 7.46.0

### Loading screen on multiple canvases
In order to manage the visibility of the loading screen on multiple canvases, we have changed the underlying data structure that tracks the loading screen info.

However, if you have implemented an own custom loading screen, this change could break your code. If it does, you should:
1. Manually set the `this._isLoading` attribute to true in `displayLoadingUI` and false in `hideLoadingUI`.
2. Manage your custom loading div through `this._loadingDivToRenderingCanvasMap` rather than the old `this._loadingDiv` attribute.

Here is an example of a PG using a custom loading screen:
<Playground id="#5Y2GIC#847" title="Custom Loading Screen Example" description="Simple example showing how to create and use a custom loading screen."/>

## 7.45.0

### `PBRMaterial` rough metals are looking closer to ray traced results
Improving the way we render is an ongoing mission for the Babylon team and as explained on the [forum](https://forum.babylonjs.com/t/differences-in-pbr-material-behavior-between-babylon-5-and-7/56585/5?u=sebavan), our PBR rendering has been improved to more correct results.

As we understand you might have changed your art to adapt to the previous behavior, you can bring it back by setting `PBRBRDFConfiguration.DEFAULT_MIX_IBL_RADIANCE_WITH_IRRADIANCE = false;` before creating your `PBRMaterial`.

## 7.34.0

### `SceneLoader.Load`, `SceneLoader.Append`, `SceneLoader.ImportMesh` does not return the plugin used to load the content anymore
In order to support automatic mimetype detection, we have removed the synchronous aspect of these 3 functions and they will return void for now on.
More details: https://forum.babylonjs.com/t/potential-new-breaking-change-please-chime-in/54651

To get the plugin used to load content, you can register to `SceneLoader.OnPluginActivatedObservable`

## 7.31.0

### Instance now get the same parent as the source
Before the instances where created with no parent but this was causing a lot of rendering issues when the source was coming from a glTF object. This change will make sure the source and the instance have the same parent.

### Deprecation of CSG class in favor of CSG2

The old CSG class was made of pure JS (it was a port from https://github.com/evanw/csg.js/) and while it served us, it was not maintained and not usable anymore.
We are now encouraging everyone to switch the new CSG2 class.

### Porting from CSG to CSG2

#### Asynchronous initialization

The main difference will be that CSG2 uses the [fantastic Manifold](https://github.com/elalish/manifold) to do the boolean operations. To that extend you have to call `BABYLON.InitializeCSG2Async()` before being able to use the CSG2 class.

Example:

```
await BABYLON.InitializeCSG2Async();

const sphereCSG = BABYLON.CSG2.FromMesh(sphere);
const boxCSG = BABYLON.CSG2.FromMesh(box);

const mesh = boxCSG.subtract(sphereCSG).toMesh("test");
```

### Inverse

The inverse feature is no more supported (and it was not really working so we all should be fine)

#### ToMesh

The method `toMesh` does take 3 parameters: name, scene and options.

#### MultiMaterials

CSG2 has a far simpler way to deal with multi-materials as it will automatically rebuild meshes with multi-materials if the source meshes had different materials.

### Example

Here is an example of a PG using CSG:
<Playground id="#VYZEEQ#4" title="CSG Subtract Example" description="Simple example of using a CSG subtract operation."/>

And the converted version:
<Playground id="#PJQHYV#1" title="CSG Subtract Example" description="Simple example of using a CSG2 subtract operation."/>

### Using CSG2

Please refer to the CSG2 [class documentation for more details](/features/featuresDeepDive/mesh/mergeMeshes#merging-meshes-with-constructive-solid-geometry)

## 7.19.0

### Main materials now generate WGSL shader code when used with WebGPU

This change is an improvement as the main shaders (StandardMaterial, PBRMaterial and BackGroundMaterial) are now generating pure WGSL and does not require compilation of GLSL code with TintWASM when used with a WebGPUEngine.
If you still need to get GLSL code (for instance if you inject your own custom code with a MaterialPlugin0, you can force the materials to use the GLSL code path with a boolean in the materials' constructors.

## 7.11.0

### mesh.overrideMaterialSideOrientation was renamed to mesh.sideOrientation

This change was made to simplify the sideOrientation process. Starting with this version, mesh.sideOrientation is used UNLESS material.sideOrientation is not null.
[content/breaking-changes.md](https://github.com/BabylonJS/Babylon.js/pull/15189)

## 7.6.0

### canvas is not xrCompatible per default

This change should not influence our XR users, but is a breaking change - instead of defining the canvas XR compatible right when creating the webgl2 context, the canvas is being made xr compatible when entering XR.
[content/breaking-changes.md](https://github.com/BabylonJS/Babylon.js/pull/15027)

## 7.2.2

### Remove WebGPUEngine Dependency on Engine

**[14931](https://github.com/BabylonJS/Babylon.js/pull/14931)**
This is a rather large change to decouple `WebGPUEngine` from `Engine` where all properties of `Engine` in the library are now properties of `AbstractEngine`. The existing methods, however, are all unchanged so the impact to them should be minimal. `ThinEngine` and `Engine` are almost entirely unchanged but there are two important considerations to be mindful of:

1. `AbstractEngine` is abstract and should not be instantiated
2. All common functions have been moved from `ThinEngine` to `AbstractEngine` so they can be shared with WebGPU Engine

The main impact on user code will only happen on TypeScript as the type of all `getEngine()` functions will be `AbstractEngine` instead of `ThinEngine`.

## 7.0.0

### Thin Instances

**[#14679](https://github.com/BabylonJS/Babylon.js/pull/14679)** The default value of the thin instances staticBuffer parameter was changed to true. When staticBuffer parameter is set to false, Angle can rearrange the buffers under the hood and completely break performance when in DirectX mode. In OpenGL mode using a value of false for the staticBuffer parameter does not affect performance. Due to the performance issues in DirectX mode, the new default for staticBuffer will be true to maintain performance levels.

### WebVR

**[#14439](https://github.com/BabylonJS/Babylon.js/pull/14439)** The WebVR API was deprecated and has now been removed from the engine. The remaining part of the previous implementation of WebVR is the VR experience helper. Leaving this in the engine will help any experiences that used WebVR and the VR experience helper to continue working. In the case that the VR experience helper is called in an experience the engine will correctly fall back to using WebXR. This will enable older VR experiences to continue to function while it is updated to remove the deprecated feature.

### ArrayBufferView

**[#13946](https://github.com/BabylonJS/Babylon.js/pull/13946)** Added ArrayBufferView to be one of the accepted types in the SceneLoader import methods. The SceneLoader methods for ImportMesh, ImportMeshAsync, Load, LoadAsync, Append, AppendAsync, and LoadAssetContainer have traditionally accepted the types string and File. Now these methods will also be able to receive the type ArrayBufferView.

This was required by Babylon Native for those times when model data is passed from C++ into JavaScript by mapping native memory into ArrayBuffers. Included with this change is an additional visualization test for loading a mesh directly from binary data.

### glTF Serializer

**[#13909](https://github.com/BabylonJS/Babylon.js/pull/13909)** The glTF exporter used to attempt to change the mesh data on serialization to bake in a conversion from a left-handed scene - which is the default in Babylon.js - to a right-handed coordinate system which is expected by the glTF specification. This conversion was causing issues with some exported assets so the breaking change was to stop attempting to change the mesh data to account for a right-handed coordinate system to match the glTF specification and instead negate one axis of the asset. This results in a negative scale on one axis of the asset's root node when imported into a left-handed Babylon.js scene.

### Material Cloning

**[#13807](https://github.com/BabylonJS/Babylon.js/pull/13807)** When cloning materials, the default behavior has been to clone any textures used in multiple material channels once for each channel that uses it. To prevent duplicating a texture multiple times when cloning a material, the parameter `cloneTexturesOnlyOnce` was added to clone to prevent extra duplication of textures allowing multiple material channels to reference a single texture. Since this behavior logically should be the way clone works, the default value of `cloneTexturesOnlyOnce` is set to true, which is a breaking change from how the engine used to work. If the previous behavior is desired, setting this parameter to false will restore the previous behavior.

### ShaderPath

**[#14908](https://github.com/BabylonJS/Babylon.js/pull/14908)** Previously when using ShaderMaterial, the constructor accepted a type of _any_ for ShaderPath which could cause issues when using ShaderMaterial in TypeScript projects. This changes adds types to ShaderPath instead of _any_, which will protect type compatibility with ShaderMaterial and TypeScript.
