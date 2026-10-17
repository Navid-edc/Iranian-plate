# Vision

Coursework and a small project from an undergraduate **Fundamentals of Computer Vision** course (Spring 2024), built with Python, OpenCV, NumPy and PyTorch.

The repo has two parts:

1. **Lab exercises** (`Pr1`–`Pr6`, `Lab7`, `lab8`): classic image-processing topics, from pixel operations to Hough transforms and geometric transformations.
2. **License plate project** (repo root, `Four/`, `Git/three/`): a CNN that regresses the four corners of a license plate in an image, with an experimental follow-up for reading the plate characters.

## Repository structure

```
Vision/
├── Pr1/        NumPy & Matplotlib basics (image flip, matrix multiplication, mean-centering)
├── Pr2/        Color channels, image blending, RGB → grayscale conversion
├── Pr3/        Video I/O (reversing a video), histogram stretching / contrast enhancement
├── Pr4/        Noise and filtering (Gaussian noise, box / Gaussian / bilateral blur)
├── Pr5/        Image gradients and edges (Sobel, edge detection, webcam input)
├── Pr6/        Thresholding, morphology, connected components, bean counting, image differencing
├── Lab7/       Hough line and Hough circle transforms (coin detection)
├── lab8/       Geometric transforms: translation, rigid, similarity, affine, perspective
│
├── loader.py / new_loader.py   Dataset loading, resizing and augmentation
├── regressor.py                CNN that predicts the 4 plate corners
├── train.py                    Train the corner regressor (80/20 split)
├── k_fold_train.py             5-fold cross-validation training
├── evaluate.py                 Evaluate a trained model and visualize predictions
├── k_fold_evaluate.py          Evaluate the cross-validation models
├── check.py                    Predict and draw corners on a single image
├── *.ipynb                     Notebooks used while developing the above
├── Four/                       Plate extraction + character classifier (experimental)
└── Git/three/                  Earlier snapshot of the loader and evaluation notebook
```

## Lab exercises

Each lab folder is self-contained: the scripts read the sample images from the same folder, so run them from inside it.

```bash
cd Pr2
python task3.py
```

Most scripts open OpenCV windows (`cv2.imshow`), so they need a machine with a display. In several of them, pressing a key moves to the next step and `q` quits.

## License plate corner detection

### Idea

Given an image of a car, predict the 4 corners of its license plate as 8 normalized coordinates. Once the corners are known, a perspective transform can warp the plate into a flat rectangle (the same technique practiced in `lab8`), which is the first step toward reading it.

### Data format

The dataset is **not included** in this repo. The loader expects:

- `images/`: `.jpg` images
- `labels/`: one `.txt` file per image with the same base name, containing 9 numbers in YOLO-style normalized form: `class x1 y1 x2 y2 x3 y3 x4 y4`

Images are resized to 256 × 256 and scaled to [0, 1]; corner coordinates are normalized to [0, 1]. `loader.py` can also generate augmented copies (random translation and blur); `new_loader.py` adds rotation, contrast, crop and perspective jitter.

### Model

`regressor.py` defines `CNNModel`:

- 4 convolution blocks (32 → 64 → 128 → 256 channels, 3×3 kernels, ReLU, 2×2 max-pooling)
- 2 fully connected layers (256 → 128) with dropout 0.5
- 8 outputs (x, y for each of the 4 corners)

### Training and evaluation

`train.py` uses MSE loss, Adam (lr = 0.001), batch size 32, 50 epochs and an 80/20 train/validation split, and saves the weights to `model.pth`. `k_fold_train.py` does the same with 5-fold cross-validation.

`evaluate.py` reports the test MSE and a corner accuracy (the fraction of corners predicted within 10 pixels on the 256 × 256 image), and saves plots of true vs. predicted corners.

### Running it

The scripts currently contain hard-coded dataset paths in their `__main__` blocks. Edit them to point to your own data first, then:

```bash
python train.py
python evaluate.py
```

## Status and limitations

- The corner regressor (`regressor.py`, `train.py`, `evaluate.py`, `loader.py`) is the main working part.
- The character-recognition experiment in `Four/` (`classifier.py`, `train_classifier.py`, `integrate.py`) is **unfinished**: it is a work-in-progress and does not run end to end as it stands.
- `check.py` is a quick visualization helper, and its preprocessing size differs from the training size in `loader.py`, so adjust it before relying on it.
- Datasets and trained weights (`.pth`) are not included.
- Results (loss and accuracy numbers, example outputs) have not been added yet.

## Requirements

Python 3.10+ and the packages in `requirements.txt`:

```bash
pip install -r requirements.txt
```

## License

No license has been chosen yet. Add one (for example MIT) if you want others to reuse the code.
