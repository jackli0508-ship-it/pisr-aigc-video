---
name: pisr-aigc-video
description: Opt-in planning and orchestration for PISR fashion image-to-video campaigns. Use when the user explicitly invokes this skill or explicitly asks to turn selected Source References, scene groups, or completed PISR stills into coordinated short-form video clips with consistent scenes, models, products, looks, motion, transitions, BGM, and final finishing. This skill is in development and must not change the default PISR image workflow unless activated.
---

# PISR AIGC Video

## Status and activation

Treat this skill as **In Development** and opt-in.

- Apply it only when the user explicitly invokes `$pisr-aigc-video` or explicitly requests video production.
- Do not alter `pisr-aigc-master` or the default image workflow merely because this skill is installed.
- Preserve all upstream decisions about Source Reference, model, products, look, placement, color, and accessories.
- Treat this skill as an additive video protocol, not a replacement for the image-production skills.

## Objective

Turn one selected PISR scene into a coherent short-form fashion video built from four coordinated stills:

- one scene;
- one model identity;
- four distinct looks;
- four controlled pose or action variations;
- four silent image-to-video clips;
- one assembled sequence with planned transitions, one BGM track, unified color, and final grain.

Default to a 9:16, 12–15 second social video with four clips of roughly 3–4 seconds each unless the user specifies otherwise.

## Read the appropriate references

- Read [references/workflow.md](references/workflow.md) before planning or executing a video job.
- Read [references/scene-group-schema.md](references/scene-group-schema.md) when creating or consuming manifests, selecting scenes, or handing off between image and video stages.
- Read [references/motion-library.md](references/motion-library.md) when designing clip actions, transitions, or prompt blocks.

## Choose an entry mode

### Planned video entry

Use this when image generation has not started.

1. Capture `delivery_mode` as `video_only` or `image_and_video`.
2. Resolve `video_scope` to all scenes, an explicit include list, or an explicit exclude list.
3. Identify each scene by its Source Reference path and a stable `source_reference_id`; do not rely on a visual nickname.
4. Ask the image-planning owner to create one `scene_group` per selected scene.
5. Require four still variants sharing the same scene and model while varying look and controlled pose delta.

For video-only work, default to four 9:16 stills and do not generate 1:1 copies unless requested.

### Existing-stills entry

Use this when completed images already exist.

1. Accept an explicit `scene_group_id`, a manifest, a group of named files, or one selected image whose metadata resolves to a scene group.
2. Recover lineage from structured manifests before filenames and never from visual guessing alone.
3. Verify that the selected stills belong to the same scene and model.
4. If fewer or more than four stills exist, select or request a deliberate sequence rather than silently inventing membership.

## Core workflow

1. Run the Delivery Intent Gate and Scene Selection Gate.
2. Create or recover the `scene_group` manifest.
3. Produce or collect four clean stills.
4. Run the Replacement Gate on every still before any animation.
5. Run the Still Quartet Gate across the group.
6. Assign one motion preset and one transition role to each still.
7. Plan clip order and transition compatibility before image-to-video generation.
8. Send each still to an independent Lovart image-to-video job with no generated audio.
9. Keep available Lovart concurrency slots filled with independent scene-group clips; reduce concurrency automatically on rate limiting.
10. Run the Motion Gate on each returned clip.
11. Regenerate only failed clips; do not rerun accepted clips or the entire scene group.
12. Normalize and assemble accepted clips in the planned order.
13. Add transitions, then one BGM track, then unified color and video grain.
14. Export the final MP4 and retain resumable state.

## Non-negotiable consistency rules

- Use each completed still as the only authoritative visual basis for its video clip.
- Do not redesign the face, hairstyle, body proportions, garment, product color, graphics, accessories, scene geometry, camera angle, or lighting.
- Animate motion, not identity or styling.
- Keep background motion subtle unless the Source Reference clearly implies stronger environmental movement.
- Do not create extra people, products, limbs, garments, props, text, or logos.
- Keep each clip silent. Add audio only after the four clips are assembled.
- Feed clean stills into image-to-video. Apply grain after assembly to avoid temporal crawling or flicker.

## Gates

### Replacement Gate

Confirm that every intended source garment or product has actually been replaced by the assigned PISR product. Reject unchanged reference products before animation.

### Still Quartet Gate

Confirm across the four stills:

- same model identity;
- same scene, camera family, lighting direction, and environmental layout;
- four distinct approved looks;
- correct product colors and graphics;
- no accidental hat, jewelry, bag, or accessory switching;
- only approved pose differences.

### Motion Gate

Inspect at least the start, middle, and end of every clip for:

- face or identity drift;
- garment, graphic, color, or accessory mutation;
- anatomy errors or new people;
- background reconstruction or camera discontinuity;
- motion that contradicts the planned transition;
- accidental audio.

Record `accepted`, `retry`, or `manual_review` per clip.

## Ownership and handoffs

- `pisr-aigc-master` owns Source Reference classification, model choice, product choice, look construction, placement, product priority, and image-backend routing.
- `pisr-aigc-lovart` or `pisr-aigc-imagen` owns still-image execution.
- `pisr-aigc-naming` owns authoritative product-derived filenames.
- This skill owns video intent, scene-group structure, motion design, clip order, transitions, BGM plan, Motion Gate, assembly plan, and final video lineage.
- Use the available Lovart execution capability for uploads, image-to-video calls, job state, retries, and downloads; do not duplicate credentials in this skill.

## Outputs and storage

When continuing the same campaign, reuse its task folder and keep video state under `run/video/`. Put final user-facing MP4 files in `results/`.

When animating unrelated existing stills as a distinct run, create a specific task folder under `/Users/tianyuli/Codex Projects/AIGC`, with `run/` for resumable state and `results/` for delivery.

Maintain these logical artifacts:

- `scene-groups.json`
- `video-plan.json`
- `motion-plan.json`
- `video-state.json`
- final `.mp4` files

Do not overwrite source stills.

## Development limits

- Do not claim the workflow is production-stable until it has passed real batch tests.
- Do not merge these rules into Master automatically.
- Do not publish this skill as a stable Index stage; label it `In Development` until the user promotes it.
- Prefer a one-scene prototype before broad batch execution when a new motion model, duration, or transition system is introduced.
