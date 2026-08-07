# Motion Library

## Contents

1. Motion selection principles
2. Presets
3. Transition compatibility
4. Prompt pattern

## 1. Motion selection principles

Choose one primary motion per clip. Keep secondary movement ambient and physically plausible. Prefer motions that reveal the outfit without deforming graphics, hems, bags, jewelry, or shoes.

Use restrained camera movement. When the Source Reference has a strong fixed-camera composition, keep the camera locked.

## 2. Presets

### `ambient-breath`

- Subtle breathing and weight shift.
- Small fabric response only.
- Best for opening or closing beauty shots.

### `micro-turn`

- Turn head and upper torso by a small angle.
- Keep feet and scene geometry stable.
- Good for connecting two frontal looks.

### `garment-touch`

- Briefly adjust a collar, sleeve, jacket hem, or bag strap.
- Use only when the hand path is unobstructed.
- Avoid covering the hero graphic for most of the clip.

### `one-step`

- Take one controlled step and settle.
- Preserve body proportions and footwear.
- Use when there is visible floor space.

### `walk-in`

- Enter from just outside the frame and settle near the original pose.
- Use only when the composition provides a plausible entry edge.
- Higher drift risk; test before batch use.

### `edge-exit`

- Turn and move toward one frame edge.
- Useful as the final clip or before a directional transition.
- Do not fully reconstruct unseen background areas unnecessarily.

### `camera-push`

- Very slow optical or physical push toward the subject.
- Keep perspective and background geometry stable.
- Avoid combining with strong body movement.

### `glance-away-return`

- Look away briefly, then return gaze near camera.
- Preserve facial identity and expression family.
- Good for close or mid-length framing.

## 3. Transition compatibility

Plan transitions before generation.

- Match leftward exits with leftward entries or directional wipes.
- Match torso turns across adjacent clips for a motion cut.
- Use a garment-touch endpoint before a detail-oriented next shot.
- Use a camera push before a closer crop, not before a wider reverse movement.
- Use restrained crossfades only when background geometry is sufficiently similar.
- Prefer hard cuts on musical beats when motion endpoints already align.
- Avoid long crossfades on full-body subjects because the overlap creates a double-model ghost.
- For a light-leak transition, place the scene swap under peak light coverage and reveal the incoming Look as the leak recedes.
- Use one primary transition language across the sequence and expose it for user adjustment in the HyperFrames Studio preview.

Do not use transitions to conceal severe identity, product, or scene inconsistency. Reject the clip instead.

## 4. Prompt pattern

Use a short instruction that limits change:

```text
Animate this exact still as the sole visual reference. Preserve the same person,
face, hairstyle, body proportions, outfit, garment graphics, product colors,
accessories, background, lighting, framing, and camera perspective. Perform only
the following motion: [MOTION]. Keep the movement subtle and physically natural.
Do not add or remove people, garments, products, props, text, logos, or audio.
End in this state for the planned transition: [EXIT STATE].
```

Avoid long creative descriptions that invite the model to redesign the frame.
