## Circular Language Experiment: Setup and Data Perturbation

This README focuses on preparing the BabyLM data, tagging it, and generating perturbed datasets using `utils.py` and `perturb.py`. Training and analysis scripts are present in the repo but are out of scope here.

This is a continuation of the work done in [Mission: Impossible Language Models](https://arxiv.org/abs/2401.06416).
This repo refers to the university project presented in [Circular Language](https://openreview.net/forum?id=PftI5myqzc)

### Requirements

- Python 3.10+ recommended
- PyTorch and Transformers (installed via `requirements.txt`)
- Stanza (for tagging; models are downloaded at first use)

Install dependencies:

```
pip install -r requirements.txt
```

### Configure paths (optional)

Open `utils.py` and review constants:

- `BABYLM_DATA_PATH` (default `./babylm_data`): base directory where raw and perturbed data live.
- `CHECKPOINT_WRITE_PATH`, `CHECKPOINT_READ_PATH`: only needed for training; can be ignored if you only generate data.

### Prepare BabyLM data

1) Download the BabyLM corpus (e.g., 100M split) from the BabyLM site and place split files under these directories (create them if needed):

- `babylm_data/train_100M/`  (e.g., `aochildes.train`, `bnc_spoken.train`, ...)
- `babylm_data/train_dev/`   (e.g., `aochildes.dev`, ...)
- `babylm_data/train_test/`  (e.g., `aochildes.test`, ...)

2) Download Stanza English models (run once):

```
python -c "import stanza; stanza.download('en')"
```

3) Tag the data with POS/lemma (and optionally constituency parses). For Hop perturbations (`hop_*`), constituency parses are required.

Examples:

```
# 100M train (with constituency parses required for hop perturbations)
python data/tag.py babylm_data/train_100M/*.train -p

# dev split
python data/tag.py babylm_data/train_dev/*.dev -p

# test split
python data/tag.py babylm_data/train_test/*.test -p
```

This creates JSON files next to the originals (e.g., `aochildes.json` or `aochildes_parsed.json`).

### Generate perturbed datasets

Perturbations are defined in `utils.py` under `PERTURBATIONS`. Available names:

- shuffle_control, shuffle_nondeterministic, shuffle_deterministic21, shuffle_deterministic57, shuffle_deterministic84
- shuffle_local3, shuffle_local5, shuffle_local10, shuffle_even_odd
- reverse_control, reverse_partial, reverse_full
- hop_control, hop_tokens4, hop_words4

Usage:

```
python perturb.py <perturbation_name> <split>
```

Where `<split>` is one of: `100M`, `10M`, `dev`, `test`, `unittest`.

Examples:

```
# Partial reversal language on 100M train
python perturb.py reverse_partial 100M

# 4-word hop language on dev (requires tagged data with -p)
python perturb.py hop_words4 dev

# Deterministic shuffle on test
python perturb.py shuffle_deterministic21 test
```

Outputs are written under `BABYLM_DATA_PATH/babylm_data_perturbed/` in subfolders:

- `babylm_<perturbation>/babylm_100M/` and `babylm_dev/` for train/dev
- `babylm_<perturbation>/babylm_test_affected/` and `babylm_test_unaffected/` for test
- `babylm_<perturbation>/babylm_test_unaffected_sents/` contains raw strings of unaffected test sentences

Notes:

- Hop perturbations use special marker tokens defined in `utils.py` (`MARKER_HOP_SING`, `MARKER_HOP_PLUR`) and require constituency parses to correctly identify affected sentences.
- Reversal perturbations insert the reverse marker (`MARKER_REV`) and rely on the GPT‑2 tokenizer augmented in `utils.py`.

### Quick verification (optional)

There are lightweight checks inside `perturb.py` (using `pytest`) comparing expected equivalences, e.g., hop tokens vs hop words, and partial reversal vs identity segments. Run selective tests after generating data:

```
pytest -k "3pres or reversal" -q
```

### Tips

- If Stanza attempts GPU use on a machine without CUDA and errors, set CPU mode by editing `data/tag.py` to use `use_gpu=False` for the pipelines.
- The shell helper `data/perturb.sh` exists but direct `python perturb.py ...` usage from the repo root is recommended.

### Citation

If you use this code or data recipes, please cite:

```
@misc{kallini2024mission,
  title={Mission: Impossible Language Models},
  author={Julie Kallini and Isabel Papadimitriou and Richard Futrell and Kyle Mahowald and Christopher Potts},
  year={2024},
  eprint={2401.06416},
  archivePrefix={arXiv},
  primaryClass={cs.CL}
}
```
