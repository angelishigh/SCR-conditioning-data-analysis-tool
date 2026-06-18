For each participant and each CS trial, the script:

- Cuts an EDA segment from **−2 to +8 seconds** around CS onset  
- Baseline-corrects using **−2 to 0 seconds**  
- Finds the **peak SCR amplitude** in the **1–6 second** window after CS onset  
- Clips negative values to 0 (non-responses)  
- Computes the **square root of the peak** (`SCR_sqrt`)  

Then it:

- Labels each trial with **Phase** (Habituation, Acquisition, Extinction), **Stimulus** (Pain, Food, Neutral, Extra), **Block** (1–3), and **Trial_in_Phase**  
- Saves all trials to `SCR_peaks_all.csv`  
- Computes **per-subject averages of `SCR_sqrt`** for each Phase × Stimulus × Block combination and saves them to `SCR_peaks_all_averages.csv` (this is what you use for the mixed ANOVA).  

This matches the proposal definition: SCR is “difference between onset and peak, square-root transformed and averaged per CS type and phase”. [file:316][file:314]

---

## Requirements

- Python 3.9+  
- Packages:
  - `mne`
  - `numpy`
  - `pandas`

Install once:

```bash
pip install mne numpy pandas
```

---

## Data format

The script expects **BrainVision Core format** with one SCR channel:

For each participant:

- `XX.vhdr` – header (used as the main entry point)  
- `XX.vmrk` – markers (S-codes for stimuli)  
- `XX.eeg`  – signal data (contains the EDA/SCR signal)  

All three must:

- Live in the same folder  
- Share the same base name (e.g. `7.vhdr`, `7.vmrk`, `7.eeg`)  

The SCR channel in the `.vhdr` is assumed to be called `EDA` (you can change this).

---

## Study design assumptions

The marker map is hard‑coded for the Group 31 conditioning task:

- **Habituation** (9 trials; 3 per CS):  
  - `S104` – Pain  
  - `S120` – Food  
  - `S136` – Neutral  

- **Acquisition** (27 trials; 9 per CS):  
  - `S152` – Pain  
  - `S160` – Food  
  - `S176` – Neutral  

- **Extinction** (36 trials; 9 per CS):  
  - `S192` – Pain  
  - `S208` – Food  
  - `S224` – Neutral  
  - `S240` – Extra  

Block logic: within each Phase × Stimulus, **every 3 trials** form one block:

- Trials 1–3 → Block 1  
- Trials 4–6 → Block 2  
- Trials 7–9 → Block 3  

If your experiment uses different codes, update the `MARKER_MAP` dictionary accordingly.

---

## How to use

1. **Organize your files**

   ```text
   C:\SCR analysis\
       scr_analysis.py
       SCR_data\
           7.vhdr
           7.vmrk
           7.eeg
           8.vhdr
           8.vmrk
           8.eeg
           ...
   ```

2. **Edit settings at the top of `scr_analysis.py`**

   ```python
   DATA_FOLDER = r"C:\SCR analysis\SCR_data"
   OUTPUT_CSV  = r"C:\SCR analysis\SCR_peaks_all.csv"

   SCR_CHANNEL = "EDA"   # must match your .vhdr channel name
   LOWPASS_HZ  = 5.0
   TMIN        = -2.0
   TMAX        = 8.0
   BASELINE    = (-2.0, 0.0)
   PEAK_START  = 1.0
   PEAK_END    = 6.0
   INVERT      = True     # set False if your SCR naturally goes up
   ```

3. **Run the script**

   ```bash
   cd "C:\SCR analysis"
   python scr_analysis.py
   ```

4. **Check the outputs**

   You should now have:

   - `SCR_peaks_all.csv`  
   - `SCR_peaks_all_averages.csv`  

   Open them in Excel/JASP/SPSS or R to verify the columns look sensible.

---

## Output files

### 1. Trial-level file – `SCR_peaks_all.csv`

One row per CS trial.

Columns:

- `Subject` – file base name (e.g. `7`)  
- `Phase` – Habituation / Acquisition / Extinction  
- `Stimulus` – Pain / Food / Neutral / Extra  
- `Trial_in_Phase` – 1–9 per Phase × Stimulus  
- `Block` – 1–3 (groups of 3 trials)  
- `SCR_peak_raw` – peak amplitude (µS), baseline-corrected  
- `SCR_latency_s` – time of the peak (s after CS onset)  
- `SCR_peak_nonneg` – same as `SCR_peak_raw`, but negatives clipped to 0  
- `SCR_sqrt` – square-root transformed amplitude (this is what you usually analyse)

### 2. Condition averages – `SCR_peaks_all_averages.csv`

One row per Subject × Phase × Stimulus × Block.

Columns:

- `Subject`  
- `Phase`  
- `Stimulus`  
- `Block`  
- `SCR_sqrt_mean` – mean √SCR across trials in that condition  
- `SCR_sqrt_sd` – standard deviation of √SCR within that condition  
- `n_trials` – number of trials in that condition (should be 3 in this design)

Use this file directly for **mixed ANOVAs** with:

- Between-subject factor: Sleep condition (deprived vs control)  
- Within-subject factors: Phase, Stimulus, Block  

(Assuming you merge in sleep group labels separately.)


## Adapting to other experiments
To adapt this pipeline to another conditioning experiment
1. Change `MARKER_MAP` to your own marker codes and labels.  
2. Adjust `TMIN`, `TMAX`, `PEAK_START`, and `PEAK_END` if your timing differs.  
3. Update `SCR_CHANNEL` if your EDA channel has another name (e.g. `"GSR"`).  
4. Optionally change the grouping logic in the averaging step if your block structure differs.
## Disclaimer
This is a lightweight teaching/research tool meant for student projects. Always visually inspect a few participants’ data and compare against your lab’s SCR guidelines (e.g. Boucsein et al., 2012) before relying on the numbers for publication. [file:314][web:387]
