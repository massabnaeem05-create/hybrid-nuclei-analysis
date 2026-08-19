# Hybrid Pipeline for Nuclei Microscopy Image Analysis

 A hybrid biomedical image-analysis pipeline that combines a local vision-language model (VLM), classical image processing, and a PyTorch U-Net segmentation network for fluorescence microscopy nuclei analysis.

**Pipeline:** raw image → segmentation → quantitative region features → structured JSON record → short narrative

Full analysis, figures, and discussion are provided in PDF report.

---

## Repository structure

```text
.
├── README.md
├── requirements.txt
├── Hybrid Biomedical Image.pdf       # final report
├── Nuclei_assignment.ipynb           # full end-to-end notebook
├── outputs/
│   ├── eda_intensity_histogram.png
│   ├── task4_hybrid_results.csv
│   ├── eda_sample_grid.png
│   ├── task2_segmentation_demo.png
│   ├── task3_prediction_panels.png
│   ├── task3_training_curves.png
│   ├── task4_robustness_trace.png
│   ├── task4_hybrid_results.csv
│   ├── task4_hybrid_results_with_ground_truth.csv
│   ├── task4_robustness_results.csv
       
```
---

## Dataset

The project uses a fluorescence microscopy nuclei-segmentation dataset with 256×256 images and paired binary masks.

| Split | Images |
|---|---:|
| train | 80 |
| validation | 20 |
| test | 12 |

The test set is used for the final hybrid pipeline and is not used during U-Net training.

---

## Model and VLM

### U-Net

A PyTorch U-Net is trained for binary nuclei segmentation.

- Input: grayscale image
- Training images: 80
- Validation images: 20
- Epochs: **40**
- Batch size: **8**
- Optimizer: **Adam**
- Loss: **BCE + Dice**
- Scheduler: **ReduceLROnPlateau**

### Local VLM

The assignment specifies `llama3.2-vision`. During implementation, the required model produced an unsupported `mllama` architecture error in the tested Ollama environment, so **`llava` was used as the multimodal model** for the VLM steps.

```bash
ollama pull llava
```

This substitution should be considered when reproducing the VLM results.

---

## How to run

The notebook is designed to run top-to-bottom in **Google Colab**. A GPU runtime is recommended for U-Net training.

### 1. Clone this repository

```bash
git clone https://github.com/massabnaeem05-create/hybrid-nuclei-analysis.git
cd hybrid-nuclei-analysis
```

### 2. Install Python dependencies

```bash
pip install -r requirements.txt
```

If `requirements.txt` is not available:

```bash
pip install torch torchvision
pip install numpy pandas matplotlib
pip install scikit-image scikit-learn pillow
pip install ollama
```

### 3. Install and start Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama serve &
pip install -q ollama
ollama pull llava
```

### 4. Open the notebook

Open:

```text
Nuclei_assignment.ipynb
```

and run the cells from top to bottom.

The notebook is organized into:

- **Task 1** : preprocessing, EDA, image samples/histograms, naive and engineered VLM prompts
- **Task 2** : Otsu thresholding, morphology, connected components, `regionprops_table`, and numbers-first LLM interpretation
- **Task 3** : U-Net definition, training, validation Dice/IoU, and prediction visualizations
- **Task 4** : U-Net → regionprops → LLM hybrid pipeline on 12 unseen test images
- **Extension** : Blur and low-contrast robustness experiments

### 5. GPU recommendation

For reasonable U-Net training time in Colab:

```text
Runtime → Change runtime type → T4 GPU
```

---

## Reproducing report figures and numbers

The notebook contains the code used to generate the figures and numerical results presented in the report.

The main generated outputs include:

- Exploratory sample images
- Pixel-intensity histograms
- Otsu segmentation visualization
- U-Net training curves
- U-Net validation prediction panels
- Hybrid test-set results
- Corruption/robustness visualizations
- `task4_hybrid_results.csv`

The VLM responses can vary slightly between runs because multimodal LLM generation is non-deterministic. The classical image-processing results and U-Net evaluation depend on the image data and model state.

---

## Key results summary

- **Otsu demonstration (`train_000`)**: detected **9/10** nuclei.
- **Best validation Dice**: **0.9972**.
- **IoU at the best-Dice epoch**: **0.9945**.
- **Training**: 40 epochs using BCE + Dice loss.
- **Test set**: **12/12** images processed by the hybrid pipeline.
- **Mean absolute instance-count error**: **11.75**.
- **Worst clean test undercount**: **40 objects** on `test_010`.
- **Low-contrast robustness**: predicted count collapsed to **1** for both `test_000` and `test_004`.
- **Blur robustness**: predicted counts decreased from 8→7 on `test_000` and 43→9 on `test_004`.

### Important interpretation

The U-Net achieves excellent **pixel-level segmentation** performance, but connected-component labeling still undercounts touching/closely packed nuclei in dense and clustered images.

The density class used by the hybrid pipeline is a simple count-based heuristic:

```text
n_objects < 15       → sparse
15 ≤ n_objects < 40  → normal
n_objects ≥ 40       → dense
```

It does not separately predict the dataset's `clustered` ground-truth category.

---

```bash
pip install -r requirements.txt
```

Ollama must be installed separately.

---

The report documents the complete analysis, including the VLM comparison, classical segmentation, U-Net evaluation, hybrid test-set results, and robustness analysis.
