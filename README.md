# Kaggle: ByteCNN sequential training on Hugging Face APK bucket

Notebook: [`ByteCNN_Sequential_Year_Training.ipynb`](ByteCNN_Sequential_Year_Training.ipynb)

Data source: [sakhawat2088/android-apks](https://huggingface.co/buckets/sakhawat2088/android-apks) — a **Hugging Face Bucket** (not a Model/Dataset repo).

Uses `list_bucket_tree` + `download_bucket_files` (`huggingface_hub>=1.10`). See [HF Buckets docs](https://huggingface.co/docs/huggingface_hub/en/guides/buckets).

## What it does

1. Lists APK paths on Hugging Face (`2020/benign`, `2020/malware`, …)
2. Trains **year by year** in order: **2020 → 2021 → 2022 → 2023**
3. **Loads the checkpoint** from the previous year before each new year (continual learning)
4. **Streams downloads** in chunks (default 400 APKs) so 108 GB never has to fit on disk at once
5. Writes **year-wise CSV** stats + final `bytecnn_continual.pth` + `bytecnn_basemodel_2020.onnx`

## Kaggle setup

1. [Create a new Kaggle notebook](https://www.kaggle.com/code)
2. **File → Upload notebook** → upload `ByteCNN_Sequential_Year_Training.ipynb`
3. **Settings**
   - Accelerator: **GPU** (T4 or better)
   - Internet: **On**
   - Persistence (optional): **On** if training large slices
4. **Add-ons → Secrets** → `HF_TOKEN` only if the bucket becomes private ( **not needed** for [sakhawat2088/android-apks](https://huggingface.co/buckets/sakhawat2088/android-apks) )
5. Run all cells

## Config knobs (cell 3)

| Variable | Default | Purpose |
|----------|---------|---------|
| `MAX_PER_CLASS` | `500` | Limit APKs per class per year (`None` = all) |
| `EPOCHS_PER_YEAR` | `10` | Epochs per year on streaming data |
| `DOWNLOAD_CHUNK` | `400` | APKs downloaded per train chunk |
| `YEAR_ORDER` | 2020…2023 | Training order |

**First run:** keep `MAX_PER_CLASS=500` to verify the pipeline (~1–2 hours).

**Full dataset:** set `MAX_PER_CLASS=None`. You will likely need extra disk (Kaggle Persistence or run on a cloud VM with 200+ GB).

If `list_bucket_tree` fails with 401 / Repository Not Found, you were using the old model-repo API. Re-upload the latest notebook (uses Buckets API).

If listing still fails, try adding `HF_TOKEN` secret (some bucket endpoints require login even for public data).

## Outputs (Kaggle Output tab)

```
/kaggle/working/bytecnn/
  models/bytecnn_continual.pth
  models/bytecnn_basemodel_2020.onnx
  stats/year_2020.csv … year_2023.csv
  stats/yearwise_summary.csv
```

## Deploy to VigiDroid

Download `bytecnn_basemodel_2020.onnx` and copy:

```bash
cp bytecnn_basemodel_2020.onnx /path/to/Faiyaz_work/vigidroid/app/src/main/assets/
```

Rebuild and run the Android app.

## Local alternative

The same continual logic can be adapted for a machine with the full dataset mounted at e.g. `/data/android-apks/2020/benign` by replacing HF download calls with local `Path` glob — use `1dcnn/src/main.py` as reference for fixed local paths.
