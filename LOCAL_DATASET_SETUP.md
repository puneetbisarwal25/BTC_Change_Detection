# Local OSCD96 Dataset Setup

This project can load the OSCD96 dataset from a local directory instead of downloading it from Hugging Face. The BTC-B and BTC-T experiment configs are already set up to use the local dataset.

## Dataset Location

The dataset root is:

```text
C:\Users\MataRajDulari\Desktop\Dataset\oscd96
```

The expected split locations are:

```text
C:\Users\MataRajDulari\Desktop\Dataset\oscd96\train
C:\Users\MataRajDulari\Desktop\Dataset\oscd96\test
```

Each split must contain `A`, `B`, and `label` directories. The files are expected to have matching PNG filenames across the three directories. For example:

```text
oscd96/
  train/
    A/scene_001.png
    B/scene_001.png
    label/scene_001.png
  test/
    A/scene_101.png
    B/scene_101.png
    label/scene_101.png
```

## Configuration

In the config you intend to run, set the `data` section like this:

```yaml
data:
  dataset: "oscd96"
  data_path: "C:/Users/MataRajDulari/Desktop/Dataset"
  use_hf: False
```

`data_path` must be the directory **above** `oscd96`, and `dataset` must match the `oscd96` directory name. The data module combines them with the split name, producing:

- `C:/Users/MataRajDulari/Desktop/Dataset/oscd96/train`
- `C:/Users/MataRajDulari/Desktop/Dataset/oscd96/test`

Do not set `data_path` to the `train` or `test` directory itself. There is no separate test-path setting in the config.

The relevant experiment files are:

- `configs/exp/BTC-B.yaml`
- `configs/exp/BTC-T.yaml`

Both currently use the local path above. If the dataset is moved, update `data_path` in the config you run to the new parent directory. Keep `dataset` set to the name of the dataset folder under that parent.

## Run Training

From the repository root, run one of the following:

```powershell
python train.py --config configs/exp/BTC-B.yaml
python train.py --config configs/exp/BTC-T.yaml
```

## Validation Behavior

With local on-demand loading (`use_hf: False` and `load_in_mem` omitted), the current data module falls back to the test split for validation. As a result, validation metrics during training are calculated on the test data, not on a separate validation split. The code only checks for a validation split when using Hugging Face, `load_in_mem: "direct"`, or `load_in_mem: "hdf5"` mode.
