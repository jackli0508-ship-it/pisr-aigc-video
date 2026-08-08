---
name: pisr-aigc-video
description: Opt-in planning and orchestration for PISR fashion image-to-video campaigns. Use when the user explicitly invokes this skill or explicitly asks to turn selected Source References, scene groups, or completed PISR stills into coordinated short-form video clips with consistent scenes, models, products, looks, motion, transitions, BGM, final finishing, and a scene-grouped authoritative product text manifest. This skill is in Beta and must not change the default PISR image workflow unless activated.
---

# PISR AIGC Video

## Status and activation

Treat this skill as **Beta** and opt-in.

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
- one HyperFrames-assembled sequence with planned transitions, one BGM track, preserved accepted clip color, and a user-reviewable Studio preview before final export;
- one concise `video-products.txt` delivery manifest grouped by scene and look.

Default to a 9:16, 12–15 second social video with four clips of roughly 3–4 seconds each unless the user specifies otherwise.

## HyperFrames dependency gate

Run this gate before planning or executing the first `pisr-aigc-video` job on a computer.

1. Check whether the `hyperframes` entry skill is available in the current Codex skill catalog.
2. If it is missing, tell the user that the required HyperFrames dependency is being installed, then install the official published skill set:

   ```bash
   npx skills add heygen-com/hyperframes --all
   ```

3. After installation, read the installed `hyperframes` entry skill and follow its routing and lifecycle instructions. Verify the installation with:

   ```bash
   npx hyperframes skills check --json
   ```

4. If HyperFrames is present but its core skill set is incomplete or stale, refresh it with `npx hyperframes skills update`, then rerun the check.
5. Reuse an existing valid installation. Do not reinstall on every job, overwrite an existing HyperFrames project, or silently reconstruct HyperFrames behavior from memory.
6. If installation or verification fails, surface the exact error and stop before the assembly stage; retain completed stills and Lovart clips so the job can resume after HyperFrames is available.

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

1. Run the HyperFrames Dependency Gate, then the Delivery Intent Gate and Scene Selection Gate.
2. Create or recover the `scene_group` manifest.
3. Produce or collect four clean stills.
4. Run the Replacement Gate on every still before any animation.
5. Run the Still Quartet Gate across the group.
6. Run the default Still Review Gate: show all four key stills together and pause until the user approves them.
7. Assign one motion preset and one transition role to each approved still.
8. Plan clip order and transition compatibility before image-to-video generation.
9. Run the Video Model Preflight Gate. Default to Kling 3.0, then follow the compatible fallback path when it cannot accept the job.
10. Send each still to an independent Lovart image-to-video job with no generated audio.
11. Keep available Lovart concurrency slots filled with independent scene-group clips; reduce concurrency automatically on rate limiting.
12. Run the Motion Gate on each returned clip.
13. Regenerate only failed clips; do not rerun accepted clips or the entire scene group.
14. Normalize accepted clips without automatic AI upscaling and assemble them in HyperFrames.
15. Run the Transition Design Gate, add one BGM track, and create a playable Studio preview by default.
16. Export the final MP4 only after the user approves the Studio preview; retain resumable state and resolution lineage.
17. Invoke `pisr-aigc-naming` in video product manifest mode and write the approved scene and look product lists to `results/video-products.txt` before final delivery.

## Non-negotiable consistency rules

- Use each completed still as the only authoritative visual basis for its video clip.
- Do not redesign the face, hairstyle, body proportions, garment, product color, graphics, accessories, scene geometry, camera angle, or lighting.
- Animate motion, not identity or styling.
- Keep background motion subtle unless the Source Reference clearly implies stronger environmental movement.
- Do not create extra people, products, limbs, garments, props, text, or logos.
- Keep each clip silent. Add audio only after the four clips are assembled.
- Feed clean stills into image-to-video.
- Do not call GrainLab or simulate grain in the video workflow.
- Preserve accepted clip color by default. Apply color correction only when the user requests it or clip-to-clip mismatch requires a minimal correction.
- Do not use automatic AI upscale. Record native clip resolution separately from delivery resolution and let the user decide whether a low-resolution clip needs regeneration.

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

### Still Review Gate

After the Replacement Gate and Still Quartet Gate pass, present the four default key stills together and wait for explicit user approval before image-to-video generation. Treat this as the default workflow, including automation runs. Record `pending_user`, `approved`, or `changes_requested` in video state.

### Video Model Preflight Gate

Use Kling 3.0 as the default image-to-video model. Before submitting the first billed or time-consuming job, verify that the candidate accepts the authoritative still, subject type, aspect ratio, duration family, and silent-output requirement.

Use this fallback order:

1. Kling 3.0;
2. the current compatible Kling image-to-video replacement when the exact model is unavailable;
3. Seedance or another Lovart image-to-video model only after preflight confirms that subject-library, real-person, or input restrictions will not block the job.

Record failed preflight or no-charge attempts. Keep the four clips on one consistent model version once execution starts unless a clip-local failure makes that impossible.

### Motion Gate

Inspect at least the start, middle, and end of every clip for:

- face or identity drift;
- garment, graphic, color, or accessory mutation;
- anatomy errors or new people;
- background reconstruction or camera discontinuity;
- motion that contradicts the planned transition;
- accidental audio.

Record `accepted`, `retry`, or `manual_review` per clip.

### Transition Design Gate

Choose a transition profile before HyperFrames assembly and allow the user to revise it at the Studio preview. Use one primary transition language across the sequence rather than unrelated effects at every cut.

- Treat short hard cuts, brief directional blur, light leak, and restrained push/zoom treatments as available profiles.
- Use a plain crossfade only as a safe fallback. Avoid long crossfades when a full-body subject would appear twice.
- For light leak, swap scenes under the peak light coverage, then reveal the incoming look as the leak recedes.
- Do not use transitions to hide a failed Motion Gate.

### Studio Preview Gate

Assemble an editable HyperFrames composition and open a playable Studio preview by default. Run the required HyperFrames checks before handoff. Wait for user approval or transition adjustments before rendering the final MP4.

### Video Product Manifest Gate

After the final scene and clip order is locked, hand off authoritative scene-group, variant, and product metadata to `pisr-aigc-naming`. Require one `results/video-products.txt` file grouped by Scene and then Look. Keep every official product name in full, including accessories, and validate that the scene and look counts match the final export. Do not rename the `.mp4` from the complete product list and do not derive product names by inspecting rendered frames.

## Ownership and handoffs

- `pisr-aigc-master` owns Source Reference classification, model choice, product choice, look construction, placement, product priority, and image-backend routing.
- `pisr-aigc-lovart` or `pisr-aigc-imagen` owns still-image execution.
- `pisr-aigc-naming` owns authoritative product-derived image filenames and the scene- and look-grouped `video-products.txt` manifest.
- This skill owns video intent, scene-group structure, motion design, clip order, video-model preflight, transitions, BGM plan, Motion Gate, HyperFrames assembly plan, preview approval, and final video lineage.
- Use the available Lovart execution capability for uploads, image-to-video calls, job state, retries, and downloads; do not duplicate credentials in this skill.
- Use HyperFrames for editable assembly, transitions, Studio preview, validation, and final rendering.

## Outputs and storage

When continuing the same campaign, reuse its task folder and keep video state under `run/video/`. Put final user-facing MP4 files in `results/`.

When animating unrelated existing stills as a distinct run, create a specific task folder under `/Users/tianyuli/Codex Projects/AIGC`, with `run/` for resumable state and `results/` for delivery.

Maintain these logical artifacts:

- `scene-groups.json`
- `video-plan.json`
- `motion-plan.json`
- `video-state.json`
- HyperFrames composition files and Studio preview state
- final `.mp4` files
- `results/video-products.txt`

Do not overwrite source stills.

## Beta limits

- Do not claim the workflow is production-stable until it has passed real batch tests.
- Do not merge these rules into Master automatically.
- Do not publish this skill as a stable Index stage; label it `Beta` until the user promotes it again.
- Prefer a one-scene prototype before broad batch execution when a new motion model, duration, or transition system is introduced.
