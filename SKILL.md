---
name: liquid-glass-ui
description: Build authentic liquid glass UI components that truly refract light, rather than just faking it with a flat blur. Uses a computed SVG displacement map and a filtered copy of the backdrop to work across all modern browsers (Safari, Firefox, Chrome). Trigger this skill when the user asks for authentic glassmorphism, liquid glass, refractive panels, or outpace-style glass UI.
---

# Liquid Glass UI

Use this skill to design and implement authentic liquid glass components that physically bend light and work across all major browsers, based on the Outpace Studios technique.

## Non-Negotiable Foundations

- **Blur isn't glass.** Do not use a simple `backdrop-filter: blur()` gradient. That is frosted glass.
- Real glass refracts. It bends the light passing through it, especially at the curved edges.
- You must use an SVG `feDisplacementMap` filter to drive the refraction.
- You must filter a *copy* of the backdrop, not the live backdrop itself. Safari and Firefox do not support SVG filters inside `backdrop-filter`.

## Typography

Typography should be crisp, modern, and highly legible, acting as a structured anchor against the fluid nature of the glass.

- **Primary Font**: Use Inter (or a similar neo-grotesque sans-serif) for all UI text and body copy.
- **Monospace Font**: Use Geist Mono (or similar) for code snippets, technical data, or monospaced numbers.
- **Headings**: Keep letter-spacing tight (`-0.4px` to `-0.1px`) and use medium font weights (`450` to `550`). Avoid ultra-bold weights; precision is key.
- **Colors**:
  - Dark text on light surfaces: High contrast `rgba(23, 23, 23, 0.92)` for headings, muted `rgba(23, 23, 23, 0.62)` for body text.
  - Light text on dark/glass surfaces: High contrast `rgba(255, 255, 255, 0.96)`.

## Density & Layout (Compact Default)

The layout should balance a spacious reading experience with tightly clustered, compact UI controls.

- **Content Constraints**: Keep reading columns narrow and focused (e.g., `max-width: 640px`). Use generous vertical margins between sections (`80px` to `104px`).
- **Control Density**: UI controls (like the glass dropdown triggers) should be compact. Use padding like `10px 14px` and smaller font sizes (`14px` to `15px`).
- **Rounding**: Use extreme rounding for floating UI elements. Buttons and floating glass panels should use pill shapes (`border-radius: 9999px` or `999px`) or large, smooth squircles (e.g., `32px` radius for large content panels).

## Color Palette

The interface relies on extreme contrast between the dark environmental background and the crisp foreground elements.

- **Environment/Backdrop**: Very dark, deep tones (`#0b0e13`). This allows the glass refraction to pick up rich, dark colors and bright specular highlights.
- **Foreground Content Panels**: Pure white (`#ffffff`) or highly opaque light panels that contrast sharply with the dark environment.
- **Glass Shell**: The glass itself should have a very subtle, translucent dark fill (e.g., `rgba(0,0,0,0.22)`) with a white rim light to define the edge.

## Core Technical Architecture

### 1) The Displacement Map
- Compute the displacement based on physical optics (e.g., a convex squircle dome), not an arbitrary gradient.
- The bend concentrates at the rim (where the slope is steepest) while the center remains relatively clear.
- Provide the displacement map to the filter as a `blob:` URL. Safari WebKit silently refuses `data:` URIs inside `feImage`.
- Force the filter to run in sRGB so displacement values are interpreted accurately.

### 2) The Component Pattern
Create a two-part system to handle the cross-browser refraction without relying on screenshots or WebGL:

1. **`GlassScene`**: Renders the backdrop once and shares it.
2. **`GlassLens`**: Drops a counter-positioned copy of the backdrop into the lens box and applies the SVG filter to bend it.

```jsx
<GlassScene content={<Backdrop />}>
  <nav>{triggers}</nav>

  {open && (
    <GlassLens x={x} y={y} width={W} height={h}>
      {menuItems}
    </GlassLens>
  )}
</GlassScene>
```

### 3) Nested Inset + Popping Button Pattern (Learned Rule)
When building tactile controls (like toggles, sliders, or segmented controls) with liquid glass, use this specific layering pattern:
1. **The Inset Well**: Create a recessed track or well on the underlying panel using dark inset shadows (e.g., `box-shadow: inset 0 2px 6px rgba(0,0,0,0.4)`). The well itself is NOT the glass; it is the physical trench that houses it.
2. **The Popping Glass Button**: The interactive object (the thumb of a slider, or the active pill of a toggle) is the refractive liquid glass `GlassLens`. It sits *inside* the well but appears raised due to the heavy refraction at its steep curved rim and a subtle drop shadow (`box-shadow: 0 4px 12px rgba(0,0,0,0.2)`).
3. **The Interaction**: As the glass button moves within the well, it actively bends the inset shadows and the backdrop beneath it, creating a highly physical, tactile sliding sensation.

### 4) Safari Concessions
- Safari caches a filter's output by its `id`. If your lens animates, generate a fresh `id` on every rebuild so it doesn't freeze on the first frame.
- Safari caps the size of the source graphic a filter will process. Clip the refraction copy to the lens box before it rasterizes, otherwise the filter produces nothing.

## Accessibility & Interaction Rules

- **Interactive Base**: The UI beneath the glass (the real backdrop) must remain fully interactive. The glass is just a visual overlay bending a copy.
- **Shared Lens Animation**: If multiple items trigger the glass (like a navigation menu), use a *single* glass lens that travels and morphs between items, rather than fading individual panels in and out. The motion should be spring-driven but interruptible.
- **Motion**: Respect `prefers-reduced-motion` by cutting to new positions instead of springing.
- **Transparency**: Respect `prefers-reduced-transparency` by dropping the refraction entirely and falling back to an opaque panel for maximum readability.
