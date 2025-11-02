# HNC Radiomics: Quantitative Imaging Analysis Pipeline

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PyRadiomics](https://img.shields.io/badge/PyRadiomics-IBSI%20Compliant-green.svg)](https://pyradiomics.readthedocs.io/)

## Overview

This repository implements a comprehensive radiomics feature extraction pipeline for **Head and Neck Cancer (HNC)** CT imaging analysis. The pipeline extracts quantitative imaging biomarkers following Image Biomarker Standardization Initiative (IBSI) guidelines, enabling objective tumor characterization and outcome prediction from medical imaging data.

### Key Features

- 🔍 **Automated RT Structure Detection**: Intelligent search and validation of tumor contours across DICOM RT structure sets
- 📊 **IBSI-Compliant Feature Extraction**: 100+ standardized radiomics features (shape, first-order, texture)
- 🎯 **Batch Processing**: Efficient pipeline for large-scale cohort analysis
- 📈 **Quality Control**: Built-in data validation and error handling
- 🔬 **Research-Ready**: Output format compatible with machine learning frameworks

### Clinical Applications

- Tumor characterization and staging
- Treatment response prediction
- Survival analysis and prognostic modeling
- Radiogenomics studies
- Clinical decision support systems

---

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Pipeline Components](#pipeline-components)
- [Dataset Requirements](#dataset-requirements)
- [Usage Guide](#usage-guide)
- [Output Description](#output-description)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [Citation](#citation)
- [Contributing](#contributing)
- [License](#license)

---

## Installation

### Prerequisites

- Python 3.8 or higher
- Conda or pip package manager
- Minimum 8GB RAM
- DICOM dataset with CT scans and RT structure sets

### Environment Setup

#### Option 1: Conda Environment (Recommended)

```bash
# Create dedicated environment
conda create -n pyradiomics python=3.8
conda activate pyradiomics

# Install dependencies
pip install pyradiomics
pip install rt-utils
pip install SimpleITK
pip install pydicom
pip install pandas
pip install numpy
pip install jupyter
```

#### Option 2: pip Installation

```bash
# Create virtual environment
python -m venv radiomics_env
source radiomics_env/bin/activate  # On Windows: radiomics_env\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Verify Installation

```python
import radiomics
import SimpleITK as sitk
import rt_utils

print(f"PyRadiomics version: {radiomics.__version__}")
print(f"SimpleITK version: {sitk.Version.VersionString()}")
print("Installation successful!")
```

---

## Quick Start

### 1. Prepare Your Dataset

Organize your DICOM data following this structure:

```
maastro dataset/
├── Images/
│   ├── PATIENT-001/
│   │   ├── CT/              # CT DICOM series
│   │   └── RTSTRUCT/        # RT structure set
│   ├── PATIENT-002/
│   └── ...
```

### 2. Check Structure Names

Before feature extraction, identify the correct structure nomenclature in your dataset:

```bash
jupyter notebook check_RTstructure_name.ipynb
```

This notebook will:
- Scan all patient RT structure files
- List available structure names (GTV, CTV, PTV, etc.)
- Generate frequency statistics
- Provide code recommendations for your specific dataset

**Example Output:**
```
Structure Names Found:
  'GTV-1': 180 patients (90.0%)
  'CTV': 175 patients (87.5%)
  'PTV': 200 patients (100.0%)
```

### 3. Extract Radiomics Features

Run the main extraction pipeline:

```bash
jupyter notebook extract_radiomics_features.ipynb
```

Configure the target structure and processing mode:

```python
TARGET_ROI = "GTV-1"           # Update based on check_RTstructure output
FEATURE_MODE = "original"      # Options: "original", "wavelet", "log", "all"
OUTPUT_CSV = "radiomics_features.csv"
```

### 4. Analyze Results

The pipeline generates a CSV file with extracted features:

```python
import pandas as pd

# Load results
df = pd.read_csv("radiomics_features.csv")

print(f"Patients analyzed: {len(df)}")
print(f"Features extracted: {len(df.columns) - 2}")
print(f"\nFeature categories:")
print("  - Shape features: ~14")
print("  - First-order features: ~18")
print("  - Texture features: ~75")
```

---

## Pipeline Components

### 1. Structure Name Checker (`check_RTstructure_name.ipynb`)

**Purpose**: Dataset reconnaissance and structure nomenclature discovery

**Functionality:**
- Scans all patient RT structure files
- Extracts structure name lists
- Generates frequency distributions
- Identifies naming inconsistencies
- Provides structure selection code recommendations

**Output:**
- Console report with structure statistics
- Code snippets optimized for your dataset

**When to Use:**
- First time analyzing a new dataset
- Dataset contains multiple structure variants
- Uncertain about structure naming conventions

### 2. Feature Extraction Pipeline (`extract_radiomics_features.ipynb`)

**Purpose**: High-throughput radiomics feature extraction

**Functionality:**

#### Stage 1: CT Loading
- Reads DICOM CT series using SimpleITK
- Preserves spatial metadata (spacing, origin, direction)
- Handles multi-slice volumes

#### Stage 2: Structure Processing
- Locates target ROI in RT structure sets
- Converts vector contours to binary masks
- Performs spatial registration with CT

#### Stage 3: Feature Extraction
Extracts IBSI-compliant features:

| Category | Features | Description |
|----------|----------|-------------|
| **Shape** | 14 | Geometric properties (volume, surface area, sphericity) |
| **First-Order** | 18 | Intensity statistics (mean, median, skewness, kurtosis) |
| **GLCM** | 24 | Gray Level Co-occurrence Matrix (texture) |
| **GLRLM** | 16 | Gray Level Run Length Matrix (texture) |
| **GLSZM** | 16 | Gray Level Size Zone Matrix (texture) |
| **GLDM** | 14 | Gray Level Dependence Matrix (texture) |
| **NGTDM** | 5 | Neighboring Gray Tone Difference Matrix (texture) |

**Total Features:** ~107 (original mode)

#### Stage 4: Quality Control
- Data validation and error handling
- Missing value detection
- Success rate reporting

**Output:**
- CSV file with complete feature matrix
- Processing logs and quality metrics

---

## Dataset Requirements

### Directory Structure

Your dataset must follow this standardized organization:

```
base_folder/
├── PATIENT-001/
│   ├── CT/                    # Or "CT SCAN" (configurable)
│   │   ├── slice001.dcm
│   │   ├── slice002.dcm
│   │   └── ...
│   └── RTSTRUCT/
│       └── RS.1.2.3.dcm
├── PATIENT-002/
└── ...
```

### DICOM Requirements

**CT Series:**
- Modality: CT
- Consistent slice spacing
- Complete series (no missing slices)
- Hounsfield Unit calibration

**RT Structure Sets:**
- Modality: RTSTRUCT
- Contains target tumor structures (GTV, CTV, etc.)
- Spatially registered with CT series
- Valid contour geometry

### Supported Structure Names

Common nomenclature patterns:
- **Primary Tumor**: GTV, GTV-1, GTV-Primary, GTV_1
- **Clinical Target**: CTV, CTV-High, CTV-Low
- **Planning Target**: PTV, PTV-1, PTV54, PTV60
- **Organs at Risk**: Lung_L, Lung_R, Heart, Esophagus, SpinalCord

---

## Usage Guide

### Basic Usage

```python
# 1. Import libraries
import os
from pathlib import Path

# 2. Configure paths
base_folder = "/path/to/maastro_dataset/Images"
os.chdir(base_folder)

# 3. Set analysis parameters
TARGET_ROI = "GTV-1"
FEATURE_MODE = "original"
OUTPUT_CSV = "radiomics_features.csv"

# 4. Run batch processing
df = process_multiple_patients(
    base_folder=base_folder,
    output_csv=OUTPUT_CSV,
    target_roi=TARGET_ROI,
    feature_mode=FEATURE_MODE
)
```

### Advanced Configuration

#### Feature Extraction Modes

```python
# Mode 1: Original (Fast, ~107 features)
FEATURE_MODE = "original"
# Execution time: ~5-10 min per 100 patients
# Use case: Large cohorts, initial exploration

# Mode 2: Wavelet (Comprehensive, ~856 features)
FEATURE_MODE = "wavelet"
# Execution time: ~40-60 min per 100 patients
# Use case: Deep phenotyping, high-dimensional analysis

# Mode 3: LoG Filtering (~428 features)
FEATURE_MODE = "log"
# Execution time: ~20-30 min per 100 patients
# Use case: Multi-scale texture analysis

# Mode 4: All Transformations (~2000+ features)
FEATURE_MODE = "all"
# Execution time: ~2-3 hours per 100 patients
# Use case: Comprehensive feature discovery
```

#### Custom Structure Selection

If your dataset has multiple GTV variants:

```python
# Option 1: Accept any GTV
for s in structures:
    if 'GTV' in s.upper():
        selected_structure = s
        break

# Option 2: Priority-based selection
gtv_priority = ['GTV-1', 'GTV', 'GTV-Primary']
for preferred in gtv_priority:
    if preferred in structures:
        selected_structure = preferred
        break
```

### Single Patient Testing

Before batch processing, test on one patient:

```python
test_patient = "PATIENT-001"

features = process_patient_folder(
    test_patient,
    target_roi="GTV-1",
    feature_mode="original"
)

# Save test results
import pandas as pd
df_test = pd.DataFrame([features])
df_test.to_csv("test_output.csv", index=False)

print(f"Features extracted: {len(features) - 2}")
```

---

## Output Description

### Feature Matrix Structure

The output CSV file contains:

```
PatientID | ROI_Name | original_shape_Volume | original_firstorder_Mean | ...
----------|----------|----------------------|-------------------------|-----
PATIENT-001 | GTV-1  | 15847.3             | -142.5                  | ...
PATIENT-002 | GTV-1  | 22103.8             | -156.2                  | ...
```

### Feature Naming Convention

Features follow PyRadiomics nomenclature:

```
{imageType}_{featureClass}_{featureName}

Examples:
- original_shape_Volume
- original_firstorder_Mean
- original_glcm_Contrast
- wavelet-LHL_glcm_Correlation  (if wavelet mode enabled)
```

### Feature Categories

**Shape Features (14):**
- `Volume`, `SurfaceArea`, `Sphericity`, `Compactness`
- `MajorAxisLength`, `MinorAxisLength`, `LeastAxisLength`
- `Elongation`, `Flatness`, `SphericalDisproportion`

**First-Order Features (18):**
- `Mean`, `Median`, `StandardDeviation`, `Variance`
- `Skewness`, `Kurtosis`, `Energy`, `Entropy`
- `Minimum`, `Maximum`, `Range`, `RootMeanSquared`

**Texture Features (~75):**
- GLCM: Contrast, Correlation, Energy, Homogeneity, etc.
- GLRLM: Run length patterns and distributions
- GLSZM: Size zone characteristics
- GLDM: Dependence metrics
- NGTDM: Gray tone differences

---

## Configuration

### Modifying Dataset Structure

If your dataset uses different folder names:

```python
# In process_patient_folder() function
ct_folder = Path(patient_folder) / "CT SCAN"  # Change from "CT"
rtstruct_folder = Path(patient_folder) / "STRUCTURES"  # Change from "RTSTRUCT"
```

### PyRadiomics Settings

Customize feature extraction parameters:

```python
from radiomics import featureextractor

extractor = featureextractor.RadiomicsFeatureExtractor()

# Modify settings
extractor.settings['binWidth'] = 25  # Intensity bin width
extractor.settings['resampledPixelSpacing'] = [1, 1, 1]  # Resampling
extractor.settings['interpolator'] = 'sitkBSpline'  # Interpolation method

# Disable specific feature classes
extractor.disableAllFeatures()
extractor.enableFeatureClassByName('firstorder')
extractor.enableFeatureClassByName('shape')
```

---

## Troubleshooting

### Common Issues

#### 1. Structure Not Found Error

```
ValueError: ROI 'GTV-1' not found in any RTSTRUCT file
```

**Solution:**
- Run `check_RTstructure_name.ipynb` to identify correct structure names
- Update `TARGET_ROI` parameter to match your dataset
- Check for case sensitivity (e.g., "GTV-1" vs "gtv-1")

#### 2. Dimension Mismatch

```
ValueError: Mask size (512, 512, 120) doesn't match CT size (512, 512, 118)
```

**Solution:**
- Verify CT and RTSTRUCT are from same imaging session
- Check for missing CT slices
- Ensure proper spatial registration

#### 3. Memory Errors

```
MemoryError: Unable to allocate array
```

**Solution:**
- Process in smaller batches
- Use "original" mode instead of "all"
- Increase system RAM or use high-memory computing resources

#### 4. Empty RTSTRUCT Folder

```
Error: No DICOM files in RTSTRUCT directory
```

**Solution:**
- Verify RTSTRUCT files are .dcm format
- Check file permissions
- Ensure complete data transfer from source

### Performance Optimization

**For Large Datasets (>500 patients):**

1. **Use Original Mode**: Faster execution, sufficient for most analyses
2. **Parallel Processing**: Modify code to use multiprocessing
3. **Checkpoint System**: Save intermediate results periodically
4. **Cloud Computing**: Consider AWS/GCP for scalability

---

## Citation

If you use this pipeline in your research, please cite:

### Software

```bibtex
@software{hnc_radiomics_pipeline,
  author = {Shaikh, Hasan},
  title = {HNC Radiomics: Quantitative Imaging Analysis Pipeline},
  year = {2025},
  url = {https://github.com/hash123shaikh/hnc-radiomics}
}
```

### PyRadiomics Framework

```bibtex
@article{vanGriethuysen2017,
  title={Computational Radiomics System to Decode the Radiographic Phenotype},
  author={van Griethuysen, Joost JM and Fedorov, Andriy and Parmar, Chintan and Hosny, Ahmed and Aucoin, Nicole and Narayan, Vivek and Beets-Tan, Regina GH and Fillion-Robin, Jean-Christophe and Pieper, Steve and Aerts, Hugo JWL},
  journal={Cancer Research},
  volume={77},
  number={21},
  pages={e104--e107},
  year={2017},
  publisher={AACR}
}
```

### IBSI Guidelines

```bibtex
@article{Zwanenburg2020,
  title={The Image Biomarker Standardization Initiative: Standardized Quantitative Radiomics for High-Throughput Image-based Phenotyping},
  author={Zwanenburg, Alex and Valli{\`e}res, Martin and Abdalah, Mahmoud A and others},
  journal={Radiology},
  volume={295},
  number={2},
  pages={328--338},
  year={2020},
  publisher={Radiological Society of North America}
}
```

---

## Contributing

We welcome contributions to improve this pipeline!

### How to Contribute

1. **Fork the Repository**
2. **Create Feature Branch**: `git checkout -b feature/improvement`
3. **Commit Changes**: `git commit -m "Add new feature"`
4. **Push to Branch**: `git push origin feature/improvement`
5. **Submit Pull Request**

### Areas for Contribution

- [ ] Multi-threading for batch processing
- [ ] Additional feature extraction modes
- [ ] Integration with other imaging modalities (MRI, PET)
- [ ] Automated quality control metrics
- [ ] Visualization tools for feature analysis
- [ ] Docker containerization
- [ ] Cloud deployment scripts

### Reporting Issues

Found a bug? Please report it:
- Use GitHub Issues
- Provide minimal reproducible example
- Include error messages and system information

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

```
MIT License

Copyright (c) 2025 Hasan Shaikh

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
```

---

## Acknowledgments

- **PyRadiomics Development Team** for the feature extraction framework
- **MAASTRO Clinic** for dataset resources
- **Image Biomarker Standardization Initiative (IBSI)** for standardization guidelines
- **Medical Imaging Community** for open-source tools and collaboration

---

## Contact

**Author**: Hasan Shaikh  
**Email**: hasanshaikh3198@gmail.com  
**GitHub**: [@hash123shaikh](https://github.com/hash123shaikh)  
**LinkedIn**: [Connect with me](https://www.linkedin.com/in/hasan-shaikh)

For questions, suggestions, or collaboration opportunities, please reach out!

---

## Related Projects

- [PyRadiomics](https://github.com/AIM-Harvard/pyradiomics)
- [RT-Utils](https://github.com/qurit/rt-utils)
- [SimpleITK](https://github.com/SimpleITK/SimpleITK)
- [IBSI](https://theibsi.github.io/)

---

**Last Updated**: January 2025  
**Version**: 1.0.0  
**Status**: Active Development

---

<p align="center">
  Made with ❤️ for the Medical Imaging Research Community
</p>
