# PISR AIGC Video Workflow

## Contents

1. Operating principle
2. Intake and scene selection
3. Still production
4. Motion planning
5. Video model preflight and Lovart execution
6. Motion review
7. HyperFrames assembly and transition preview
8. Delivery resolution
9. Recovery behavior

## 1. Operating principle

Treat video as a downstream extension of approved PISR still production. Do not put motion-generation rules inside Master, Imagen, or the still-image Lovart executor.

The target unit is a `scene_group`: one Source Reference, one model, four looks, four controlled poses, four silent clips, and one assembled video.

## 2. Intake and scene selection

Resolve two fields before planning:

- `delivery_mode`: `image_only`, `video_only`, or `image_and_video`.
- `video_scope`: `all`, `include`, or `exclude` with Source Reference paths or IDs.

Prefer explicit paths such as `Batch 5/Creative Industry/scene-03.jpg`. Convert each selected path into a stable `source_reference_id` and carry it through every still, clip, and final video record.

If the user invokes the video skill but does not specify scenes:

1. Inspect the available Source References or completed manifests.
2. Use all scenes only when the user's wording clearly implies the whole batch.
3. Otherwise ask one concise scene-selection question.

## 3. Still production

For each selected scene:

1. Lock the Source Reference, model identity, scene geometry, camera family, and light direction.
2. Create four distinct looks using the upstream product-priority rules.
3. Avoid repeating the same hero product unnecessarily unless its sales priority justifies additional exposure.
4. Give each still a small, controlled `pose_delta`; retain the same visual world.
5. Generate 9:16 only for video-only work unless another ratio is explicitly required.
6. Use the first accepted still as the strongest scene-and-identity anchor when later variants need additional consistency support.

Run the Replacement Gate per still before creating any derived ratio or motion asset. Then run the Still Quartet Gate across the full group.

After both internal gates pass, run the default Still Review Gate:

1. present all four key stills together;
2. identify the scene group and ordered Look IDs;
3. wait for explicit user approval;
4. record `pending_user`, `approved`, or `changes_requested`.

Do not start image-to-video generation while key-still approval is pending.

## 4. Motion planning

Plan the complete sequence before generating clips. For every still assign:

- `motion_preset`;
- `entry_state`;
- `exit_state`;
- `camera_motion`;
- `transition_in`;
- `transition_out`;
- target duration.

Choose motions that can connect. Example:

1. ambient pose, then slight turn;
2. continue the turn and touch garment;
3. one step toward camera;
4. settle, glance away, and exit frame edge.

Avoid four unrelated motions that force the editor to hide continuity problems with aggressive effects.

## 5. Video model preflight and Lovart execution

Default to Kling 3.0. Before submitting the first job, preflight the candidate model against the authoritative still and required subject type, input mode, 9:16 ratio, duration family, and silent-output behavior.

Use this fallback path:

1. Kling 3.0;
2. the current compatible Kling image-to-video replacement if the exact model is unavailable;
3. Seedance or another Lovart image-to-video model only when preflight confirms that subject-library, real-person, and input restrictions will not block the job.

Record rejected preflights and no-charge failures in `video-state.json`. After one clip succeeds, keep the remaining clips on the same model version unless that version cannot complete a specific clip.

Create one independent image-to-video job per still. Use independent Lovart threads so clips can run concurrently without sharing conversational context.

For each job:

1. Upload or reference exactly one authoritative still plus only the minimum required motion instruction.
2. State that the source frame controls identity, clothing, products, color, accessories, scene, lighting, and camera.
3. Ask only for the planned motion.
4. Disable generated music, dialogue, and ambient audio.
5. Use one consistent aspect ratio, frame rate, duration family, and model version across the group.

Use rolling concurrency:

- start jobs up to the currently safe Lovart limit;
- submit the next ready job whenever a slot completes;
- reduce concurrency on rate limiting or repeated transient failures;
- increase again only after a stable success window;
- keep retries clip-local.

## 6. Motion review

Sample the start, midpoint, and end frame. Compare them with the authoritative still and record:

- identity consistency;
- product and graphic consistency;
- color and accessory consistency;
- anatomy;
- scene and camera consistency;
- motion quality;
- transition compatibility;
- audio absence.

Classify each clip:

- `accepted`: ready for assembly;
- `retry`: a specific failure can be regenerated;
- `manual_review`: ambiguity requires user judgment.

Do not delete imperfect clips automatically. Preserve them as resumable evidence.

## 7. HyperFrames assembly and transition preview

After all required clips are accepted:

1. Record every clip's native resolution, frame rate, codec, pixel format, and color space.
2. Normalize the assembly format without automatic AI upscaling.
3. Trim clips to their useful motion windows.
4. Assemble the sequence as an editable HyperFrames composition.
5. Run the Transition Design Gate and choose one primary transition language.
6. Add one BGM track after visual assembly.
7. Align major cuts or transitions to musical beats where practical.
8. Preserve accepted clip color by default; apply only minimal correction when required.
9. Do not call GrainLab and do not simulate video grain.
10. Run HyperFrames lint/check and open a playable Studio preview.
11. Wait for user approval or requested transition adjustments.
12. Render the final MP4 only after preview approval, then verify duration, dimensions, playback, and audio.

Transition defaults:

- use short hard cuts or a restrained editorial treatment as the safe starting point;
- avoid long full-body crossfades that create double-subject ghosting;
- for light leak, use oversized warm overlays and hard-swap scenes under peak light coverage;
- keep one coherent transition language across the sequence and expose it for user adjustment at preview.

The default assembly step ends at Studio preview, not at an automatic final render.

## 8. Delivery resolution

Store native and delivery resolution separately:

```json
{
  "native_clip_resolution": "720x1280",
  "delivery_resolution": "1080x1920",
  "ai_upscale": false
}
```

Rendering a larger delivery canvas is ordinary resampling, not permission to use an AI upscaler. Do not regenerate or upscale automatically when a clip is below the preferred delivery resolution; report it and let the user decide.

## 9. Recovery behavior

Persist job IDs, thread IDs, input still paths, prompts, model-preflight attempts, key-still approval, status, output paths, transition profile, Studio preview status, native resolution, delivery resolution, and review decisions in `video-state.json`.

On resume:

- stop while key-still approval is pending;
- skip accepted clips;
- poll pending jobs before resubmitting;
- retry only rejected or failed clips;
- reopen or rebuild the Studio preview when clips, transition settings, or BGM change;
- render the final MP4 only after the current preview version is approved.
