# Metrics

Every number in the README comes from one of the sources below. The purpose of this file is to
let a reader check each figure, not to ask them to accept it. Each table is preceded by the
command or query that produces it. Commands run from the repository root unless the Derivation
column names another location; "original repository" means
`https://github.com/stephanieyenyu/LLM_RobotArm`, "working copy" means the owner's local
folders listed under Sources.

**Snapshot date** 2026-10-10  
**Observation window** earlier planner 2026-07-04 to 2026-09-17 · real-arm experiments 2026-09-29 to 2026-10-01 · simulation experiments 2026-10-01 to 2026-10-07  
**Raw export** [data/](../data), with [runs-census-by-date.csv](../data/runs-census-by-date.csv) for the census and the run exports listed under Sources

---

## Sources

| Source | What it contains |
|---|---|
| Original repository on GitHub | Searched for tracked output files; the three branches used here are `main`, `zero-constraint-bitmap-coverage` and `rulebased_version` |
| Working copy `LLM_RobotArm/csharp_server/outputs/experiments` | 11 real-arm task folders (2026-09-29 to 2026-10-01), 5 `result.json`, one `metrics.json`, about 20 MB |
| Working copy `zero constraint LLM/zero constraint LLM/csharp_server/outputs/experiments_sim` | 12 pure-simulation task folders of the 0% variant (2026-10-01 and 2026-10-07), 802 files |
| Working copy `LLM_RobotArm/csharp_server/outputs` (top level) | 13 earlier-planner files from 2026-07-04 to 2026-09-17 |
| Working copy `LLM_RobotArm_old/csharp_server/outputs` | 640 earlier-planner files: 538 step files and 102 bitmap designs, copied on 2026-09-18 |
| Second copy `全部都在這/LLM_RobotArm/csharp_server/outputs` | 4 earlier-planner files from 2026-09-03 (1 batch, 3 steps) |
| `<variant>/` source code | Every constant quoted in the README, with file and line |
| `<variant>/tests/ExperimentChecks` | Offline checks, run on 2026-10-10 |

The sources are the records on one computer. The team ran far more experiments (the owner
estimates about 500, run together on several computers), and the records of those are on
devices the owner cannot reach. No figure below counts them, and none of the figures below is the
number of experiments run.

Only small files of the run folders were exported to `data/`: the five real-arm `result.json`
files, the aggregate, one `feedback.txt`, and for the simulation runs a task index, an attempt
table and the twelve `config.json` files. Prompts, model replies, scenes and images stay in the
owner's working copy and are excluded from version control by `.gitignore`.

---

## Scale

| Figure | Value | Derivation |
|---|---|---|
| Output files ever tracked on any GitHub ref | 3 | Original repository: `git log --all --name-only --format= \| sort -u \| grep -iE '(^\|/)outputs?/\|result\.json\|metrics\.json\|/experiments(_sim)?/'` |
| Planner C# files, 0% / 50% / 100% | 21 / 21 / 20 | `git ls-files '<variant>/csharp_server/*.cs' \| grep -vE '/(bin\|obj)/' \| wc -l` |
| Planner C# lines, 0% / 50% / 100% | 4,764 / 4,890 / 5,780 | The same list piped through `xargs cat \| wc -l` |
| Perception server files | 1 per variant | `git ls-files '<variant>/csharp_server/*.py' \| wc -l` |
| Isaac Sim scripts, 0% / 50% / 100% | 8 / 8 / 5 | `git ls-files '<variant>/isaac_sim/*.py' \| wc -l` |
| Unity scripts, 0% / 50% / 100% | 16 / 15 / 15 | `git ls-files '<variant>/unity_project/Assets/Scripts/*.cs' \| wc -l` |
| Unity script lines, 0% / 50% / 100% | 8,051 / 7,878 / 7,780 | The same list piped through `xargs cat \| wc -l` |
| Offline checks, 0% / 50% | 89 / 94 | `dotnet run --project <variant>/tests/ExperimentChecks`, last line |
| Experiments run by the team in total | about 500 | Not derivable from this repository: the owner's estimate, with the records on other devices |
| Experiment tasks recorded here, real arm and simulation | 23 | The two rows below, summed |
| Real-arm task folders | 11 | `tail -n +2 data/experiments/task-index.csv \| wc -l` |
| Simulation task folders (0% variant) | 12 | `tail -n +2 data/experiments_sim_0pct/task-index.csv \| wc -l` |

The sums in the README (62 planner files, 3 perception servers, 21 Isaac scripts, 46 Unity scripts) are plain
additions of the per-variant rows and do not count distinct code, since each variant ships its
own copy.

---

## Experiment Run Census

The census counts every experiment record found on the owner's computer on 2026-10-10, whatever
the planner version that wrote it. The search covered the Desktop, Documents, Downloads and
OneDrive folders for directories named `outputs`, `experiments`, `experiments_sim`,
`attempt_01`, `unity_2026*` and `isaac_2026*` and for every `task.txt`. It found four
`outputs` folders of this project, which are the rows below, and one `outputs` folder of an
unrelated project (`素材/Juvaly`), which was excluded. Two tiers are counted separately because
they are different things.

**Tier 1, formal experiment tasks.** One task is one instruction run through the free-planning
experiment pipeline, and it has its own folder with a `task.txt`.

Query: `ls -d */ | wc -l` in each experiment folder for tasks, and `ls -d */attempt_* | wc -l` for
attempts.

| Store | Tasks | Attempt folders | Dates |
|---|---|---|---|
| `LLM_RobotArm/csharp_server/outputs/experiments` (real arm) | 11 | 26 | 2026-09-29 (5 tasks), 2026-09-30 (3), 2026-10-01 (3) |
| `zero constraint LLM/…/outputs/experiments_sim` (0% variant, simulation) | 12 | 39 | 2026-10-01 (8), 2026-10-07 (4) |
| Total | 23 | 65 | four dates |

**Tier 2, commands of the earlier planner.** Before the experiment pipeline the planner wrote
`step_<unix seconds>.json` for every step it sent to Unity, a final step with `"done": true` when
a command finished, `batch_<unix seconds>.json` when it sent a whole batch, and a
`pattern_*.json` for every bitmap it designed. The timestamps in the names give the real dates;
the file-system dates of the 640-file folder are only its copy date (2026-09-18).

Query, run in each earlier-planner folder, then summed over the three folders:
`python -c "import glob,json;print(sum(json.load(open(f,encoding='utf-8'))['done'] for f in glob.glob('step_*.json')))"`
for finished commands, and `ls step_*.json batch_*.json pattern_*.json | wc -l` for files. The
per-date counts are in [runs-census-by-date.csv](../data/runs-census-by-date.csv).

| Quantity | Value | Reading |
|---|---|---|
| Finished commands (`done` steps) | 89 | 86 with a dated id, 3 with ids 1 to 25 that carry no date |
| Pick-and-place steps | 456 | 435 dated, 21 undated |
| Batches sent | 4 | 2026-09-03 (6 steps), 2026-09-16 (9), 2026-09-17 (13 and 12) |
| Bitmap designs saved | 106 | Lower bound: a design reusing an id overwrites its file |
| Detection result and robot plan | 2 | 2026-07-04, one command |
| Dates with at least one record | 11 | 2026-07-04, 08-12, 08-25, 08-26, 08-27, 08-28, 09-02, 09-03, 09-04, 09-16, 09-17 |

Finished commands per date, from the same CSV:

| Date | Finished commands | Pick-and-place steps |
|---|---|---|
| 2026-08-25 | 19 | 82 |
| 2026-08-26 | 20 | 112 |
| 2026-08-27 | 12 | 52 |
| 2026-08-28 | 27 | 127 |
| 2026-09-02 | 1 | 0 |
| 2026-09-03 | 2 | 1 |
| 2026-09-04 | 1 | 61 |
| 2026-09-16 | 2 | 0 |
| 2026-09-17 | 2 | 0 |
| Undated | 3 | 21 |

All dates together, the project has dated records on 15 dates, from 2026-07-04 to 2026-10-07.
The runs of 2026-08-25 to 2026-08-28 account for 78 of the 89 finished commands, in 16
sessions (a session ends after a gap of 30 minutes in the step ids).

**What this census does not claim.** A finished command is not a successful one: the `done`
step is written when a command ends, whatever its result, and a command that crashed before
writing it is not counted. Runs that left no file, for example a command typed while the
planner was not running, are not counted. Neither are the experiments run on other computers: the
owner reports about 500 experiments done together by the team on several devices, whose
records are not reachable, so the census is a small part of what was run. The second copy under
`全部都在這`, with files of 2026-09-03, is one visible trace of such a device. Steps are counted per
file, so a step file overwritten by a later step with the same id is lost. The earlier-planner
files carry no outcome, so none of these 89 commands is known to have succeeded or failed.

---

## Recorded Runs, Real Arm

Query: `cat data/experiments/task-index.csv data/experiments/metrics.json`. The model-call and
token figures come from the `*.usage.json` files in the working copy:
`python -c "import json,glob;print(sum(json.load(open(f))['InputTokenCount'] for f in glob.glob('csharp_server/outputs/experiments/*/attempt_*/*.usage.json')))"`.

| Quantity | Value | Reading |
|---|---|---|
| Task folders | 11 | Letters C (5 folders on 2026-09-29) and L (6 folders, 2026-09-30 and 2026-10-01) |
| Attempt folders | 26 | Sum of the third column of the index |
| Folders with `result.json` | 5 | All `infrastructure_error` |
| Folders without `result.json` | 6 | Outcome not recorded |
| `valid_trials` in `metrics.json` | 0 | Undefined success rates are stored as null |
| Decomposition / plan calls | 25 / 25 | Counted from `decomposition.usage.json` and `operation_plan.usage.json` |
| Translation / validation / reflection calls | 16 / 18 / 18 | Counted the same way |
| Input / output tokens | 394,461 / 469,424 | Sum of the usage files |

**What these figures do not claim.** The folders predate the 2026-10-01 switch to the local
translator, the 2026-10-08 unified verification flow and the 2026-10-10 changes to calibration,
grasp depth and perception, and the prompt still defined a placement zone. No row says that any variant succeeded or failed at a task. The six folders
without a result file are not in `metrics.json`; they are unknown, not failures.

---

## Recorded Runs, Simulation of the 0% Variant

Query: `python -c "import csv,collections;r=list(csv.DictReader(open('data/experiments_sim_0pct/attempts.csv',encoding='utf-8')));print(collections.Counter(x['outcome'] for x in r))"`.

| Quantity | Value | Reading |
|---|---|---|
| Task folders | 12 | Letters L (5), X (3), M (1), N (1), P (2); scene `letter_blocks.json`, which no longer exists |
| Attempt folders | 39 | |
| Attempts with a recorded result | 27 | Each folder's last attempt has none: the run was stopped by hand |
| Attempts that succeeded | 0 | |
| Attempts ending in `execution` | 15 | See the next table |
| Attempts ending in `precheck` | 9 | 8 for a position outside the workspace, 1 for a letter taller or wider than 5 cells |
| Attempts ending in `translation_rejected` | 3 | Each: execution path with more pick-and-place groups than explicit source/target pairs |
| Task folders with `result.json` | 0 | All stopped before a result was written |
| Task folders with `reflection: false`, `unbounded_attempts: true` | 12 | From `configs/*.config.json` |
| Decomposition / plan / validation calls | 32 / 31 / 27 | From the usage files |
| Input / output tokens | 291,780 / 454,489 | Sum of the usage files |

The 15 `execution` failures, grouped by the first words of the recorded text:

| Cause as recorded | Attempts |
|---|---|
| Arm path check rejected the plan before the preview, self-collision | 5 |
| Arm path check rejected the plan, no reachable collision-free travel path | 4 |
| UR not connected | 4 |
| Unity restarted and discarded the pending command | 2 |

**What these figures do not claim.** The runs used a code snapshot from before the current
verification flow, in which a 2D batch was previewed and executed within one stage and the
result was judged afterwards (`global_validation` was still called). Zero successes in 27 recorded
attempts says nothing about the current code. In 6 of the 15 `execution` failures the cause is the
setup, not the plan: a UR that was not connected, or a Unity restart.

---

## Failure Causes in the Real-Arm Reflection Texts

Query, run from `data/experiments`:

```
python -c "import json,glob
for f in sorted(glob.glob('*/result.json')):
    r=json.load(open(f,encoding='utf-8'))['final_rules']
    print(f[:15],len(r),
          sum('安全停止' in x and 'descend' in x for x in r),
          sum('來源半徑' in x for x in r),
          sum('safe stacking height' in x for x in r),
          sum('UR 未連線' in x for x in r))"
```

Columns are folder, reflection entries, descent safety stops, source-radius rejections,
stacking-height rejections, UR-not-connected.

| Task folder | Entries | Descent stops | Radius | Stacking | Not connected |
|---|---|---|---|---|---|
| 20260929_023011 | 0 | 0 | 0 | 0 | 0 |
| 20260930_045846 | 1 | 1 | 0 | 0 | 0 |
| 20261001_053242 | 0 | 0 | 0 | 0 | 0 |
| 20261001_054533 | 1 | 0 | 0 | 0 | 1 |
| 20261001_055255 | 5 | 2 | 2 | 1 | 0 |

**What these counts do not claim.** The texts are written by the model after each attempt. They
quote system messages, such as the radius limits and the stacking-height requirement, but they
also contain guesses. The counts are of phrases in that text and not of independently logged
events. Task 20261001_055255 ran six attempts and holds five entries, so one attempt left no
reflection. The radius figures (0.430 m and 0.431 m against 0.200 to 0.420 m) were part of the
message the model saw in this snapshot; the current 50% code withholds them (known issue D-6).

The pre-execution rejection `height_m must be 0.05..0.15` that the README mentions is in
`data/experiments/20260929_030552_…/attempt_01/feedback.txt`, line 1.

---

## Early-Era Output Files

Query, in the working copy: `find LLM_RobotArm_old/csharp_server/outputs -type f | sed 's/.*[\\/]//; s/_.*//' | sort | uniq -c`, and the same for
`LLM_RobotArm/csharp_server/outputs` excluding the `experiments` folder.

| Location | Files | Content |
|---|---|---|
| `LLM_RobotArm/csharp_server/outputs` (top level) | 13 | 4 `pattern_*.json`, 4 `step_*.json`, 3 `batch_*.json`, `detection_result.json`, `robot_plan.json`; 2026-07-04 to 2026-09-17 |
| `LLM_RobotArm_old/csharp_server/outputs` | 640 | 538 `step_*.json`, 102 `pattern_*.json`; all dated 2026-09-18 |

These are plans and bitmaps of the early rule-based planner, not experiment trials. They carry no
success label, and no figure in the README uses them.

---

## Tracked Output Files on GitHub

Query: the `git log --all --name-only` command in the Scale table.

| Path ever tracked | Kind |
|---|---|
| `csharp_server/outputs/detection_result.json` | One detection result |
| `csharp_server/outputs/visual_result.jpg` | One annotated image |
| `csharp_server/open_vocab/outputs/objects_open.json` | One open-vocabulary detection result |

No experiment folder, `result.json` or `metrics.json` was ever added to any branch or tag.
The recorded runs exist only in the owner's working copy, which is why `data/` holds an export.

---

## Constants Read from the Code

Each value was read from the file and line named; check with `sed -n '<line>p' <file>`. Paths
are relative to the repository root. "Unity" is
`50-partial-constraint/unity_project/Assets/Scripts/JsonExecutor.cs`; the 0% copy holds the same
constants a few lines further down, because it adds code above them.

| Figure | Value | File and line |
|---|---|---|
| Stable frames before a task starts | 3 | `50-partial-constraint/csharp_server/Program.cs:148` |
| Attempts per task, 50% | 10 | `50-partial-constraint/csharp_server/Program.cs:179` |
| Attempts per task, 0% | unlimited | `0-zero-constraint/csharp_server/Program.cs:67` |
| Operations per batch / functions per operation | 50 / 20 | `Program.cs:239`; `NaturalLanguagePlanAdapter.cs:41` |
| Table-reset tolerance XY / Z | 15 mm / 10 mm | `ExperimentChecks.cs:10-11` |
| Source tracking radius | 35 mm | `ExperimentChecks.cs:6` |
| Validator workspace, X / Y maximum | 0.72 m / 0.45 m, plus 25 mm tolerance | `MotionPlanValidator.cs:8,10,14` |
| Validator maximum transfer distance | 0.70 m | `MotionPlanValidator.cs:17` |
| Validator `height_m` range | 0.05 to 0.15 m | `MotionPlanValidator.cs:100` |
| Target-figure rounds / vote weights / single-candidate minimum | 2 / 0.80 and 0.20 / 0.50 | `PatternDesigner.cs:10,11,14` |
| Gemini request timeout | 150 s | `PatternDesigner.cs:52` |
| Letter size limit | 5 by 5 cells | `FigureBitmap.cs:12` |
| Unity base exclusion, source / target | 0.20 m / 0.16 m | Unity `:311`, `:308` |
| Unity maximum reach, source / target | 0.42 m / 0.47 m | Unity `:318`, `:319` |
| Overlap threshold, strictly greater than | 0.9 | Unity `:280`, `:1778` |
| Unity motion timeout / URSim safety-stop wait | 360 s / 300 s | Unity `:324`, `:330` |
| Default arm / URSim addresses in Unity | 192.168.50.204 / 192.168.50.221 | Unity `:165`, `:167` |
| Fingertip length / table correction | 0.179 m / −0.030 m | `LayeredGraspGeometry.cs:12,16` |
| Layer height / grasp depth below the top | 0.025 m / 0.0125 m (0.019 m until 2026-10-10) | `LayeredGraspGeometry.cs:18,24` |
| Contact-stop: enabled by default / approach / speed / force | false / 0.010 m / 0.010 m/s / 8 N | Unity `:262,264,266,268` |
| Workbench-frame history / height history, 50% perception | 50 frames / 8 frames | `perception_server.py:155,149` |
| Isaac position / height / tilt tolerance | 20 mm / 8 mm / 10° | `isaac_sim_server.py:184-186` |
| Isaac knocked / stability / fingertip-table tolerance | 10 mm / 2 mm / 3 mm | `isaac_sim_server.py:187,188,193` |
| Isaac overlap threshold / alignment search | 0.9 / ±5 mm | `isaac_sim_server.py:189,191` |
| Isaac default gripper opening / block mass | 0.035 m / 0.015 kg | `isaac_sim_server.py:97,155` |
| Camera colour resolution / frame rate / detect interval | 1920 by 1080 / 15 / 0.2 s | `perception_server.py:42,46,54` |
| Perception port / stability window / marker size | 5000 / 5 frames / 0.073 m | `perception_server.py:56,111,139` |
| 100%: supply zone, X / Y | 0.05 to 0.35 m / 0.02 to 0.25 m | `100-rule-based/csharp_server/TaskAssigner.cs:21-24` |
| 100%: figure cell size / canvas | 0.052 m / 5 by 5 | `100-rule-based/csharp_server/LayeredTypes.cs:50,53,54` |
| 100%: motion-plan retries / bitmap compaction retries | 3 / 2 | `100-rule-based/csharp_server/Program.cs:873,254` |
| 100%: Unity step timeout | 600 s per step | `100-rule-based/csharp_server/Program.cs:93` |
| 100%: reach limits in Unity, source / target | 0.10 to 0.47 m / up to 0.48 m | `100-rule-based/unity_project/Assets/Scripts/JsonExecutor.cs:312,319,320` |
| 100%: image-judge votes | up to 3, majority | `100-rule-based/csharp_server/SimulationImageJudge.cs:59` |

Rows without a variant prefix refer to `50-partial-constraint/` for `csharp_server` and
`unity_project` paths and to `isaac_sim/` and `csharp_server/perception_server.py` for the rest.
Line numbers refer to the files as they stood on 2026-10-10; they move when a file is edited.

---

## Interpretation

### "0 valid trials" is a joint result of the hardware and the label

Five of five recorded real-arm tasks ended as `infrastructure_error`, which excludes them from the
success rate. Of the three descent safety stops among them, none is clearly an infrastructure
fault: a UR protective stop during a planned descent can come from a wrong height, a collision or
a reach limit, all properties of the plan. Reading the file as "the system failed five times
because of infrastructure" over-states the label; reading it as "the system failed five times"
ignores that one task had no connected arm. Neither reading is supported.

### "0 successes in 27 attempts" is not a rate for the 0% variant

The simulation runs belong to an earlier snapshot, were stopped by hand, and include
configuration failures (UR not connected, Unity restarted). The 27 recorded attempts are all
failures, and the 12 final attempts have no outcome at all. Dividing 0 by 27 would present an
experiment that was never run to completion as a measured rate.

### The run counts are a small part of the experiments run

The 23 tasks and 89 commands are the records that survived on one computer. The team ran far more
experiments, about 500 by the owner's estimate and done together on several computers, and
those records are not reachable, so these figures are a small part of the whole and not a sample
of it. Runs that left no file and files that were overwritten are missing too. The counts also
mix planners: the 89 commands belong to the earlier planner and the 23 tasks to the experiment
pipeline, so they must not be added into one "number of experiments".

### The percentages are not rates

The 0%, 50% and 100% in the folder names order the variants by how much code fixes. They are not
measured shares, and 50% does not lie half-way between the others in any computed sense. The 90%
overlap threshold and the 80% / 20% model weighting are parameters set by the team, not outcomes.

### The offline checks pass, and that does not mean the arm succeeds

The 89 and 94 passing checks cover reset matching, source resolution, the adapter, the
validators, layer heights, figure bitmaps, the overlap arithmetic, the virtual world and the
metrics code. They run without a camera, an arm or a model, so they cannot show that a plan
reaches the right place. Reading "94 of 94" as a success rate would be a misreading.

### Latency is not quoted anywhere

No planning or execution time is given in this repository. None was analysed, so any figure would
be unsupported.
