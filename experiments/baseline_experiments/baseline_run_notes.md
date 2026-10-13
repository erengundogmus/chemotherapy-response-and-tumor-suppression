# Baseline Run Notes

## Goal

Run the reference implementation and record the outputs for different observation windows.

## Reference Repository

https://github.com/kisblilla/piecewise-pinn-tumor-growth

## Local Setup

Operating system: Windows  
Python version: 3.12.10  
Environment: Local virtual environment inside the reference repository

## Commands Used

```bash
cd C:\pinn-project
mkdir reference-code
cd reference-code
git clone https://github.com/kisblilla/piecewise-pinn-tumor-growth.git
cd piecewise-pinn-tumor-growth
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe main.py
```

## Output Location

The reference script generated outputs in:

```text
C:\pinn-project\reference-code\piecewise-pinn-tumor-growth\results_extended
```

The generated figures were copied into:

```text
results\baseline\
```

Saved folders:

```text
105_days/
100_days/
50_days/
30_days/
20_days/
```

Each folder contains:

```text
00_init.png
01_phase1_end.png
02_final.png
03_b_convergence.png
```

## Recorded Results

| Observation Window | True b          | Predicted b |
| 105 days           | 13.112122459437 | 12.3982     |
| 100 days           | 13.112122459437 | 12.4101     |
| 50 days            | 13.112122459437 | 12.4604     |
| 30 days            | 13.112122459437 | 12.4411     |
| 20 days            | 13.112122459437 | 12.3405     |

## Notes

Python 3.15 was not compatible with PyTorch, so the environment was recreated with Python 3.12.10.

The original Google Colab download section was removed from the local copy, and outputs were saved locally instead.