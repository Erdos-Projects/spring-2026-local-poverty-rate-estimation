# Quickstart Guide

## 1. Get the Code
Choose one of the following methods to get the project files.

### Option A: Git Clone
```bash
git clone https://github.com/zhenyu-yue/erdos-poverty-sae.git
cd erdos-poverty-sae
```

### Option B: Download ZIP
1. Go to: https://github.com/zhenyu-yue/erdos-poverty-sae
2. Click the green **Code** button and select **Download ZIP**.
3. Extract the ZIP file and open the folder in your terminal or editor.

## 2. Install Requirements
Choose one of the following methods to set up your Python environment.

### Option A: Pip (Direct Install)
Install dependencies directly into your current active environment:
```bash
pip install -r requirements.txt
```

### Option B: Conda Environment
Create a dedicated environment named `poverty-sae`:
```bash
conda env create -f environment.yml
conda activate poverty-sae
```

## 3. Run the Pipeline
Open the project folder in VS Code or Jupyter Notebook.

**Step 1:** Open and run **`00_download_data.ipynb`**.
* This notebook automates the download of the ACS PUMS data for DC, MD, and VA.
* **Note:** The SNAP dataset is already included in the `data/` folder of this repository.