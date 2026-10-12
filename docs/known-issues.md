# Known Issues

Items are classified by how much is actually known about them. The A series records things that
were checked and are not defects, B records things nobody has verified, C records defects and
design limitations, and D records documentation debt. An item is not deleted when it is
resolved: the classification is the point, because it shows what was checked and what was only
assumed.

Snapshot 2026-10-10 · 11 real-arm and 12 simulation task folders · 89 finished commands of the earlier planner

| ID | Issue | Class | Disposition |
|---|---|---|---|
| A-1 | No credential-like strings in any file of the repository | Not a defect | No action |
| A-2 | All three planners build and the offline checks pass | Not a defect | No action |
| A-3 | The three variants share an identical set of simulation scenes | Not a defect | No action |
| A-4 | Each variant folder is identical to its source branch | Not a defect | No action |
| A-5 | A missing Gemini key stops the figure design with a message | Not a defect | No action |
| A-6 | GitHub holds three output files and no experiment record | Not a defect | No action |
| B-1 | A physical outcome after a simulated pass has never been observed | Unverified | Open |
| B-2 | Unity and Isaac Sim parts were not opened or run during assembly | Unverified | Open |
| B-3 | No recorded run used Unity's own inverse kinematics for flat figures | Unverified | Open |
| B-4 | Most experiment records are on devices the owner cannot reach | Unverified | Open |
| C-1 | Build output with files over 20 MB is in the history | Defect | Fix recommended |
| C-2 | YOLO weights and the Python environment are not in the repository | Defect | Fix recommended |
| C-3 | Launcher, Unity and Isaac Sim hard-code one machine's paths and addresses | Defect | Fix recommended |
| C-4 | Text outside the six whitelisted function names is ignored without an error | Design limitation | Open |
| C-5 | Arm safety stops are recorded as infrastructure errors | Design limitation | Open |
| C-6 | The 0% variant has no attempt limit, so the within-ten rate is not native | Design limitation | Open |
| C-7 | In the 100% variant a switch lets the arm run after a failed flat-figure check | Design limitation | Open |
| C-8 | The 100% variant writes no result or metrics file | Design limitation | Open |
| C-9 | A failed figure design counts as a failed task with zero attempts | Design limitation | Open |
| C-10 | Reach limits are written in four places and differ between variants | Design limitation | Open |
| C-11 | The Isaac Sim server listens on all network interfaces without authentication | Security | Fix recommended |
| D-1 | `llm_motion_planner.md` describes a design most variants no longer have | Documentation | Fix recommended |
| D-2 | The 0% README misdescribes one file and its inverse kinematics | Documentation | Fix recommended |
| D-3 | `weekly_progress.md` is empty in every variant | Documentation | Fix recommended |
| D-4 | `PartC_LLM.docx` describes the earliest design | Documentation | Fix recommended |
| D-5 | Module ownership within the team is not recorded | Documentation | Open |
| D-6 | The recorded runs show reach figures that the current 50% code withholds | Documentation | Open |
| D-7 | Unreferenced and prototype code is shipped beside the live code | Documentation | Fix recommended |

---

### A-1　No credential-like strings in any file of the repository

**Check.** All blobs reachable from `main` under 3,000,000 bytes were searched for OpenAI-style
keys (`sk-…`), GitHub tokens (`gh?_…`, `github_pat_…`), Google API keys (`AIza…`), AWS key ids,
private-key headers, and assignments such as `api_key = "…"` with a value of 16 or more
characters. There were no matches among 1,501 blobs.

**Verified.** A one-off script run before publication, on 2026-10-10. A command with the same
patterns, slow on a large repository, is
`git grep -nIE 'sk-[A-Za-z0-9_-]{20,}|gh[opsu]_[A-Za-z0-9]{30,}|AIza[0-9A-Za-z_-]{35}' $(git rev-list --all)`.

**Assessment.** The 17 blobs of 3,000,000 bytes or more were not scanned: nine build files under
`csharp_server/bin/Debug` (see C-1), the ONNX model `csharp_server/models/yolo11n.onnx` and
seven 3D model or asset files. The search covers patterns, so a secret in an unusual format
would not be found. The exported run files in `data/` were read for the same patterns.

---

### A-2　All three planners build and the offline checks pass

**Verified.** `dotnet build` in each variant's `csharp_server` finished with no errors on
2026-10-10. `dotnet run --project tests/ExperimentChecks` printed `89 checks passed` for the 0%
variant and `94 checks passed` for the 50% variant. The 100% variant has no such project.

**Assessment.** This shows the code compiles and the covered logic behaves as its tests
expect. It says nothing about robot behaviour (see the README, Evaluation).

---

### A-3　The three variants share an identical set of simulation scenes

**Check.** The protocol says the scene files are identical across variants.

**Verified.** `git rev-parse HEAD:sim_scenes` inside each variant folder of the source clones
gives the same tree, `4c1747ffa736fe84c0f8b5d4b67a7b13ab470c55`, for all three.

---

### A-4　Each variant folder is identical to its source branch

**Verified.** The content of each folder in this repository is identical to the tip of its source
branch. Compared by tree object: `git rev-parse HEAD:50-partial-constraint` gives
`cdcd5214c8f59a8de50184e8fb71bd6020b58c82`, the tree of `origin/main`; the folder
`0-zero-constraint` gives `d25b6aa…` for `zero-constraint-bitmap-coverage` and
`100-rule-based` gives `7f85128…` for `rulebased_version`.

**Assessment.** No file content was changed when the three branches were put into one repository.

---

### A-5　A missing Gemini key stops the figure design with a message

**Check.** The planner starts without `GEMINI_API_KEY`, and the README of the 50% variant says to
switch pattern review off in that case. What happens when review is left on was not documented.

**Verified.** By reading the code on 2026-10-10, not by running it. With review on and no key,
`PatternDesigner.cs:81-83` and `SpatialPatternDesigner.cs:61-63` throw
`PatternDesignUnavailableException`, defined in `PatternDesign.cs:28` as a missing-configuration
error. It is not caught as a design failure, so the task ends as `infrastructure_error` with the
message that names the key and the switch.

**Assessment.** The failure is loud and does not count toward the success rate.

---

### A-6　GitHub holds three output files and no experiment record

**Check.** Whether run outputs were ever added to the original repository.

**Verified.** `git log --all --name-only --format= | sort -u | grep -iE '(^|/)outputs?/|result\.json|metrics\.json|/experiments(_sim)?/'`
in a clone with all branches and tags returns three paths: `csharp_server/outputs/detection_result.json`,
`csharp_server/outputs/visual_result.jpg` and `csharp_server/open_vocab/outputs/objects_open.json`.

**Assessment.** The run folders exist only in the owner's working copy, so `data/` holds the
only published copy of the figures that could be exported.

---

### B-1　A physical outcome after a simulated pass has never been observed

**Symptom.** Among the records available here, no task shows a plan passing simulation and then
running to completion on the arm.

**Observation.** All five real-arm tasks with a result ended as `infrastructure_error`, six have
no result, and none of the 27 recorded simulation attempts passed (`data/`).

**Hypothesis.** The verification flow is likely sound for flat figures, since the overlap and
the model verdict are computed on real renders, but nothing recorded tests it.

**Check.** Run a fixed list of tasks per variant with `FIXED_BASELINE=1`, record the final
table with a camera, and compare it with the simulated verdict.

---

### B-2　Unity and Isaac Sim parts were not opened or run during assembly

**Observation.** The C# planners were built and the offline checks run. The Unity projects
(`unity_project/`), the Isaac Sim scripts (`isaac_sim/`) and the perception server were not
opened, compiled or run, and no arm, URSim or camera was available.

**Assessment.** Their behaviour as documented here rests on reading the code, and the reading
was not uniform. Read in full: the planner of the 50% variant, the 0% and 100% files that
differ from it, `perception_server.py`, `UIManager.cs`, `RunMode.cs`, `LayeredGraspGeometry.cs`
and `UrSimIkClient.cs`. Read in part: `JsonExecutor.cs` (about 3,800 of 3,966 lines in the 50%
variant, plus the full diffs to the other two) and `isaac_sim_server.py` (about 1,100 of 1,951
lines: configuration, verification, overlap, the URSim follower, self-collision set-up and the
HTTP endpoints). Read by header and member list only: `SceneSyncer.cs`, `SyncGripper.cs`,
`RobotArm.cs`, `UR3eKinematics.cs`, `UR3eArmBuilder.cs`, the calibrator, the world-building and
calibration functions of the Isaac server, and the Isaac helper scripts. The 100% variant's
`Program.cs` was read for its live path (lines 1 to 560 and 1450 to 1810), not for its
unreachable block (D-7).

---

### B-3　No recorded run used Unity's own inverse kinematics for flat figures

**Observation.** In the 0% variant `unityIkFor2D` defaults to true
(`0-zero-constraint/unity_project/Assets/Scripts/JsonExecutor.cs:177`), so flat figures are solved
by Unity's kinematics and only stacked figures by URSim. The 0% README describes URSim for all.

**Hypothesis.** The default is the intended behaviour, since the field's tooltip says so. Whether
the 12 simulation runs (2026-10-01 and 2026-10-07) used it is not recorded in their folders.

**Check.** Start the 0% planner with a flat-figure instruction and read the Unity log line
beginning `[Unity-IK]` or `[URSim-IK]`.

---

### B-4　Most experiment records are on devices the owner cannot reach

**Symptom.** Only 23 experiment tasks (65 attempts) exist as files on the computer used to
assemble this repository.

**Observation.** The owner reports that the team ran about 500 experiments together on
several computers. That figure is an estimate and is not backed by any file here. The experiment
folders are excluded from version control (`.gitignore`, `**/csharp_server/outputs/`), and the
GitHub repository never held them (A-6), so each computer kept its own.

**Impact.** No success rate can be computed from the available records, and the failure causes
described in the README come from a small part of what was run. The missing records could confirm
or contradict any conclusion drawn from the available ones.

**Check.** Collect the `csharp_server/outputs/experiments` folders from every computer, run the
census commands of `docs/metrics.md` over them, and recompute `metrics.json`.

**Fix direction.** Export `result.json` and `attempt_result.json` files into the repository after
each session, or keep results in one shared place.

---

### C-1　Build output with files over 20 MB is in the history

**Symptom.** Pushing the history to GitHub produced a warning that
`OpenCvSharpExtern.dll` is 63.68 MB, above GitHub's recommended 50 MB.

**Cause.** `OpenCvSharpExtern.dll` was added on 2026-05-23 as part of
`csharp_server/bin/Debug/net8.0/`, which no current folder contains; the files remain in the
repository's stored history.
The largest blobs are the 66,774,016-byte `OpenCvSharpExtern.dll`, a 52,503,206-byte
`onnxruntime.xcframework.zip`, a 43,596,581-byte `onnxruntime.aar` and a 37,310,032-byte
`libonnxruntime.dylib`.

**Location.** `git rev-list --objects --all | git cat-file --batch-check='%(objecttype) %(objectsize) %(rest)' | awk '$1=="blob" && $2>20000000'`

**Impact.** A clone downloads these files although no current file needs them.

**Fix.** Remove `csharp_server/bin/` from all history with `git filter-repo --path-glob`
and force-push. This rewrites the stored history from the first affected point onward.

---

### C-2　YOLO weights and the Python environment are not in the repository

**Symptom.** A clean clone cannot start the perception server.

**Cause.** `perception_server.py:206` loads `yolo11n.pt` from the `csharp_server` folder. `.gitignore`
excludes `*.pt` (line 38) and `csharp_server/yolo11_env/` (line 58). The variant READMEs
list `models/pliers.pt`, but the code never loads it: `PLIERS_MODEL` is `None` at `perception_server.py:210`.

**Verified.** `git ls-files '*.pt' | wc -l` prints 0 in all three variants.

**Observation.** An ONNX export of the small model, `csharp_server/models/yolo11n.onnx`
(10.7 MB), is in the history, but not the `.pt` file the code loads.

**Impact.** The real-camera path cannot be reproduced from the repository alone. The planner,
the offline checks and pure-simulation runs do not need these files.

**Fix direction.** Publish the weights as a release asset and document a `requirements.txt` for
the environment.

---

### C-3　Launcher, Unity and Isaac Sim hard-code one machine's paths and addresses

**Symptom.** `start_real_robot.bat`, the Unity executor and the Isaac Sim server work only on the
machine they were written for.

**Cause.** The launcher sets the Isaac Sim interpreter to `D:\isaacsim\python.bat` and the URSim
address to `192.168.50.221` (`start_real_robot.bat:22-23`), expects a `yolo11_env` environment
inside `csharp_server` (line 39), and its comments and window titles refer to a folder named
`main`. Unity defaults the arm to `192.168.50.204` and URSim to `192.168.50.221`
(`JsonExecutor.cs:165,167`). The Isaac Sim server defaults the robot asset to a path under
`D:\isaacsim\` (`isaac_sim_server.py:78`).

**Impact.** Running elsewhere fails at the first missing path or an unreachable address. The
three variants ship the same files.

**Fix.** Read these values from environment variables and fall back to the current defaults.

---

### C-4　Text outside the six whitelisted function names is ignored without an error

**Symptom.** A plan can contain `go_home` or another function that never runs.

**Cause.** The adapter matches only `move_above`, `descend`, `grasp`, `release`, `lift` and
`wait` (`NaturalLanguagePlanAdapter.cs:12`); anything else in the text is never seen, so it is
neither executed nor rejected (`50-partial-constraint/docs/experiment_protocol.md:46`). The
validator, by contrast, allows `go_home`.

**Consequence.** The text the model wrote and the batch the arm runs can differ. The
documentation does not say that the model is told which functions were ignored.

**Fix direction.** Treat an unrecognised call as a translation failure, or report it in the
feedback for the next attempt.

---

### C-5　Arm safety stops are recorded as infrastructure errors

**Symptom.** Tasks that ended in a UR protective stop during descent are excluded from the
success rate.

**Observation.** Three descent stops in two real-arm tasks are recorded under
`infrastructure_error` (`docs/metrics.md`, Failure Causes).

**Assessment.** A stop during a planned descent can come from the plan (a wrong height, a
collision) as well as from the hardware, so the label may hide planner failures.

**Fix direction.** Add a failure class for controller stops and report it separately from
network or camera faults.

---

### C-6　The 0% variant has no attempt limit, so the within-ten rate is not native

**Symptom.** The 0% variant retries until success or a manual abort, while the primary metric in
`metrics.json` is success within ten attempts.

**Cause.** `0-zero-constraint/csharp_server/Program.cs:67,167` set no limit and loop without
bound; `ExperimentMetrics.cs` counts a success within ten attempts.

**Consequence.** A within-ten rate for the 0% variant has to be derived from the per-attempt
logs, and its totals are not directly comparable with the 50% variant's. The recorded
simulation runs show the other face of this: all 12 were stopped by hand, so none has a result.

**Fix direction.** Report both a capped and an uncapped rate for the 0% variant.

---

### C-7　In the 100% variant a switch lets the arm run after a failed flat-figure check

**Symptom.** With the Unity button "Unity驗證" set to off, a flat-figure plan whose overlap or
model verdict fails is still sent to the arm.

**Cause.** `VerificationSwitch.cs:22` reads `verification_enabled.txt`, and
`100-rule-based/csharp_server/Program.cs:1638-1644` logs the failure and continues when it is off.
The switch exists to run a control group. It defaults to on, and stacked figures ignore it.

**Impact.** A mistaken click turns off the gate that protects the arm for flat figures.

**Fix direction.** Require a confirmation, or make the control group simulation-only.

---

### C-8　The 100% variant writes no result or metrics file

**Symptom.** No success rate can be computed for the 100% variant.

**Verified.** `grep -c "ExperimentMetrics\|result.json" 100-rule-based/csharp_server/*.cs` is 0 for
every file. A command ends with a console message and a `done` step; the per-command folders
(`unity_<time>_<label>`, `isaac_<time>_<label>`) hold images and verdicts only.

**Consequence.** The three variants cannot be compared on the same metric from their own records.

**Fix direction.** Write `result.json` and update a metrics file in the same format as the others.

---

### C-9　A failed figure design counts as a failed task with zero attempts

**Symptom.** In the 50% variant a task whose target figure cannot be designed ends as `failed`
with `attempts` 0.

**Cause.** `50-partial-constraint/csharp_server/Program.cs:187-192` sets `maxAttempts` to 0 when
`PatternDesignException` is thrown, the loop is skipped, and `status` keeps its initial value
`failed` (line 85), which `counts_toward_success_rate` accepts (line 598).

**Consequence.** A failure of the figure-drawing models lowers the success rate of the planning
loop, which never ran.

**Fix direction.** Report design failures separately, or give them their own status.

---

### C-10　Reach limits are written in four places and differ between variants

**Symptom.** The same instruction can pass one variant's reach check and fail another's.

**Cause.** The validator allows X 0 to 0.72 m and Y 0 to 0.45 m (`MotionPlanValidator.cs:8,10`).
The 100% layout accepts targets 0.18 to 0.48 m from the base (`LayoutRealizer.cs:143-144`), its
task assigner sources 0.10 to 0.47 m (`TaskAssigner.cs:30-31`), and its Unity executor sources 0.10
to 0.47 m and targets up to 0.48 m (`JsonExecutor.cs:312,319,320`). The 0% and 50% executors use 0.20
to 0.42 m for sources and 0.16 to 0.47 m for targets (`JsonExecutor.cs:311,318,319,308`).

**Impact.** Results cannot be compared across variants without checking which band applied.

**Fix direction.** Put the limits in one shared file.

---

### D-1　`llm_motion_planner.md` describes a design most variants no longer have

**Symptom.** The file describes `TaskAssigner → MotionPlanner → MotionPlanValidator` with a
model that retries up to three times.

**Verified.** `git ls-files 'csharp_server/MotionPlanner.cs'` returns one file in the 100%
variant and none in the 0% and 50% variants. The text is the same in all three. In the 100%
variant the description matches `Program.cs:873`.

**Fix.** Mark the file as describing the 100% variant only, and describe the other flows in
`docs/architecture.md`.

---

### D-2　The 0% README misdescribes one file and its inverse kinematics

**Location.** `0-zero-constraint/README.md:47-48`. Both lines describe `ExperimentLlm.cs`; the
first calls it free decomposition, planning, verification and reflection, the second says the
model outputs execution JSON. The README also says every TCP pose is solved through URSim,
while `JsonExecutor.cs:177` makes Unity's own kinematics the default for flat figures, and a
comment above `bitmapOverlapThreshold` describes a cell-count overlap although the code uses the
pixel overlap (`RunBitmapCheck`, `passed = overlap > bitmapOverlapThreshold`).

**Fix.** Merge the two lines, state the Unity-IK default, and correct the comment. The README
elsewhere says this variant uses no reflection.

---

### D-3　`weekly_progress.md` is empty in every variant

**Verified.** `wc -c docs/weekly_progress.md` prints 2 in all three variants.

**Fix.** Remove the file or fill it in.

---

### D-4　`PartC_LLM.docx` describes the earliest design

**Observation.** The document describes a console planner with `Program.cs`, `RobotPlan.cs`
and `llm_planner.cs`, with two supported actions, and test data in place of the perception
output. It predates all three variants. No credential-like strings are in its text.

**Fix.** Label it as a historical note.

---

### D-5　Module ownership within the team is not recorded

**Observation.** The project had six contributors, but no file states who was responsible for which
module (perception, planner, Unity executor, Isaac Sim).

**Impact.** The README makes no per-module attribution and no contribution count as a result.

**Fix.** Add a short responsibilities table agreed by the team.

---

### D-6　The recorded runs show reach figures that the current 50% code withholds

**Observation.** The reflections of the real-arm runs quote "source radius 0.430 m (allowed
0.200 to 0.420 m)". The current 50% executor sends only "source … is outside the arm's reach"
to the model and keeps the figures in the Unity console (`JsonExecutor.cs:3983-4007`), while the
0% variant still sends them.

**Impact.** Evidence from those runs describes a model that saw the band, and the Problem
Statement quotes it as such. A reader may take it for the behaviour of the final 50% code.

**Fix.** Note the code version beside each recorded run when the runs are re-recorded.

---

### D-7　Unreferenced and prototype code is shipped beside the live code

**Observation.** Read from the call graph on 2026-10-10:
- `100-rule-based/csharp_server/Program.cs:695-1449` (755 lines) defines `RunPatternTaskAsync`,
  `RunSpatialPatternTaskAsync`, `RunSingleObjectTaskAsync` and `RunMultiStackTaskAsync`, which no
  live path calls; the live path uses the `…BatchAsync` functions.
- `0-zero-constraint/csharp_server/NaturalLanguagePlanAdapter.cs` and `LayoutRealizer.cs` are not
  referenced by the 0% planner; `Verifier.cs` is not referenced by the 0% or 50% planner and is
  compiled only into the offline checks.
- `isaac_sim/phase3_pick_place.py`, `stack_verifier.py` and `stack_verifier_with_robot.py` are
  early prototypes for a UR10 with a suction cup, with paths under `C:\Users\ASUS\isaac_env`; the
  live server is `isaac_sim_server.py`.

**Impact.** A reader cannot tell live code from leftovers without the call graph, and line counts
in this repository include them.

**Fix.** Delete the unreferenced code, or move it to a `legacy/` folder with a note.

---

### C-11　The Isaac Sim server listens on all network interfaces without authentication

**Symptom.** Any machine on the same network can call the Isaac Sim server's HTTP endpoints.

**Cause.** `isaac_sim_server.py` binds `--host` to `0.0.0.0` by default (line 77), with the help
text "so another computer can connect", and no endpoint checks a credential. The perception
server, by contrast, binds `127.0.0.1` (`perception_server.py:55`).

**Impact.** A caller can reset the simulated world (`/reset`, `/sim/load`), start or end a
verification (`/verify/begin`, `/verify/end`) and read frames. The server only reads URSim and
sends no command to the arm, so it cannot move the real arm, but it can make a verification fail
or pass on a changed world.

**Fix.** Default to `127.0.0.1`, and require the flag to expose it on a trusted network.
