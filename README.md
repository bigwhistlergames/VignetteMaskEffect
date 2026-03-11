# VignetteMaskEffect

# Vignette Package

A powerful and flexible texture-based vignette system for Unity URP (Universal Render Pipeline).

## Features

- **Texture-based masking** - Use any texture with alpha channel to create custom vignette shapes
- **Runtime control** - Enable/disable and animate vignette effects at runtime
- **Animation curves** - Smooth animations with customizable easing curves
- **URP Render Graph** - Uses Render Graph when enabled; falls back to compatibility mode automatically
- **Aspect ratio correction** - Maintains texture proportions across different screen sizes
- **Simple API** - Fluent builder pattern for easy setup

## Requirements

- Unity 2020.3 or later
- Universal Render Pipeline (URP)
- For Render Graph path: Unity 6 / URP 17+ (optional; compatibility mode works on older versions)

## Quick Start

### 1. Setup Renderer Feature

1. Select your **Forward Renderer** asset (usually `Assets/ForwardRenderer.asset`)
2. Click **Add Renderer Feature**
3. Select **Vignette Renderer Feature**
4. Configure settings:
   - **Enabled By Default**: Whether vignette is active on scene start. You can modify it later in runtime by API.
   - **Default Mask Texture**: Your vignette mask texture (white alpha = visible, black alpha = hidden)
   - **Default Scale**: Initial scale value (1.0 = original size)

Optional: For best performance use RenderGraph. PROJECT SETTINGS -> GRAPHICS -> COMPATIBILITY MODE (disable).

### 2. Prepare Mask Texture

1. Import your vignette mask texture (PNG with alpha channel)
2. In the texture import settings:
   - **Texture Type**: Default or Sprite (2D and UI)
   - **Alpha Source**: Input Texture Alpha

### 3. Basic Usage

```csharp
    void Start()
    {
        // Initialize vignette with custom texture
        VignetteController.For(Camera.main)
            .WithTexture(vignetteTexture)
            .WithScale(1.0f)
            .Init();
    }

    void Start()
    {
        // Initialize with default values
        VignetteController.For(Camera.main).Init();
    }


    var _camera = Camera.main;
    ...
    // set vignette true
    VignetteController.SetActive(_camera, true);
    ...
    // or
    VignetteController.SetActive(_camera, false);
    ...
    // modify scale un runtime 
    VignetteController.SetScale(_camera, 0.5f),
    ...
    // sample of DOTWEEN usage for custom animations
    var _currentVignetteScale = 0f;
    DOTween.To(() => _currentVignetteScale, 
        x => VignetteController.SetScale(Camera.main, x), 
        1f, 
        1f)
    .SetEase(Ease.Linear);
```

## Common Use Cases

### Fade In Effect

```csharp
void FadeIn()
{
    VignetteController.Enable(Camera.main);
    VignetteController.SetScale(Camera.main, 0.1f);
    VignetteController.AnimateScale(Camera.main, 2.0f, 1.0f, () => 
    {
        Debug.Log("Fade in complete!");
    });
}
```

### Fade Out Effect

```csharp
void FadeOut()
{
    VignetteController.Enable(Camera.main);
    VignetteController.SetScale(Camera.main, 2.0f);
    VignetteController.AnimateScale(Camera.main, 0.1f, 1.0f, () => 
    {
        VignetteController.Disable(Camera.main);
    });
}
```

### Custom Animation Curve

```csharp
void FadeWithBounce()
{
    AnimationCurve bounceCurve = new AnimationCurve(
        new Keyframe(0, 0),
        new Keyframe(0.5f, 1.1f),
        new Keyframe(1, 1)
    );
    
    VignetteController.AnimateScale(
        Camera.main, 
        2.0f, 
        1.5f, 
        null, 
        bounceCurve
    );
}
```

### Scene Transition

```csharp
IEnumerator TransitionToScene(string sceneName)
{
    // Fade out
    bool fadeComplete = false;
    VignetteController.AnimateScale(Camera.main, 0.1f, 1.0f, () => 
    {
        fadeComplete = true;
    });
    
    yield return new WaitUntil(() => fadeComplete);
    
    // Load scene
    SceneManager.LoadScene(sceneName);
    
    // Fade in
    VignetteController.SetScale(Camera.main, 0.1f);
    VignetteController.AnimateScale(Camera.main, 2.0f, 1.0f);
}
```

## API Documentation

See [API.md](API.md) for complete API reference.

## Troubleshooting

### Vignette not visible
- Ensure VignetteRendererFeature is added to your Forward Renderer
- Check that "Enabled By Default" is checked or call `VignetteController.Enable()`
- Verify mask texture has proper alpha channel
- Check texture import settings (Alpha Source should be "Input Texture Alpha")

### Performance issues
- **Render Graph mode** (URP default): one full-screen blit plus a zero-cost buffer swap; best on mobile.
- **Compatibility mode**: two blits (apply + copy-back); still minimal (~0.1-0.3ms).
- Ensure you're not creating multiple animator GameObjects.

## Package Structure

```
Vignette/
├── README.md                       # This file
├── API.md                          # API documentation
├── Scripts/
│   ├── VignetteController.cs      # Main API
│   ├── VignetteRendererFeature.cs # URP renderer feature
│   └── VignetteRenderPass.cs      # Single pass (Execute + RecordRenderGraph)
├── Shaders/
│   └── VignetteMask.shader        # Blit-compatible vignette shader
└── Demo/
    └── Script/
        ├── TestVignette.cs        # Example implementation
        └── Editor/
            └── TestVignetteEditor.cs  # Inspector buttons
```

The render pass supports both URP **Render Graph** and **Compatibility Mode**. No configuration is needed: URP calls `RecordRenderGraph` or `Execute` automatically based on **Project Settings > Graphics > URP > Render Graph**.

## Support

For issues, questions, or feature requests, please contact the package maintainer.
