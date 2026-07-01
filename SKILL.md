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

### 3) Safari Concessions
- Safari caches a filter's output by its `id`. If your lens animates, generate a fresh `id` on every rebuild so it doesn't freeze on the first frame.
- Safari caps the size of the source graphic a filter will process. Clip the refraction copy to the lens box before it rasterizes, otherwise the filter produces nothing.

### 4) Accessibility & Interaction Rules
- The UI beneath the glass (the real backdrop) must remain fully interactive. The glass is just a visual overlay bending a copy.
- **Motion:** Respect `prefers-reduced-motion` by cutting to new positions instead of springing.
- **Transparency:** Respect `prefers-reduced-transparency` by dropping the refraction entirely and falling back to an opaque panel for maximum readability.
