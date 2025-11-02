# 🧭 Minimal Multi-Image Angle Picker (OpenCV)

Interactive tool to **stack multiple images**, pick **triplets of points** (anchor + two rays), and compute **inside/outside angles**. Results are shown on a right sidebar and exported to Excel.

## ✨ Features
- Load a fixed list of image files from the project folder (`IMAGE_PATHS`)
- Resize & stitch images horizontally with a **measurement panel**
- Click to place points in triplets → draw lines & compute angles
- Live sidebar: per-measurement angle + running average (deg)
- Export to `angles_output.xlsx` (with per-row values & averages)
- Zoom, reset, quick snapshot (PNG)

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
