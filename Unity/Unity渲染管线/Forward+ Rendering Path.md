The Forward+ Rendering Path 让你能使用超出forward rendering path的每个灯光的上限。

The Forward+ Rendering Path has the following advantages compared with the Forward Rendering Path:

- There is no per-object limit for the number of Lights that affect GameObjects, the per-Camera limit still applies.  
    The per-Camera limits for different platforms are:
    
    - Desktop and console platforms: 256 Lights
    - Mobile platforms: 32 Lights. OpenGL ES 3.0 and earlier: 16 Lights.
    
    This let's you avoid splitting big meshes when more than 8 lights affect them.
    
- Blending of more than 2 reflection probes.
    
- Support for multiple Lights when using Unity Entity Component System (ECS).
    
- More flexibility with procedural draws.
    

See also: [Rendering Path comparison](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/urp-universal-renderer.html#rendering-path-comparison).

## How to select the Forward+ Rendering Path

To select the Forward+ Rendering Path, use the property **Rendering** > **Rendering Path** in the URP Universal Renderer asset.

![Select the Rendering Path in the URP Universal Renderer asset](https://docs.unity.cn/Packages/com.unity.render-pipelines.universal@16.0/manual/images/rendering/forward-plus/urp-select-forward-plus-path.png)

When you set the Rendering Path to Forward+, Unity ignores the values in the following properties in URP Asset, **Lighting** section:

- **Main Light**. With Forward+ the value of this property is **Per Pixel** regardless of the value you select.
    
- **Additional Lights**. With Forward+ the value of this property is **Per Pixel** regardless of the value you select.
    
- **Additional Lights** > **Per Object Limit**. Unity ignores this property.
    
- **Reflection Probes** > **Probe Blending**. Reflection probe blending is always on.
    

## Limitations

The Forward+ Rendering Path has no limitations compared with the Forward Rendering Path.