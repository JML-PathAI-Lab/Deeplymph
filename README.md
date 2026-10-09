# DeepLymph

DeepLymph is a computational-pathology framework for H&E-based regional
lymph-node phenotyping and prognostic stratification in colorectal cancer. It
combines multi-task lymph-node prediction, prototype-based morphological
representation learning, and patient-level nodal profiling.

## Project status

This GitHub repository currently provides the public project documentation.
The source package is being prepared for release. No whole-slide images,
patient-level clinical data, trained checkpoints, institution-specific
configuration files, evaluation outputs, or visualisation results are
included in this repository.

## Method overview

The main model, **SoftMT-MoE**, receives pre-extracted patch features for a
lymph node and routes them through shared, metastatic, and negative expert
branches. Its training objective supports lymph-node phenotype prediction and
separate prototype dictionaries for exploratory morphological subtype
analysis. Patient-level summaries are subsequently derived from the predicted
nodal phenotypes.

The released implementation is designed to keep the reusable training
components separate from data-governed research utilities. This allows the
methodology to be inspected and reused without publishing protected clinical
information or internal comparison workflows.

## Local development layout

The maintainers work with the following two sibling directories locally. The
names below describe the local layout; `DeepLymph-private/` is deliberately
not part of this GitHub repository.

```text
DeepLymph/
├── DeepLymph-main/       # Public-release candidate: model and training code
└── DeepLymph-private/    # Local-only research, evaluation, and visualisation code
```

### Public main program

`DeepLymph-main/` is the part intended for public release. Its modules are
organised as follows:

```text
DeepLymph-main/
├── configs/              # Portable example configuration, without data paths
├── core/                 # Configuration loading, datasets, losses, metrics, utilities
├── models/               # SoftMT-MoE and prototype-mixture model components
├── clustering/           # Training-split prototype dictionary pretraining
├── training/             # Training entry point and training loop
├── requirements.txt      # Direct Python dependencies
└── README.md             # Developer-oriented setup and module guide
```

The public code reads dataset locations and other machine-specific settings
only from a JSON file selected through `DEEPLYMPH_DATA_CONFIG`. The repository
contains `configs/dataset.example.json` as a schema example; it contains no
real paths or protected identifiers.

After the source package is released, the expected workflow is:

```bash
cd DeepLymph-main
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Create and edit a local configuration outside the repository.
cp configs/dataset.example.json /secure/location/dataset.local.json
export DEEPLYMPH_DATA_CONFIG=/secure/location/dataset.local.json

# Fit independent prototype dictionaries from training-split patches only.
python -m clustering.pretrain_kmeans --sample_type pos --encoder uni_v2
python -m clustering.pretrain_kmeans --sample_type neg --encoder uni_v2

# Train SoftMT-MoE.
python -m training.main --encoder uni_v2 --gpu 0
```

The example commands require appropriately prepared feature files and a local
configuration. They do not download data and do not make any protected data
available.

### Local private companion code

`DeepLymph-private/` is retained only on authorised local storage. It is not a
second public package and must never be copied, linked, committed, or pushed
to this repository. Its role is to support internal research activities that
are outside the public release scope:

```text
DeepLymph-private/
├── config/               # Institution-specific dataset configuration
├── data_tools/           # Data-quality checks and local table utilities
├── baselines/            # Comparison-only baselines
├── evaluation/           # Internal performance and confidence-interval analyses
├── experiments/          # Reviewer and ablation experiment orchestration
├── cluster_analysis/     # Cluster selection, reproducibility, and patch review
├── visualization/        # Heatmaps and whole-slide visualisations
├── reporting/            # Internal report helpers
└── private_models/       # Comparison-only compatibility implementations
```

The private directory may refer to local metadata, protected data locations,
internal checkpoints, or unpublished analyses. These materials must remain
outside the public repository even when they are used to run the main program
locally.

## Publication boundary

Before any public update, maintainers should review the staged file list with
`git status --short` and add only selected public files. Do not use a bulk add
command from the local parent directory. In particular, do not publish:

- data configuration files containing real paths or identifiers;
- whole-slide images, feature files, clinical tables, and patient-level data;
- model checkpoints, derived predictions, or experiment outputs;
- internal evaluations, baseline comparisons, reviewer analyses, or figures;
- manuscripts and other unpublished research materials.

## Data availability

Whole-slide images and patient-level clinical data are not publicly available
because of privacy and institutional data-governance requirements. Public code
will therefore require users to supply data for which they have the necessary
permissions.
