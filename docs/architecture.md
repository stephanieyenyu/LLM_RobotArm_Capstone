# Architecture

This project has no hosted services and no deployment target. It is a set of processes that run
on one workstation next to one arm. This document describes the data flow between them and the
one thing that differs between the three variants: how much of the plan the model writes.

Every statement below was read from the source code of the three variants on 2026-10-10 (the
files named in each section), not copied from the variant READMEs. Where the code and a README
disagree the code is followed and the disagreement is listed in `known-issues.md`.

![System pipeline shared by the three variants](images/architecture.svg)

The editable source of the figure is [diagrams/architecture.mmd](diagrams/architecture.mmd).

---

## Processes

| Process | Technology | Listens or connects | Responsibility |
|---|---|---|---|
| Perception server | Python, Flask, pyrealsense2, YOLO11n, OpenCV (`csharp_server/perception_server.py`) | `127.0.0.1:5000`: `/scene`, `/scene/mode`, `/health`, `/camera`, `/debug/frame`, `/debug/live` | Streams the camera, detects objects, returns positions in workbench coordinates |
| Planner | C# (.NET 8), `csharp_server/Program.cs` | Reads the scene over HTTP; calls OpenAI and Gemini | Turns an instruction into a verified batch; runs pre-execution checks and the simulation checks |
| Unity project | Unity 2022.3, `JsonExecutor.cs` | Arm on TCP 30002 (URScript); URSim on 30002 and 30004 | Instruction UI, kinematic batch preview, joint-target executor |
| Isaac Sim server | Python, Isaac Sim, `isaac_sim/isaac_sim_server.py` | HTTP on port 6000 on all interfaces (the planner's address is set with `ISAAC_SIM_URL`; known issue C-11); reads URSim on port 30013 | Physics twin of the table and arm for stacked figures; also the world for pure-simulation runs |
| Arm | UR3e or URSim | 30002 commands, 30004 RTDE, 30013 read-only state | Executes joint targets; URSim also answers inverse-kinematics queries |

The planner and Unity exchange work through files in `StreamingAssets`: `user_input.txt` carries
the instruction in, `current_step.json` carries the batch out and `step_done.json` carries the
result back. Other shared files are `run_mode.json` (real or simulation, and the scene),
`skip_pattern_review.txt` (`1` = draw the figure with one model only), `sim_check.json` with its
images (the flat-figure overlap report), and in the 100% variant `verification_enabled.txt`.

### Perception, in detail

- Colour 1920 by 1080 at 15 frames per second, depth 640 by 480, depth aligned to colour;
  detection every 0.2 s; smoothing over a 5-frame window.
- Yellow and black cubes (2.5 cm) and dominoes (2:1) are found by HSV thresholds, contour area,
  aspect ratio and solidity; two touching cubes are split by a watershed when it finds exactly
  two similar parts. YOLO11n serves a whitelist of seven COCO classes (cup, bottle, cell phone,
  book, mouse, keyboard, laptop), each with its own confidence floor. The pliers model is switched
  off in the code.
- ArUco markers 1 to 4 (0.073 m) define the workbench frame by `solvePnP` or by depth; the frame
  is cached when the markers are hidden.
- A cube or domino gets x and y from a ray intersected with the plane at 2.5 cm height and z from
  the depth image; other objects are placed from depth alone. In the 50% variant's copy (updated
  2026-10-10) the ray goes through the top-face centre, which is the centre of the colour mask
  corrected by the offset the visible side face causes, z is read at that point and takes the
  median of 8 frames, and the workbench frame is the median of the last 50 frames of the marker
  centres (a frame that deviates by more than 15 mm is ignored, and ten such frames in a row reset
  it). The 0% and 100% copies use the centre of the whole mask and a one-frame frame.
- Detections outside the polygon of the four markers, or with a box larger than half the polygon
  area, are dropped.

---

## Shared Flow

1. The user types an instruction in Unity. The planner reads it from `user_input.txt` and clears
   the file.
2. The planner reads `run_mode.json`. If the file exists but cannot be read, the command is
   refused rather than guessed, because a wrong guess could move the real arm.
3. In the real mode the planner waits until three consecutive scenes match one another (names,
   shapes, 15 mm in XY, 10 mm in Z, and domino direction), or, with `FIXED_BASELINE=1`, match the
   stored baseline. That scene becomes the task's `initial_scene.json`. In pure simulation the
   scene comes from a scene file and Isaac Sim, or from a built-in virtual world when Isaac Sim is
   not running.
4. One photo and the camera pose are taken. The camera is not read again for the task.
5. A target figure is chosen (see the table below) and the plan is produced at the variant's
   level of constraint.
6. Pre-execution checks reject plans that break the validator or overlap a block on the same
   layer (see Checks Before Execution).
7. The batch is verified in simulation:
   - **Flat figures.** Unity previews the whole batch with the arm moving only in the preview.
     The preview is kinematic: blocks are grasped when the simulated tool centre point is within
     20 mm of one at the right height (15 mm in the 100% variant) and are released straight down
     onto the nearest support.
     A top-down orthographic render of the blocks, 1 mm per pixel, is compared with the expected
     cells. Overlap is the sum of per-pixel minima of layer counts divided by the sum of maxima,
     and must be strictly greater than 0.9. A separate call to the same model, shown the Unity
     main-camera view, must then answer PASS (the 100% variant asks up to three times and takes
     the majority).
   - **Stacked figures.** Unity forwards the same joint trajectory to URSim without previewing.
     Isaac Sim follows URSim with physics. Isaac then checks that blocks were lifted, position
     (20 mm), height (8 mm), tilt (10 degrees), unmoved neighbours (10 mm), stability over one
     second (2 mm), fingertip clearance above the table (3 mm), arm self-collision and
     completeness of the URSim data, and computes the overlap of a height map after aligning the
     figure by up to 5 mm, which must again be strictly greater than 0.9. A separate call to the
     same model, shown the Isaac view, must answer PASS.
8. If every check passes, the same batch is sent again, now to the real arm (or, in pure
   simulation, finished). If any check fails, the arm does not move. On the real arm, an optional
   contact-stop (`contactStopOnPlace`, off in the code, set on in the 100% scene only) replaces the
   last 10 mm of a stacked placement descent with a slow descent at 10 mm/s that stops when the
   vertical force exceeds 8 N.
9. After execution nothing re-photographs the table. If the arm fails part-way, the task ends
   (0% and 50%).

### Checks before execution

- `MotionPlanValidator` (all variants): source and target inside X 0 to 0.72 m and Y 0 to 0.45 m
  with 25 mm tolerance; transfer distance at most 0.70 m; a target that overlaps another block
  needs a safe stacking height; `height_m` within 0.05 to 0.15 m and above a clearance computed
  from the tallest block; at most 20 calls; a state machine that rejects a second `grasp` while
  holding and `go_home` while holding or at contact height.
- Same-layer overlap of a placement with another block (0% and 50%).
- A source already within 20 mm of its target is not moved (0% and 50%).
- In Unity, before any joint target is sent: inverse kinematics must have a solution, joint
  limits and singularities are checked every 2 degrees along each segment, collision with the
  table and between links uses capsules (flat figures) or the arm's own mesh (stacked figures),
  and the base must face the target. In the 0% and 50% executors a source must lie 0.20 to 0.42 m
  and a target 0.16 to 0.47 m from the arm base; the 100% executor allows 0.10 to 0.47 m and up
  to 0.48 m.

---

## What Differs by Variant

| Decision | 0% zero-constraint | 50% partial-constraint | 100% rule-based |
|---|---|---|---|
| Understanding the instruction | Decomposition call; the model writes "needs flat bitmap" or "needs stacked bitmap" on its own line | Decomposition call; a separate call decides flat or stacked, with no keyword rules | A router call classifies it as `arrange_pattern`, `arrange_3d_pattern`, `move_relative` or `stack` |
| Target figure | Two-model bitmap, used to score the result; the model declares its own `bitmap_grid` origin and spacing | Two-model bitmap; the plan's figure must equal it cell for cell | Two-model bitmap, mapped to fixed coordinates by `LayoutRealizer` (cell 0.052 m) |
| Choosing source blocks | Model | Model | Code (`TaskAssigner`: far targets first, nearest block in the supply zone) |
| Choosing placement coordinates | Model | Model, written in its plan text | Code |
| Writing the motion | Plan text, then a third model call outputs JSON with a TCP pose for every move | Plan text; a deterministic adapter reads the source index, target and six function names | A model call per step returns a function sequence under a JSON schema, validated, up to 3 tries |
| Inverse kinematics | Unity's own for flat figures, URSim (RTDE registers 24 to 29) for stacked | Unity's own, from fixed orientations | Unity's own, from fixed orientations |
| On failure | Independent retry, no limit, no reflection | Up to ten attempts; failure summary and provisional rules carried forward | The command ends; no retry |
| Records | `result.json`, `metrics.json`, task folders | `result.json`, `metrics.json`, task folders | Folders of simulation images only |

The 100% variant also has a switch in Unity ("Unity驗證") that turns the flat-figure check
into a log entry only. With it off the arm runs after a failed check; stacked figures are
unaffected. It defaults to on.

---

## Records

In the 0% and 50% variants each task writes a folder under `csharp_server/outputs/experiments/`
(`experiments_sim` in pure simulation) with the task text, the configuration, the initial scene,
every system and user prompt, the raw model text, the translated plan, scenes and images, token
usage, `feedback.txt` and `attempt_result.json` per attempt, and a `result.json`. After each
task `metrics.json` is rewritten with trial counts and success rates. `result.json` records one
of `success`, `failed`, `infrastructure_error`, `execution_unknown` and `adapter_error`, and only
`success` and `failed` count toward the success rate. A failed figure design in the 50% variant
ends as `failed` with zero attempts. These folders are excluded from version control; the
exported ones are in `data/`.

---

## Cost of This Structure

The process split follows what each part needs from its platform: the camera libraries are
Python, the physics twin runs in Isaac Sim's own interpreter, and robot control and the visual
preview live in Unity. The accepted costs are these.

- Work is handed over by polling files, so a task has no transaction around it. If the planner
  stops between writing a batch and Unity reading it, nothing reconciles the two. The 0% and
  50% planners then wait with no timeout; the 100% planner gives up after 600 s per step.
- Five processes must be started in the right order. `start_real_robot.bat` starts three of
  them but carries one machine's paths and addresses (known issue C-3), and Unity carries default
  addresses of its own.
- The simulations are more optimistic than the arm. The flat-figure preview has no controller and
  no physics, and the stacked-figure twin lands about 2 mm from planned positions.
- Reach limits are written in four places (planner layout, task assigner, validator, Unity), and
  the numbers differ between variants (known issue C-10).
- A fixed task starting table needs `FIXED_BASELINE=1` and a person to restore the blocks by hand;
  nothing moves blocks back automatically.
