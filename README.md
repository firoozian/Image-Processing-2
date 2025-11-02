# 🧭 Minimal Multi-Image Angle Picker (OpenCV)

Interactive tool to **stack multiple images**, pick **triplets of points** (anchor + two rays), and compute **inside/outside angles**. Results are shown on a right sidebar and exported to Excel.

## ✨ Features
- Load a fixed list of image files from the project folder (`IMAGE_PATHS`)
- Resize & stitch images horizontally with a **measurement panel**
- Click to place points in triplets → draw lines & compute angles
- Live sidebar: per-measurement angle + running average (deg)
- Export to `angles_output.xlsx` (with per-row values & averages)
- Zoom, reset, quick snapshot (PNG)

## 🔧 Requirements
```bash
pip install opencv-python numpy pandas openpyxl
```

## ▶️ Run
```bash
python main.py
```
> Make sure your images (e.g., `nlt1.jpg`, `nrt1.jpg`, …) are **in the same folder** as the script.  
> If you use a notebook, the **working directory** must be that folder.

## 🖱 How to Measure
1. **Left-click** to add points in groups of **three**:
   - 1st click: red dot (anchor) with index label  
   - 2nd & 3rd clicks: blue lines from the anchor to each point
2. After each triplet, the tool computes:
   - **Inside angle** (0–180) and **Outside angle** (360–inside)
   - Updates the sidebar and the Excel file

## ⌨️ Shortcuts
- `+` / `=` → Zoom in  
- `-` → Zoom out  
- `r` → Reset canvas & measurements  
- `s` → Save a `snapshot.png` of the current view  
- `q` → Quit

## 📁 Output
- **Excel:** `angles_output.xlsx`
  - Columns: No, Inside Angles (°), Outside Angles (°), Avg Inside (°), Avg Outside (°)
- **Snapshot (optional):** `snapshot.png` (on demand)

## ⚠️ Troubleshooting
- **“Not found or unreadable”**: image names/paths don’t match the files next to the script.
- In Jupyter, check your working directory:
  ```python
  import os; print(os.getcwd())
  ```
  Move images there or `cd` into the project folder before running.

## 📝 Config (inside the script)
```python
IMAGE_PATHS = ["nlt1.jpg","nlt2.jpg","nrt1.jpg","nrt2.jpg","nrt3.jpg","nrt4.jpg","nrt5.jpg","nrt6.jpg","nrt7.jpg"]
SCALE_PCT = 30         # resize percentage
BOX_W, BOX_H = 275, 200
OUT_XLSX = "angles_output.xlsx"
OUT_PNG = "Via_Angles.png"
```
> Edit `IMAGE_PATHS` or filenames to match your dataset. Keep images in the **project folder** for zero-hassle loading.

---
Built with Python, OpenCV, NumPy, and Pandas.
