# Authored worker characters

`worker-male.glb` and `worker-female.glb` are locally bundled, coherent humanoid worker meshes by Quaternius. Use the public URLs `/characters/worker-male.glb` and `/characters/worker-female.glb`. No runtime third-party asset requests are needed. See [ATTRIBUTION.md](ATTRIBUTION.md) before distributing, especially the female model's CC BY 3.0 credit requirement.

| Asset  | GLB bytes | Indexed triangles | Draw primitives | Materials | Images / textures |
| ------ | --------: | ----------------: | --------------: | --------: | ----------------: |
| Male   |   816,092 |             5,240 |              13 |        11 |             0 / 0 |
| Female |   922,440 |             6,112 |              12 |         9 |             0 / 0 |

The original creator-page triangle estimates are lower than the actual GLB index totals. The numbers above count each triangle in each primitive once. Models use flat-color PBR materials and have no morph targets. Materials currently have an exported metallic factor of 0.4; any application-specific roughness or metalness changes should apply to cloned materials.

## Integration

Each GLB has four mesh groups: `Worker_Body`, `Worker_Feet`, `Worker_Head`, and `Worker_Legs`. Three.js GLTFLoader produces one `SkinnedMesh` per material primitive under each group. Clone the scene with Three.js `SkeletonUtils.clone` for independent animated characters. Clone materials before customization; geometry can remain shared between instances. Play `CharacterArmature|Idle_Neutral` immediately for a front-facing, relaxed standing pose. The exported default pose already has rotated limbs and torso and is not a neutral bind pose.

Preserve the existing exported armature and mesh transforms: both use scale 100 and an X rotation of approximately -90 degrees. The loaded scene is upright, faces +Z in the neutral pose, and is approximately 1.85 m tall including its helmet. Do not apply a second axis conversion or multiply its size by 100. At neutral frame zero, bounds are approximately:

- Male: X [-0.264, 0.264], Y [-0.004, 1.854], Z [-0.100, 0.287] m.
- Female: X [-0.266, 0.255], Y [-0.002, 1.851], Z [-0.090, 0.271] m.

These are animated bounds, not collision dimensions. Walk/run clips move limbs outside them and may put a foot slightly below Y=0. Place the outer avatar group at floor level and use a separate character collision capsule. Use the model-specific animation clips: male and female rigs have the same names but different proportions and clip durations.

Hide the complete `Worker_Head` group for first-person rendering. Its children include the face, eyes, hair, and helmet. Its local position is zero, so a rule based on a top-level child's Y position being greater than 1.4 will not hide it. Parent attached helmet branding to the `Head` bone and chest branding to `Chest`, accounting for the bone's inherited scale. Keep those attachments hidden along with the head when needed. Preserve skinned-mesh parents; reparenting a primitive directly to a bone can change its skinning transform.

## Customization mapping

Identify a primitive by its parent group and material name, rather than its global material index. `Worker_Yellow` is shared by the helmet and reflective tape, so recoloring that shared material globally changes both. Clone the helmet primitive's material independently.

| Part            | Male                                                  | Female                                                       |
| --------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| Helmet          | `Worker_Head` / `Worker_Yellow` (`Worker_Head_1`)     | `Worker_Head` / `Worker_Yellow` (`Worker_Head_2`)            |
| Reflective tape | `Worker_Body` / `Worker_Yellow`                       | `Worker_Body` / `Worker_Yellow`                              |
| Vest            | `Worker_Body` / `Worker_Vest`                         | `Worker_Body` / `Worker_Vest`                                |
| Shirt           | `Worker_Body` / `LightBrown`                          | `Worker_Body` / `White`                                      |
| Skin            | all `Skin` primitives                                 | all `Skin` primitives, including ankle skin in `Worker_Feet` |
| Hair / eyebrows | `Worker_Head` / `Eyebrows`                            | `Worker_Head` / `DarkBrown`                                  |
| Facial hair     | `Worker_Head` / `Moustache` (can hide this primitive) | none                                                         |
| Eyes            | `Worker_Head` / `Eye`                                 | `Worker_Head` / `Brown`                                      |
| Trousers        | `Worker_Legs` / `Brown`, `Brown2`                     | `Worker_Legs` / `Brown_02`, `Brown2`                         |
| Boots           | `Worker_Feet` / `Black`, `Grey`                       | `Worker_Feet` / `Black`                                      |

Each skin references the same 62-joint hierarchy within that model. Important original glTF joint names are `Root`, `Body`, `Hips`, `Abdomen`, `Torso`, `Chest`, `Neck`, `Head`, `Shoulder.L`, `UpperArm.L`, `LowerArm.L`, `Wrist.L`, and their right counterparts, plus finger, thigh, shin, foot, and toe joints. Three.js sanitizes the dots away: use `ShoulderL`, `UpperArmL`, `LowerArmL`, `WristL`, `UpperLegL`, `LowerLegL`, `FootL`, and `PTL`, with equivalent R names.

## Animation and preparation

Both files retain exactly these original clip names:

- `CharacterArmature|Idle`
- `CharacterArmature|Idle_Neutral`
- `CharacterArmature|Interact`
- `CharacterArmature|Run`
- `CharacterArmature|Walk`
- `CharacterArmature|Wave`

The other 18 clips, including weapon and combat actions, were removed. The GLB JSON, remaining accessor list, and binary buffer views were compacted. There is no lossy compression or decoder dependency. All retained buffer-view bytes were compared with the original downloads, including vertex data, indices, inverse bind matrices, and retained animation samples. Nodes, materials, and skin joint lists were preserved.

Validation used the installed Three.js GLTFLoader and AnimationMixer in Bun. Both optimized files loaded successfully. Every skinned vertex matched the original model at four sample times in every retained animation. This validates source preservation and parsing; it is not a browser-render or Quest performance result.

[manifest.json](manifest.json) records source/binary URLs, source and bundled SHA-256 hashes, sizes, material names, kept and removed clips, and per-model metadata evidence paths. [sources/](sources/) preserves the creator-supplied metadata parsed from the exact Poly Pizza model pages, including their differing `Licence` fields and page hashes. Canonical legal-code text was downloaded from the linked Creative Commons license URLs (`legalcode.txt`). The current general Quaternius website license is not substituted for these older, explicitly licensed creator uploads.
