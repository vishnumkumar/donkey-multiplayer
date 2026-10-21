# ADR-0001: Flutter + Flame 2.5D Rendering for Card Table

**Status:** ACCEPTED  
**Date:** 2026-10-20  
**Decision Makers:** Product Owner / Technical Lead  
**Relates to:** D11 (2.5D vs 3D), `09-visual-ux-art-direction.md`, `05-architecture-and-data-model.md`

## Context

The Donkey multiplayer card game requires a polished, "3D-style" visual presentation of a card table with four player seats, card dealing/playing/collecting animations, shadows, perspective, and highlights. The visual quality must feel premium while running at 60 FPS on affordable Indian smartphones (target: devices with 2–3 GB RAM, entry-level Mali/Adreno GPUs).

The app targets Android and iOS from a single codebase, which already constrains the technology choice to cross-platform frameworks.

Three broad rendering approaches were considered:

1. **Full 3D engine** (Unity, Unreal, or Godot with Flutter embedding)
2. **2.5D presentation** (Flutter + Flame engine with pre-rendered assets, transform matrices, and layered shadows)
3. **Pure Flutter Canvas** (custom painting without a game engine)

## Decision

**Use Flutter with the Flame game engine for 2.5D rendering.** The card table and cards achieve a 3D *appearance* through:

- Pre-rendered perspective table surface (PNG/WebP background)
- Card face/back texture atlases with layered shadow sprites
- Flutter/Flame `Transform` matrices for perspective skew, rotation, and scale
- Eased animation curves for deal, play, collect, and flip motions
- Highlight/glow overlays for selection and turn indication
- Quality presets (low/medium/high) with reduced effects on weaker devices

The game UI (menus, overlays, HUD elements) uses standard Flutter widgets overlaid on the Flame game canvas.

## Rationale

### Why not full 3D (Unity/Unreal/Godot)?

| Factor | Full 3D | 2.5D Flame |
|---|---|---|
| **App size** | +50–150 MB for engine runtime | Minimal overhead; Flame is a lightweight Dart package |
| **Build complexity** | Flutter-Unity/Godot bridge adds native platform layers, separate build pipelines, and interop overhead | Single Dart/Flutter build pipeline |
| **Developer skill** | Requires 3D modeling, shaders, lighting; different skill set from Flutter | Stays within Dart/Flutter ecosystem; 2D sprite and transform skills |
| **Performance floor** | Full 3D scene graph, physics, lighting overhead even for a static table | Only composites flat sprites with transforms; GPU cost is minimal |
| **iOS constraints** | Metal required; OpenGL ES deprecated; engine must support Metal well | Flutter's Impeller renderer handles Metal natively |
| **Iteration speed** | Longer rebuild cycles; engine-specific tooling | Hot reload; standard Flutter dev workflow |
| **Maintenance** | Two ecosystems to maintain and update | One ecosystem |

A card game does not need physics simulation, real-time lighting, or mesh rendering. The visual "3D" effect is achieved entirely through authored art (shadows, highlights, perspective distortion) and 2D transforms — the same technique used in most successful mobile card games.

### Why Flame over pure Flutter Canvas?

- Flame provides a game loop, sprite management, component system, and animation utilities optimized for game rendering
- Built-in sprite sheet/atlas support for efficient card texture management
- Component lifecycle management for card entities, table regions, and animations
- Well-documented: https://docs.flame-engine.org/
- Active community and regular updates tracking Flutter stable
- Still pure Dart — no native bridge overhead

### Why not raw Flutter CustomPainter?

CustomPainter can render the same visuals but requires building game-loop timing, sprite management, animation sequencing, and input hit-testing from scratch. Flame provides these as tested, maintained abstractions.

## Asset Pipeline

1. **Card faces (52) + back (1–3 variants):** Authored as high-resolution PNG/WebP, exported to texture atlas (sprite sheet) at 2x and 3x densities. Target per-card size: ~200×280 logical pixels.
2. **Table surface:** Pre-rendered perspective felt texture at multiple resolutions (phone portrait, tablet landscape). Dark emerald/navy palette.
3. **Shadow layers:** Separate shadow sprites composited below cards; adjustable opacity for quality presets.
4. **Suit tokens:** 4 suit indicator graphics for lead-suit display.
5. **UI atlas:** Buttons, indicators, countdown elements, emoji reactions.
6. **Audio:** Shuffle, deal, play, collect, hit alert, podium — loaded lazily, not bundled at max quality.

Total estimated asset budget: 15–25 MB compressed in app bundle; lazy-load optional HD packs if needed.

## Animation Approach

| Animation | Duration | Technique |
|---|---|---|
| Deal (per card burst) | 150–250 ms | Translate + rotate from deck position to hand fan; eased curve |
| Card play | 180–280 ms | Translate from hand to table center; slight scale up |
| Round collect (hit) | 450–650 ms | Cards slide to winner's pile; brief highlight flash |
| Round discard (no hit) | 350–500 ms | Cards fade/slide to discard area |
| Card flip | 200–300 ms | Horizontal scale to 0 (showing edge), swap texture, scale back to 1 |
| Turn indicator | Continuous | Subtle glow pulse on active seat; configurable frequency |

All durations respect `AccessibilityFeatures.reduceMotion` — when enabled, animations complete instantly or use minimal 50 ms fades.

## Performance Strategy

### Quality Presets

| Preset | Shadows | Highlights | Table | Animations | Target Devices |
|---|---|---|---|---|---|
| **High** | Full layered shadows | Specular card-edge glow | HD perspective texture | Full eased curves | Flagships, 4+ GB RAM |
| **Medium** | Single shadow layer | Flat selection highlight | Standard texture | Standard curves | Mid-range, 3 GB RAM |
| **Low** | No shadows | Border-only highlight | Static flat color | Instant/minimal | Entry-level, 2 GB RAM |

### Memory Management

- Texture atlas loaded once; individual card sprites are regions, not separate images
- Off-screen cards not rendered (hand fan culling)
- No unbounded texture growth — fixed atlas set per quality preset
- Dispose textures on match end; reload on next match
- Monitor GPU memory via Flutter DevTools during development

### iOS Considerations

- Flutter's Impeller rendering engine is the default on iOS and uses Metal natively
- No OpenGL ES dependency
- Test on iPhone SE (oldest supported) and current iPhone for performance floor/ceiling
- Verify Flame compatibility with Impeller (Flame 1.x+ supports Impeller)

## Consequences

### Positive

- Single codebase, single build pipeline, single language (Dart)
- Minimal app size overhead from rendering engine
- Hot reload during visual development
- Quality presets provide graceful degradation
- Art direction can be iterated independently of code (swap texture atlases)
- Performance headroom — compositing flat sprites is far cheaper than 3D scene rendering

### Negative

- Cannot achieve true 3D effects (parallax on device tilt, real-time shadows from dynamic light sources) — only approximations through authored art
- Flame ecosystem is smaller than Unity; fewer ready-made effects/shaders
- If future requirements demand true 3D (e.g., AR card placement, 3D avatar customization), significant rework would be needed
- Card perspective distortion via 2D transforms has limits — extreme angles may look flat

### Risks and Mitigations

| Risk | Likelihood | Mitigation |
|---|---|---|
| Flame version incompatible with latest Flutter stable | Low | Pin Flame version; test on each Flutter upgrade; Flame tracks stable closely |
| Authored 2.5D assets look flat compared to competitor | Medium | Invest in quality art direction; shadow/highlight layers; A/B test with users |
| Low-end device frame drops during animations | Medium | Quality presets; animation budget enforcement; profile on real devices |
| Impeller rendering bugs on specific GPU drivers | Low | Test on Mali and Adreno devices; report/workaround Flutter issues |

## Alternatives Considered

| Alternative | Reason Rejected |
|---|---|
| **Unity + Flutter bridge** | Excessive complexity, binary size, build pipeline split, developer skill mismatch, maintenance burden |
| **Unreal Engine** | Overkill for 2D card game; massive binary; C++ development |
| **Godot** | Smaller ecosystem; Flutter bridge is experimental; adds native layer |
| **Raw Flutter CustomPainter** | Viable but reinvents game-loop, sprite management, and animation utilities that Flame provides |
| **Rive/Lottie for all animations** | Good for isolated animations but not for interactive game state rendering with input handling |

## References

- Flame engine documentation: https://docs.flame-engine.org/
- Flutter Impeller renderer: https://docs.flutter.dev/perf/impeller
- `09-visual-ux-art-direction.md` — visual design tokens and art deliverables
- `05-architecture-and-data-model.md` — client architecture context
