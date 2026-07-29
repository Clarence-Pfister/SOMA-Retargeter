# SOMA-Retargeter — G1 23-DoF fork

Fork of [NVIDIA/soma-retargeter](https://github.com/NVIDIA/soma-retargeter).
This branch tracks upstream; the work lives on the branch below.

## Branches

| Branch | Contents |
|---|---|
| `main` | Upstream NVIDIA/soma-retargeter, plus this README. No functional changes. |
| `feature/g1-23dof-retargeting` | Adds a Unitree G1 **23-DoF** retargeting target alongside the upstream 29-DoF one. |

For installation, the retargeting pipeline, the interactive viewer, batch
conversion and everything else, see the official repository:
**https://github.com/NVIDIA/soma-retargeter**

## What the 23-DoF branch adds

Upstream targets the 29-DoF Unitree G1 and hardcodes its MJCF. The branch makes
the robot model configurable and adds the 23-DoF joint set:

| File | Change |
|---|---|
| `soma_retargeter/assets/csv.py` | `UnitreeG123DOF_CSVConfig` — 23-joint CSV header (drops `waist_roll`/`waist_pitch` and both wrist `pitch`/`yaw` pairs). |
| `soma_retargeter/configs/unitree_g1/soma_to_g1_23dof_retargeter_config.json` | SOMA → G1 23-DoF retargeter config. |
| `soma_retargeter/configs/unitree_g1/g1_23dof_feet_stabilizer_config.json` | Matching feet-stabilizer config. |
| `soma_retargeter/pipelines/newton_pipeline.py` | Honours an `mjcf_path` key in the retargeter config; falls back to the downloaded 29-DoF asset. |
| `soma_retargeter/pipelines/feet_stabilizer.py` | Loads its config via a `feet_stabilizer_config` key. |
| `soma_retargeter/utils/newton_utils.py` | `resolve_file_path` for relative `mjcf_path` values. |

The 29-DoF path is unchanged: with no `mjcf_path` set, the pipeline still
downloads and uses `unitree_g1/mjcf/g1_29dof_rev_1_0.xml`.

## Usage

```python
from soma_retargeter.assets.csv import UnitreeG123DOF_CSVConfig, save_csv
from soma_retargeter.pipelines.newton_pipeline import NewtonPipeline
from soma_retargeter.utils import io_utils

config = io_utils.load_json(
    io_utils.get_config_file("unitree_g1", "soma_to_g1_23dof_retargeter_config.json")
)
config["mjcf_path"] = "/path/to/g1_23dof.xml"   # your 23-DoF MJCF

pipeline = NewtonPipeline(skeleton, "soma", "unitree_g1", retarget_config=config)
pipeline.add_input_motions([anim_buffer], [offset], scale_animation=True)
csv_buffers = pipeline.execute()

save_csv("motion_g1_23dof.csv", csv_buffers[0], UnitreeG123DOF_CSVConfig())
```

## Used by

[Clarence-Pfister/GEM-X](https://github.com/Clarence-Pfister/GEM-X) consumes this
as its `third_party/soma-retargeter` submodule, pinned to
`feature/g1-23dof-retargeting`, to provide `--retarget g1_23dof`.

## License

Apache 2.0, same as upstream — see [LICENSE](LICENSE).
