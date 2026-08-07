# Scene Group Schema

## Contents

1. Identity rules
2. Manifest example
3. Selection rules
4. Lineage rules

## 1. Identity rules

Use stable IDs rather than visual descriptions:

- `source_reference_id`: identifies the original scene or format reference.
- `scene_group_id`: identifies one video treatment of that reference.
- `variant_id`: identifies one still and look within the group.
- `clip_id`: identifies the image-to-video result derived from one variant.

Derive IDs from normalized batch-relative paths plus a short hash when duplicate names are possible. Never use absolute local paths as public IDs.

## 2. Manifest example

```json
{
  "schema_version": "0.1-development",
  "delivery_mode": "image_and_video",
  "video_scope": {
    "mode": "include",
    "source_reference_ids": [
      "batch05-creative-scene03"
    ]
  },
  "scene_groups": [
    {
      "scene_group_id": "batch05-creative-scene03-video01",
      "source_reference_id": "batch05-creative-scene03",
      "source_reference_path": "Batch 5/Creative Industry/scene-03.jpg",
      "subject_mode": "on-model",
      "model_id": "model-07",
      "target_ratio": "9:16",
      "target_duration_seconds": 14,
      "variants": [
        {
          "variant_id": "look01",
          "look_id": "look-creative-01",
          "product_ids": ["top-101", "bottom-204", "shoe-030"],
          "pose_delta": "relaxed front stance, slight left shoulder drop",
          "still_path": "results/scene03-look01.png",
          "clip_id": "scene03-look01-clip",
          "motion_preset": "ambient-breath"
        },
        {
          "variant_id": "look02",
          "look_id": "look-creative-02",
          "product_ids": ["top-118", "bottom-215", "shoe-042"],
          "pose_delta": "small torso turn toward camera right",
          "still_path": "results/scene03-look02.png",
          "clip_id": "scene03-look02-clip",
          "motion_preset": "micro-turn"
        }
      ]
    }
  ]
}
```

Production groups normally contain four variants. The shortened example shows two only for readability.

## 3. Selection rules

Support these inputs in descending authority:

1. explicit `scene_group_id`;
2. manifest-provided `source_reference_id`;
3. explicit Source Reference path;
4. a selected still whose manifest points to a scene group;
5. filenames only when the naming plan provides a verified mapping.

Do not group images solely because they look similar.

Support:

```json
{"mode":"all"}
```

```json
{"mode":"include","source_reference_ids":["scene01","scene04"]}
```

```json
{"mode":"exclude","source_reference_ids":["scene02"]}
```

## 4. Lineage rules

Every clip must reference exactly one authoritative still. Every final video must list its ordered clip IDs. Every clip must inherit the still's model and product list without modification.

Minimum clip record:

```json
{
  "clip_id": "scene03-look01-clip",
  "scene_group_id": "batch05-creative-scene03-video01",
  "variant_id": "look01",
  "authoritative_still": "results/scene03-look01.png",
  "model_id": "model-07",
  "product_ids": ["top-101", "bottom-204", "shoe-030"],
  "status": "accepted"
}
```

Do not change the still, model, product list, color, or accessories at the image-to-video stage.
