---
name: Local OSCD96 paths
overview: Point training at your local OSCD96 folders by switching off Hugging Face download and setting `data_path` so the existing loader resolves `...\oscd96\train` and `...\oscd96\test`.
todos:
  - id: update-yaml-data
    content: Set dataset oscd96, img_size 96, data_path to Desktop/Dataset, use_hf False in BTC-B.yaml and/or BTC-T.yaml
    status: completed
isProject: false
---

# Use local OSCD96 train/test folders

You do **not** need to hardcode the two folder strings in Python. [data/datamodule.py](data/datamodule.py) already builds them as:

```52:56:data/datamodule.py
            data = {
                "train": self.data_path / self.dataset_name / "train",
                "test": self.data_path / self.dataset_name / "test",
                "val": self.data_path / self.dataset_name / "val",
            }
```

With:

- `data_path` = `C:\Users\MataRajDulari\Desktop\Dataset`
- `dataset` = `oscd96`

that becomes exactly:

- `C:\Users\MataRajDulari\Desktop\Dataset\oscd96\train`
- `C:\Users\MataRajDulari\Desktop\Dataset\oscd96\test`

There is no `val` folder in your paths. The same file already falls back to the test set for validation (`"Validation data not present, using test set."`).

```mermaid
flowchart LR
  yaml[BTC yaml data block]
  dm[CDDataModule.prepare_dataset]
  ds[CDDataset A B label pngs]
  yaml --> dm
  dm --> ds
```

## Where to change

**1. Experiment YAML (main place)** — [configs/exp/BTC-B.yaml](configs/exp/BTC-B.yaml) and/or [configs/exp/BTC-T.yaml](configs/exp/BTC-T.yaml) (whichever you actually run).

Update the `data:` block:

```yaml
data:
  dataset: "oscd96"
  img_size: 96
  num_workers: 8
  batch_size: 32
  pin_memory: True
  data_path: "C:/Users/MataRajDulari/Desktop/Dataset"
  use_hf: False
```

Why each flag:

- `use_hf: False` — skip Hugging Face (`blaz-r/OSCD_RGB_Cropped_96` in [data/hf_datasets.py](data/hf_datasets.py)) and read disks instead.
- `data_path` — parent of `oscd96`, **not** the `train` folder itself.
- `img_size: 96` — OSCD96 tiles are 96×96; current configs use `256` for CLCD.

Do **not** uncomment `load_in_mem: "hdf5"` unless your files are `train.h5` / `test.h5`. For PNG folders, leave `load_in_mem` unset so [data/dataset.py](data/dataset.py) uses `load_paths_from_dir` (`A/*.png`, matching `B/` and `label/`).

**2. Optional CLI instead of editing YAML**

```bash
python train.py --config configs/exp/BTC-B.yaml --data.dataset oscd96 --data.img_size 96 --data.data_path "C:/Users/MataRajDulari/Desktop/Dataset" --data.use_hf False
```

`DataArgs` already includes `data_path` and `use_hf` in [configs/config_parser.py](configs/config_parser.py); [train.py](train.py) passes them into `CDDataModule`.

**3. No change required** in [data/datamodule.py](data/datamodule.py) or [data/hf_datasets.py](data/hf_datasets.py) if you use the layout above.

## Folder layout the loader expects

Each of `train/` and `test/` must look like:

```
oscd96/train/A/*.png
oscd96/train/B/<same filenames>
oscd96/train/label/<same filenames>
```

Same for `test/`. If images are not PNG, or folders are named differently, `CDDataset.load_paths_from_dir` will raise `No images found`.

## After you confirm this plan

The implementation step is only YAML (and optionally a one-line README note). Python stays as-is unless your OSCD96 tree is not `A/B/label` PNGs.
