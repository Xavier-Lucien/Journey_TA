If a project has the URP package installed, Unity shows the URP Global Settings section in the Graphics tab in the Project Settings window.

The URP Global Settings section lets you define project-wide settings for URP.

![URP Settings Window](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/images/Inspectors/global-settings.png)

## [[Rendering Layers]]
[[Rendering Layers]]仅仅在3D项目渲染里面存在

## Shader Stripping
The check boxes in this section define which shader variants Unity strips when you build the Player.

|**Property**|**Description**|
|---|---|
|Shader Variant Log Level|Select what information about Shader variants Unity saves in logs when you build your Unity Project.  <br>Options:  <br>• Disabled: Unity doesn't save any shader variant information.  <br>• Only SRP Shaders: Unity saves only shader variant information for URP shaders.  <br>• All Shaders: Unity saves shader variant information for every shader type.|
|Strip Debug Variants|When enabled, Unity strips all debug view shader variants when you build the Player. This decreases build time, but prevents the use of Rendering Debugger in Player builds.|
|Strip Unused Post Processing Variants|When enabled, Unity assumes that the Player does not create new [Volume Profiles](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/VolumeProfile.md) at runtime. With this assumption, Unity only keeps the shader variants that the existing [Volume Profiles](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/VolumeProfile.md) use, and strips all the other variants. Unity keeps shader variants used in Volume Profiles even if the scenes in the project do not use the Profiles.|
|Strip Unused Variants|When enabled, Unity performs shader stripping in a more efficient way. This option reduces the amount of shader variants in the Player by a factor of 2 if the project uses the following URP features:<br><br>- Rendering Layers<br>- Native Render Pass<br>- Reflection Probe Blending<br>- Reflection Probe Box Projection<br>- SSAO Renderer Feature<br>- Decal Renderer Feature<br>- Certain post-processing effects<br><br>Disable this option only if you see issues in the Player.|
|Strip Screen Coord Override Variants|When enabled, Unity strips Screen Coordinates Override shader variants in Player builds.|