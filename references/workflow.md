# PISR AIGC Video Workflow

## Contents

1. Operating principle
2. Intake and scene selection
3. Still production
4. Motion planning
5. Lovart execution
6. Motion review
7. Assembly and finishing
8. Recovery behavior

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

## 5. Lovart execution

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

## 7. Assembly and finishing

After all required clips are accepted:

1. Normalize resolution, frame rate, codec, pixel format, and color space.
2. Trim clips to their useful motion windows.
3. Assemble in the planned order.
4. Add restrained transitions that match adjacent exit and entry states.
5. Add one BGM track after visual assembly.
6. Align major cuts or transitions to musical beats where practical.
7. Apply one unified color treatment and video grain after assembly.
8. Export the final MP4 and verify duration, dimensions, playback, and audio.

Do not apply still-image grain before image-to-video generation because the motion model may animate the grain and create temporal flicker.

## 8. Recovery behavior

Persist job IDs, thread IDs, input still paths, prompts, status, output paths, and review decisions in `video-state.json`.

On resume:

- skip accepted clips;
- poll pending jobs before resubmitting;
- retry only rejected or failed clips;
- rebuild the final video only when an accepted clip, transition plan, BGM, or finishing setting changes.
