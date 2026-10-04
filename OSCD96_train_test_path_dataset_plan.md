---
name: OSCD96 test path
overview: The local OSCD96 test folder is already used. There is no separate test-path setting; it is built as `data_path / dataset / test` in the datamodule.
todos:
  - id: confirm-test-path
    content: "No code change: test already resolves to Desktop/Dataset/oscd96/test via YAML data_path + datamodule.py"
    status: completed
isProject: false
---

# Where the OSCD96 test set is set

Your test folder is already wired. You do **not** add `C:\Users\MataRajDulari\Desktop\Dataset\oscd96\test` as its own string anywhere unless you want a new config field.

## How test is resolved today

[`configs/exp/BTC-B.yaml`](configs/exp/BTC-B.yaml) and [`configs/exp/BTC-T.yaml`](configs/exp/BTC-T.yaml) already have:

```yaml
data:
  dataset: "oscd96"
  data_path: "C:/Users/MataRajDulari/Desktop/Dataset"
  use_hf: False
  img_size: 96
```

[`data/datamodule.py`](data/datamodule.py) builds the test split here:

```52:56:data/datamodule.py
            data = {
                "train": self.data_path / self.dataset_name / "train",
                "test": self.data_path / self.dataset_name / "test",
                "val": self.data_path / self.dataset_name / "val",
            }
```

That `test` path is:

`C:\Users\MataRajDulari\Desktop\Dataset` + `oscd96` + `test`

which is exactly `C:\Users\MataRajDulari\Desktop\Dataset\oscd96\test`.

It is loaded as `self.test_data` and used by `test_dataloader()`. [`train.py`](train.py) always runs `evaluate(...)` after training, which calls Lightning `trainer.test()` on that dataloader. `--eval_only` skips training and still uses the same test folder.

If `val` is missing, validation also uses this same test set.

## What you would change (only if the folder were different)

- Change **`data.data_path`** and **`data.dataset`** in the YAML (or CLI). Do not put `\test` inside `data_path`.
- The Python line to edit for a hardcoded test dir is [`data/datamodule.py`](data/datamodule.py) `"test": ...` (line 54). That is **not** needed for your current layout.

## Test folder layout required

```
C:\Users\MataRajDulari\Desktop\Dataset\oscd96\test\A\*.png
C:\Users\MataRajDulari\Desktop\Dataset\oscd96\test\B\<same names>
C:\Users\MataRajDulari\Desktop\Dataset\oscd96\test\label\<same names>
```

## Implementation

No further file edits. YAML and datamodule already point at your local test set. Run:

```bash
python train.py --config configs/exp/BTC-B.yaml
```

Eval-only (needs `--ckpt_path`):

```bash
python train.py --config configs/exp/BTC-B.yaml --eval_only --ckpt_path <your_checkpoint>
```
