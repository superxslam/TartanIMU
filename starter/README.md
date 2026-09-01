# TartanIMU Challenge — Starter Kit

Everything you need to make a valid submission to the **TartanIMU Challenge:
Multi-Platform Inertial Odometry** on Kaggle, plus the released pretrained
baseline.

The task: from a **1.0 s window of 6-axis IMU** (accelerometer + gyroscope,
200 Hz, body frame, gravity retained), predict the sensor's **3-D body-frame
velocity** `(vx, vy, vz)` — the mean velocity over the window. A single model
must handle four embodiments: **car / dog (legged) / drone / human**.

## Data you download from Kaggle

| File | Columns / keys | Notes |
| --- | --- | --- |
| `train/<platform>/*.npz` | `imu (N,6)`, `ts`, `pos`, `quat`, `vel_body (N,3)`, `platform_id`, `fs` | labelled training trajectories |
| `val/<platform>/*.npz` | same as train | validation trajectories |
| `test/test_0000.npz` … `test_0088.npz` | `imu (N,6)`, `ts`, `fs` only | **anonymized** — no pose, no platform anywhere |
| `index/{train,val}_windows.csv` | `window_id, platform, traj_id, win_idx` | window → trajectory map |
| `index/{train,val}_targets.csv` | `window_id, vx, vy, vz` | supervision (body-frame velocity) |
| `index/test_windows.csv` | `window_id, traj_id, win_idx` | **no platform column** (anonymized) |
| `sample_submission.csv` | `window_id, vx, vy, vz` | all-zero template, the exact 30,644 rows to fill |

IMU channel order in every `.npz` is `[ax, ay, az, gx, gy, gz]` (accel m/s²,
gyro rad/s). A window is `1.0 s = 200 frames`, non-overlapping: window `k` of a
trajectory is `imu[k*200:(k+1)*200]`.

> ⚠️ `platform_id` in train/val is **0-based** (`car=0, dog=1, drone=2, human=3`).
> The released TartanIMU model's `motion_type` API is **1-based** — add 1 when
> passing a dataset `platform_id` into that model.

## Submission format

A CSV with exactly these columns, one row per test `window_id` (30,644 rows):

```
window_id,vx,vy,vz
1,0.0,0.0,0.0
2,0.0,0.0,0.0
...
```

`vx, vy, vz` is the predicted mean **body-frame** velocity (m/s) of that window.

## Scoring — the TartanIMU Score (lower is better)

The leaderboard metric is the **TartanIMU Score**, a dimensionless combination of
two components:

```
TartanIMU Score = 0.6 × (AVE / 0.7356384388)  +  0.4 × (ATE20 / 3.1160277267)
```

| | Component | Unit | What it measures |
| --- | --- | --- | --- |
| **60 %** | **AVE** — Absolute Velocity Error | m/s | **Instantaneous accuracy.** The mean over a trajectory's windows of the Euclidean error `‖v_pred − v_gt‖` — the quantity a downstream state estimator actually consumes. |
| **40 %** | **ATE20** — 20 m-segment Absolute Trajectory Error | m | **Temporal consistency.** Integrate the predictions into a path and compare it with the true path over fixed 20 m pieces. This punishes correlated bias and drift that a per-window average hides. |

ATE20 in detail:

1. Each test trajectory's ground-truth path is cut into consecutive segments of
   about **20 m of travelled distance**.
2. Within a segment, your per-window velocities are rotated to the world frame
   with ground-truth orientation (used *only* in scoring, never available as a
   model input), multiplied by the window duration, and accumulated into an
   estimated path.
3. The estimated segment is **SE(3)-aligned** to the ground-truth segment
   (Umeyama, rotation + translation, no scale); its error is the RMS position
   error after alignment.
4. Trajectory error = mean over its segments.

Both components use the same hierarchical averaging: per window → mean over a
trajectory → mean over a platform's trajectories → **equal-weight mean over the
four platforms**, so each platform carries exactly 25 % of each term and no
platform (or window count) dominates.

Two properties worth knowing before you tune:

- **The all-zero submission scores exactly 1.000.** Any score above 1.0 is worse
  than submitting nothing at all. Scoring ATE20 over *segments* rather than whole
  trajectories is what makes that true: a whole-trajectory ATE rewards predicting
  nothing, because an all-zero submission integrates to a single point whose
  aligned error is merely the path's radius of gyration. With fixed-length
  segments, standing still loses on **every** platform.
- **The stated 60 / 40 weights are the weights that actually act.** Each
  component is divided by the value the all-zero submission reaches on the full
  test set, which puts m/s and metres on a common scale. Un-normalized, ATE20
  spans roughly 0.15 – 3.5 m while AVE spans only 0.0 – 0.75 m/s, so a raw
  `0.6·AVE + 0.4·ATE20` would let ATE20 drive about 75 % of the ranking. The two
  constants are fixed properties of the test set, published on the **Evaluation**
  tab, and are not adjusted during the competition.

The exact scoring code is `starter/kaggle_metric_tartanimu_score.py` — the same
scorer that runs on the leaderboard. You cannot score the test set locally (the
pose is withheld), but you can **self-score on the labelled `val` split** while
iterating.

Reference points, matching the Evaluation tab:

| Submission | AVE (m/s) | ATE20 (m) | Score Public | Score Private |
| --- | --- | --- | --- | --- |
| ground-truth velocities (the floor) | 0.000 | 0.151 | 0.021 | 0.019 |
| **released unified baseline** | 0.461 | 1.261 | **0.637** | **0.456** |
| all-zeros (`sample_submission.csv`) | 0.736 | 3.116 | 1.054 | 0.922 |
| per-platform mean velocity | 0.749 | 3.496 | 1.080 | 1.036 |

The AVE and ATE20 columns are the macro-averaged components over the full test
set; the two score columns are the combined metric on each split. The floor is
0.019 rather than 0.000 because even exact per-window *average* velocities leave
a small ATE20 residual when integrated — a one-second window cannot represent
sub-second path curvature.

## Quick start

```bash
# 0. install the library
git clone https://github.com/superxslam/TartanIMU && cd TartanIMU
pip install -e .
pip install huggingface_hub          # for the pretrained baseline

# 1. a guaranteed-valid all-zero submission (verifies your pipeline)
python starter/baseline_submission.py \
    --sample_submission /path/to/sample_submission.csv \
    --out submission_zero.csv

# 2. the released pretrained baseline on the labelled val split
#    (heads route automatically from the CSV's platform column)
python starter/tartanimu_submission.py \
    --test_root  /path/to/val \
    --windows    /path/to/index/val_windows.csv \
    --out        submission_val.csv

# 3. the same model on the anonymized test split — no platform column exists,
#    so a head must be named (or supply your own routing via --routing)
python starter/tartanimu_submission.py \
    --test_root  /path/to/test \
    --windows    /path/to/index/test_windows.csv \
    --head       human \
    --out        submission_tartanimu.csv
    # optional: --checkpoint <local .pt>  --config <local yaml>  --device cpu
```

Upload the resulting CSV on the competition's **Submit Predictions** page.

> **The rules in one line:** predictions must come from a **single model with
> one shared set of weights**. The test set is anonymized precisely so windows
> cannot be routed to four per-platform experts; a single model that adapts
> internally (learned conditioning, mixture-of-experts inside one network) is
> allowed and encouraged.

## Files in this kit

| File | Purpose |
| --- | --- |
| `README.md` | this guide |
| `baseline_submission.py` | writes an all-zero (or constant) valid submission |
| `tartanimu_submission.py` | released pretrained baseline → submission |
| `kaggle_metric_tartanimu_score.py` | the exact leaderboard metric (for reference / val self-scoring) |
| `starter.ipynb` | notebook walking through data → prediction → submission |
| `REPORT_TEMPLATE.md` | **the report every team submits with its code and weights** |

## After your predictions: code, weights, and report

Uploading a CSV puts you on the leaderboard; it does not finalize your
placement. **A placement becomes final only after we receive your code,
weights, and report and audit them** — so plan for that package the same way
you plan for the submission itself.

Fill in **[`REPORT_TEMPLATE.md`](REPORT_TEMPLATE.md)** and send it with the
checkpoint, the training code at the exact commit, the config file, the
inference script, and your environment. A private link or an archive is fine;
we do not redistribute code or weights, and every submitting team is credited
in the challenge analysis paper.

**Due 2026-09-20 23:55 UTC**, the same moment the competition closes — earlier
is much better. One concrete tip: the template's *"What did NOT work"* section
is the one we value most and the one that is nearly impossible to reconstruct a
month after you stop experimenting. Keep that list as you go and it costs you
almost nothing.

## Pretrained model

The released baseline weights live on the Hugging Face Hub:
**[`Tartan-IMU/TartanIMU`](https://huggingface.co/Tartan-IMU/TartanIMU)**
(unified 4-head model + per-platform experts, Apache License 2.0; drone-derived
weights are research / non-commercial use). See the model card for
per-platform accuracy and known limitations.
