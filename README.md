<h1 align="center">TartanIMU</h1>

<p align="center">
  <strong>A Light "Foundation Model" for Inertial Positioning in Robotics</strong>
  <br>
  CVPR 2025
</p>

<p align="center">
  <a href="https://openaccess.thecvf.com/content/CVPR2025/papers/Zhao_Tartan_IMU_A_Light_Foundation_Model_for_Inertial_Positioning_in_CVPR_2025_paper.pdf"><img alt="CVPR 2025 paper" src="https://img.shields.io/badge/Paper-CVPR_2025-B31B1B?style=flat-square&logo=adobeacrobatreader&logoColor=white"></a>
  <a href="https://superodometry.com/tartanimu"><img alt="TartanIMU project website" src="https://img.shields.io/badge/Website-TartanIMU-E76F00?style=flat-square&logo=googlechrome&logoColor=white"></a>
  <a href="https://superodometry.com/imuchallenge/data/explorer/"><img alt="Live TartanIMU inference" src="https://img.shields.io/badge/Live-Inference-0F766E?style=flat-square&logo=gradio&logoColor=white"></a>
  <a href="https://huggingface.co/Tartan-IMU/TartanIMU"><img alt="TartanIMU model weights on Hugging Face" src="https://img.shields.io/badge/Model-Hugging_Face-FFD21E?style=flat-square&logo=huggingface&logoColor=black"></a>
  <a href="https://www.kaggle.com/competitions/tartan-imu-challenge-iros2026"><img alt="TartanIMU Kaggle challenge" src="https://img.shields.io/badge/Challenge-Kaggle-20BEFF?style=flat-square&logo=kaggle&logoColor=white"></a>
  <a href="https://superodometry.com/imuchallenge/setup/"><img alt="TartanIMU challenge setup guide" src="https://img.shields.io/badge/Guide-Challenge_Setup-4B5563?style=flat-square&logo=readthedocs&logoColor=white"></a>
</p>

<p align="center">
  <a href="https://superodometry.com/tartanimu">
    <img src="doc/tartanimu_drone.gif" alt="TartanIMU drone inertial odometry demo" width="768">
  </a>
</p>

## Overview

TartanIMU learns a shared inertial representation across ground vehicles,
quadrupeds, drones, and humans. Given 6-axis accelerometer and gyroscope
measurements, it predicts 3D body-frame velocity for inertial positioning.

<table align="center">
  <tr>
    <td align="center" width="120"><img src="doc/icons/pretrain.svg" alt="Pretrain" width="32" height="32"><br><strong>Pretrain</strong></td>
    <td align="center">&rarr;</td>
    <td align="center" width="120"><img src="doc/icons/generalize.svg" alt="Generalize" width="32" height="32"><br><strong>Generalize</strong></td>
    <td align="center">&rarr;</td>
    <td align="center" width="120"><img src="doc/icons/adapt.svg" alt="Adapt" width="32" height="32"><br><strong>Adapt</strong></td>
    <td align="center">&rarr;</td>
    <td align="center" width="120"><img src="doc/icons/deploy.svg" alt="Deploy" width="32" height="32"><br><strong>Deploy</strong></td>
  </tr>
</table>

| 100+ hours | 4 platforms | 36% | 200 FPS |
| ---: | ---: | ---: | ---: |
| Multi-platform training data | Car, quadruped, drone, human | Reported ATE improvement | Reported online adaptation speed |

The released implementation provides a ResNet-LSTM multi-head foundation
model, pretrained inference, configurable training and evaluation, and an
IROS 2026 challenge starter kit.

> [!NOTE]
> This public release includes the LSTM-based `Foundation_Model`. The
> Transformer registration is retained for compatibility, but its core is not
> included. Selecting `model_name: Transformer` raises `NotImplementedError`.

## Quick Start

Requirements: Python 3.10+ and PyTorch 2.0+.

### Install

```bash
git clone https://github.com/superxslam/TartanIMU.git
cd TartanIMU

# Library and inference dependencies
pip install -e .

# Add experiment tracking for the training CLI
pip install -e ".[logging]"

# Add linting and tests for development
pip install -e ".[logging,dev]"
```

Confirm that the released model is available:

```bash
python -c "from tartan_imu.model.registry import available; print(available())"
# ['Foundation_Model', 'Transformer']
```

### Run Pretrained Inference

Download the released configuration and weights from the
[TartanIMU model repository](https://huggingface.co/Tartan-IMU/TartanIMU):

```bash
pip install huggingface_hub
huggingface-cli download Tartan-IMU/TartanIMU \
  --local-dir ./tartanimu_weights
```

Run inference on one trajectory:

```bash
python example/inference_example.py \
  --config ./tartanimu_weights/config/unified.yaml \
  --model ./tartanimu_weights/checkpoints/unified.pt \
  --npz <path/to/trajectory.npz> \
  --motion_type human
```

See [`example/minimal_example.py`](example/minimal_example.py) for a complete
fine-tuning and evaluation example.

## Train and Evaluate

`main_net.py` is the entry point for both training and evaluation. Experiment
behavior is defined by a YAML configuration.

Run the small end-to-end smoke experiment:

```bash
WANDB_MODE=disabled CUDA_VISIBLE_DEVICES=0 \
  python main_net.py \
  --config ./config/datasets/tartanimu/tartan_imu_multihead_smoke.yaml
```

Train on one or more GPUs:

```bash
# Single GPU
CUDA_VISIBLE_DEVICES=0 \
  python main_net.py --config <path/to/config.yaml>

# Multi-GPU: also set train.use_multi_gpu: True in the YAML
CUDA_VISIBLE_DEVICES=0,1,2,3 \
  python main_net.py --config <path/to/config.yaml>
```

Evaluate a checkpoint:

```bash
CUDA_VISIBLE_DEVICES=0 \
  python main_net.py \
  --config <path/to/config.yaml> \
  --checkpoint <path/to/checkpoints/best_model.pt>
```

### Command-Line Options

| Option | Purpose |
| --- | --- |
| `--config`, `--yaml` | Experiment YAML path |
| `--checkpoint` | Load weights for evaluation or warm start |
| `--resume_from` | Resume model, optimizer, scheduler, and AMP state |
| `--exp_name` | Override the experiment and output name |
| `--pdb` | Use single-process debug mode and disable W&B |

### Configuration

Dataset configurations live under `config/datasets/`. Their
`model.model_yaml` field points to a model definition such as
`config/resnet_lstm_multihead.yaml`.

The release includes two TartanIMU experiment configs:

| Config | Purpose |
| --- | --- |
| `tartan_imu_dataset.yaml` | Full car, drone, dog, and human training/evaluation |
| `tartan_imu_multihead_smoke.yaml` | Short single-GPU smoke run used by the example and tests |

| Section | Key settings |
| --- | --- |
| `data` | Dataset reader, platform paths, split names, and sample rates |
| `model` | Registered model name, model YAML, and prediction targets |
| `train` | Output directory, epochs, AMP, and multi-GPU behavior |

Training artifacts are written to `train.out_dir`. Checkpoints are stored in
`<train.out_dir>/checkpoints/`.

## Data

### Library Dataset Format

Each configured dataset root contains trajectory files grouped by split:

```text
<dataset_root>/
|-- train/
|-- val/
`-- test/
```

The included `AirLab` reader expects synchronized arrays in each `.npz` file:

| Key | Description |
| --- | --- |
| `retargetted_ts` | Timestamps |
| `retargetted_imu` | Accelerometer and gyroscope measurements |
| `retargetted_pos` | Ground-truth position |
| `retargetted_quat` | Ground-truth orientation in `xyzw` order |

Available readers:

- `AirLab`
- `Humanoid`
- `HumanoidPostProcessed`
- `HumanoidPostProcessedCached`

Select a reader with `data.dataset`. To support another format, add a module
under `tartan_imu/dataloader/` and register it in
`tartan_imu/utils/registry.py`.

### Challenge Dataset Format

The TartanIMU Challenge uses a separate window-level format with 1-second,
200-frame IMU windows. It does not use the `retargetted_*` schema above.

```text
train/<platform>/*.npz
val/<platform>/*.npz
test/test_*.npz
index/*_windows.csv
sample_submission.csv
```

The test split is anonymized and contains neither pose nor platform labels.
See the [challenge starter guide](starter/README.md) for the complete schema,
submission commands, and evaluation protocol.

## IROS 2026 TartanIMU Challenge

The challenge evaluates one shared model across car, dog/legged, drone, and
human motion. Given a 1-second IMU window, the model predicts mean body-frame
velocity `(vx, vy, vz)`.

> [!IMPORTANT]
> **Participating teams: your code, weights, and report are due at the same
> moment the competition closes — 2026-09-20 23:55 UTC.** A placement becomes
> final only after we receive those artifacts and audit them. Please send them
> as soon as your final submission is chosen rather than on the last evening.
> Use [`starter/REPORT_TEMPLATE.md`](starter/REPORT_TEMPLATE.md) for the report,
> and see [Final Submission](#final-submission-code-weights-and-report) below.

| Resource | Purpose |
| --- | --- |
| [Kaggle competition](https://www.kaggle.com/competitions/tartan-imu-challenge-iros2026) | Join the challenge and check the current schedule, rules, submissions, and leaderboard |
| [Challenge setup guide](https://superodometry.com/imuchallenge/setup/) | Follow the complete data, training, evaluation, and submission workflow |
| [Challenge dataset](https://huggingface.co/datasets/Tartan-IMU/IROS-Tartan-IMU-Challenge) | Access the multi-platform training and validation data (Hugging Face access may be required) |
| [Foundation model](https://huggingface.co/Tartan-IMU/TartanIMU) | Download the released unified configuration and model weights |
| [Live model demo](https://huggingface.co/spaces/Tartan-IMU/imu_odometry_challenge_demo) | Explore the reference models interactively |

Submissions are scored with the **TartanIMU Score**, a dimensionless combination
of 60 % per-window Absolute Velocity Error (AVE, m/s) and 40 % 20-meter segment
Absolute Trajectory Error (ATE20, m):

```text
TartanIMU Score = 0.6 * (AVE / 0.7356384388)  +  0.4 * (ATE20 / 3.1160277267)
```

Both components are macro-averaged so that all four platforms weigh equally, and
each is normalized by the value the all-zero submission reaches on the test set,
which makes the score dimensionless and pins an all-zero submission to exactly
1.000. Lower is better; the released baseline scores 0.637 on the public split.

| File | Purpose |
| --- | --- |
| `starter/starter.ipynb` | Data-to-submission walkthrough |
| `starter/baseline_submission.py` | Valid zero or constant baseline |
| `starter/tartanimu_submission.py` | Released model inference |
| `starter/kaggle_metric_tartanimu_score.py` | Leaderboard metric for validation |
| `starter/REPORT_TEMPLATE.md` | Team report template for the final submission |

Predictions must come from one model with one shared set of weights.
Platform-specific internal routing is allowed, but four separately selected
expert models are not.

The released weights and full model card are available at
[`Tartan-IMU/TartanIMU`](https://huggingface.co/Tartan-IMU/TartanIMU). Review
the model card for artifact-specific terms and known limitations.

### Final Submission: Code, Weights, and Report

Predictions alone do not settle the ranking. As stated in the rules from the
start, **a placement becomes final only after we receive a team's code,
weights, and report and audit them** — teams that do not submit these will not
appear in the final ranking. This applies equally to every team.

Please send us:

1. **Final checkpoint(s)** for the submission you want ranked, plus the
   submission ID it corresponds to and the score you expect. We re-run it and
   compare.
2. **Training code** at the exact commit that produced that checkpoint.
3. **The exact config / hyper-parameters** used — the file, not a description.
4. **The inference script** that turns the checkpoint into a submission CSV,
   including any test-time processing.
5. **Environment**: a lockfile, `requirements.txt`, or a container image.
6. **The report**, written with
   [`starter/REPORT_TEMPLATE.md`](starter/REPORT_TEMPLATE.md).

A private repository link or an archive is fine. Artifacts are used for exactly
two things — verifying the final ranking, and the challenge analysis paper — and
we do not redistribute code or weights. Reach the organizers through the
[Kaggle competition](https://www.kaggle.com/competitions/tartan-imu-challenge-iros2026)
discussion tab or the [challenge site](https://superodometry.com/imuchallenge/)
to arrange a private hand-off.

**Why we ask, beyond fairness.** We are writing an analysis paper on what this
challenge collectively discovered, and it feeds directly into the next
generation of the benchmark. We can already decompose *what* each team achieved,
per platform and per error term — but not *how*. Your report is the difference
between a leaderboard and a usable body of knowledge, and **every team that
submits is credited in that paper.**

**Deadline: 2026-09-20 23:55 UTC**, the same moment the competition closes; see
the Kaggle competition page for the authoritative schedule. Earlier is better
for us and better for you — in particular, the template's *"What did NOT work"*
section is the one teams find hardest to reconstruct a month later and the one
we value most, so start that list now rather than writing it in October.

## Repository Structure

```text
TartanIMU/
|-- tartan_imu/
|   |-- config/          # Configuration loading and object construction
|   |-- dataloader/      # Dataset readers
|   |-- evaluation/      # Metrics and trajectory analysis
|   |-- model/
|   |   |-- backbones/   # Registered model builders
|   |   |-- common/      # Shared blocks, losses, and helpers
|   |   `-- lstm/        # Released ResNet-LSTM model
|   |-- training/        # Trainer, checkpoints, and plots
|   `-- utils/           # Logging, constants, and registries
|-- config/              # Model and experiment YAML files
|-- doc/                 # README media
|-- example/
|   |-- inference_example.py
|   `-- minimal_example.py
|-- starter/             # Challenge metric, baselines, notebook, report template
|-- tests/unit/          # Unit and characterization tests
|-- tools/               # Dataset, analysis, and plotting utilities
|-- main_net.py
|-- train.py
`-- test.py
```

## Development

```bash
pip install -e ".[logging,dev]"
ruff check tartan_imu/
pytest tests/unit -q
```

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for code style, naming, testing,
checkpoint compatibility, and pull request guidelines.

Analysis and plotting helpers are available under `tools/`, including
`drift_analysis.py`, `plot_2d_traj.py`, and `gen_experiment_entry.py`.

## Citation

If TartanIMU supports your research, please cite the CVPR 2025 paper:

```bibtex
@inproceedings{zhao2025tartan,
  title={Tartan IMU: A Light Foundation Model for Inertial Positioning in Robotics},
  author={Zhao, Shibo and Zhou, Sifan and Blanchard, Raphael and Qiu, Yuheng and Wang, Wenshan and Scherer, Sebastian},
  booktitle={Proceedings of the Computer Vision and Pattern Recognition Conference},
  pages={22520--22529},
  year={2025}
}
```

## License

TartanIMU is released under the [Apache License 2.0](LICENSE).

Copyright 2026 Shibo Zhao.
