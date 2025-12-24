# Repository Guidelines

## Project Structure & Module Organization
- Core steps live under `STEP1_collect_data/` (sensing + visualization), `STEP2_build_dataset/` (IK + HDF5 export), and `STEP3_train_policy/robomimic/` (policy training). Assets for docs and demos are in `assets/`. Environment specs are in `install/`.
- Scripts are written as standalone entry points; keep new utilities in the matching step folder and import shared helpers (e.g., `hyperparameters.py`, `utils.py`) instead of duplicating logic.
- Large meshes and URDFs sit in `STEP2_build_dataset/leap_hand_mesh/`; avoid modifying them unless you regenerate assets.

## Build, Test, and Development Commands
- Create Windows data-collection env:  
  ```bash
  cd install
  conda env create -n mocap -f env_nuc_windows.yml
  ```
- Create Ubuntu/workstation env for processing/training:  
  ```bash
  conda create -n dexcap python=3.8
  conda activate dexcap
  cd install && pip install -r env_ws_requirements.txt
  cd ../STEP3_train_policy && pip install -e .
  ```
- Run data recording (NUC): `cd STEP1_collect_data && python redis_glove_server.py` in one shell, then `python data_recording.py -s --store_hand -o ./save_data_scenario_1` in another.
- Visualize/clean data: `python replay_human_traj_vis.py --directory save_data_scenario_1` (add `--calib` to adjust drift), then `python transform_to_robot_table.py --directory save_data_scenario_1`.
- Build dataset: `cd STEP2_build_dataset && python demo_create_hdf5.py`.
- Train policy: `cd STEP3_train_policy/robomimic && python scripts/train.py --config training_config/[NAME].json`.

## Coding Style & Naming Conventions
- Python 3.8; prefer PEP 8: 4-space indents, snake_case for functions/variables, CamelCase for classes, UPPER_SNAKE_CASE for constants. Keep modules self-contained and import from siblings rather than relative file copies.
- Scripts should expose argparse CLIs; default outputs go to a user-supplied directory instead of hard-coded paths. Keep GPU/robot host addresses configurable flags.
- Do not commit generated artifacts (`__pycache__`, `trained_models`, `.egg-info`); honor `.gitignore`.

## Testing & Validation
- No formal test suite; validate changes by running the relevant pipeline stage: recording + replay for collection changes, `demo_create_hdf5.py` for dataset changes, and a short `scripts/train.py` run with a small config for training edits.
- When touching transforms/IK, sanity-check with `replay_human_traj_vis.py` and verify point cloud alignment after `transform_to_robot_table.py`.
- Log outputs to the run directory; avoid writing outside the project tree.

## Data Handling & Security
- Keep raw recordings and generated HDF5 files out of the repo; store under a user-created folder (`./save_data_*`) and sync externally if needed.
- Network endpoints (e.g., `Forward IP`, redis hosts) should remain configurable; avoid hard-coding lab-specific IPs in committed code.

## Commit & Pull Request Guidelines
- Follow the existing log style: short, present-tense summaries (e.g., `clean up ik`, `fix typo & add valid script`). Group related changes per commit.
- PRs should include: summary of intent, affected step(s), key commands used for validation, and any config files or sample outputs touched. Add screenshots or brief notes when UI/visualizations change. Link related issues or datasets when applicable.
