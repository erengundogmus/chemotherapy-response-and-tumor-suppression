# Observation Window Experiment Analysis

## Executive Summary

This project studies tumor growth under chemotherapy with a piecewise Physics-Informed Neural Network (PINN). The model combines observed total tumor volume with the governing tumor-growth equations and estimates the treatment-efficacy parameter $b$. In the model equations, $b$ controls the drug-dependent transfer from living tumor volume to dead tumor volume; a larger $b$ therefore represents a stronger treatment effect under the same drug concentration.

The experiment compares nominal observation windows of 105, 100, 50, 30, and 20 days to examine how the amount of observed dosing and tumor-dynamics information affects parameter inference. The repository records a true value of $b=13.112122459437$. Every recorded estimate is below that target. Based on the recorded values, the 50-day run has the lowest absolute error, $0.651722$ (4.9704%), and the 20-day run has the highest, $0.771622$ (5.8848%). Performance does not deteriorate monotonically as the window becomes shorter: the 100-, 50-, and 30-day estimates are all closer to the target than the recorded 105-day baseline estimate, while the 20-day result is worse.

The final trajectory figures show that the PINN follows the synthetic total-tumor trajectory and measurement points closely within each plotted window. The independently solved ODE trajectories using the estimated $b$ values remain visibly above the ground-truth trajectory, which is consistent with underestimating treatment efficacy. The convergence plots rise toward the target but level off below it; none reaches the true value.

These results come from one recorded training run per window. Random initialization, stochastic optimization, and differences in the number of piecewise segments can therefore influence the ranking, so the results do not establish robustness or a general advantage for a particular window. There are also two provenance inconsistencies in the stored 105-day baseline: its recorded estimate is 12.3982, while `02_final.png` displays 12.3568, and its ground-truth and measurement traces extend beyond day 105. The appropriate next step is to rerun each window with multiple controlled random seeds, save machine-readable metrics and parameter histories, and report means and standard deviations together with trajectory errors.

## Project and Model Overview

The reference model represents four coupled states: living tumor volume $x_1$, dead tumor volume $x_2$, blood drug concentration $x_3$, and tissue drug concentration $x_4$. The two tumor-state equations are

$$
\frac{dx_1}{dt}=(a-n)x_1-\frac{b x_1 x_3}{ED_{50}+x_3},
$$

$$
\frac{dx_2}{dt}=n x_1+\frac{b x_1 x_3}{ED_{50}+x_3}-w x_2.
$$

The term containing $b$ removes volume from the living compartment and adds it to the dead compartment according to the drug concentration. Thus, $b$ is the inferred treatment-efficacy parameter. The available measurements constrain total tumor volume, $x_1+x_2$, rather than both internal states separately.

Chemotherapy doses create instantaneous changes in drug concentration at known dose times. A single smooth neural network can have difficulty representing dynamics separated by these discontinuous inputs. The piecewise PINN therefore assigns a small neural network to each interval between successive dose boundaries. A continuity loss connects the predicted tumor states at adjacent boundaries, while drug concentration is calculated analytically from the dosing history.

Training combines four sources of information: measured total tumor volume, residuals of the tumor ODEs at collocation points, continuity between piecewise networks, and the initial condition. Automatic differentiation supplies the time derivatives used in the physics residuals. Training has two phases. During the 10,000-epoch warm-up, $b$ is fixed at 10.0 while the networks learn the trajectory shape. During the 25,000-epoch discovery phase, the networks and $b$ are optimized together. This construction lets the model use both the data and the biological equations rather than treating parameter estimation as an unconstrained curve-fitting problem.

## Objective of the Experiment

The experiment asks how shortening the observation period affects recovery of $b$ and the visible behavior of the trained model. A longer window contains more dose events, more measurements, a longer decay tail, and more distinct response regimes, but it also creates more piecewise networks and a larger optimization problem. A shorter window reduces both the available evidence and the number of segments. Comparing several windows therefore tests whether less temporal information necessarily produces a worse estimate and helps identify where parameter identifiability may begin to weaken.

The 105-day result is treated as the baseline for numerical comparisons because it is the longest nominal window defined by the experiment records. This designation is comparative, not a claim that the 105-day run is the most accurate or that its stored artifacts are free of provenance issues.

## Experimental Setup

The repository contains completed result folders for nominal 105-, 100-, 50-, 30-, and 20-day windows. The observation-window patch retains only dose times at or before the selected end day. On that intended basis, the windows contain 29, 27, 13, 7, and 4 dose events, respectively. The stored 105-day figures require a qualification: their ground-truth and measurement traces extend to approximately day 119, matching the unpatched reference code's inclusion of later dose boundaries, while the plotted PINN trajectory stops at day 105.

The true parameter and recorded estimates are:

- True $b$: 13.112122459437
- 105 days: 12.3982
- 100 days: 12.4101
- 50 days: 12.4604
- 30 days: 12.4411
- 20 days: 12.3405

This analysis uses only the committed PNG outputs and values recorded in `results/baseline/README.md` and `experiments/baseline_experiments/baseline_run_notes.md`. No training was rerun, and no figure was regenerated or modified. Numerical error calculations use the recorded values above. For four windows, the recorded value agrees with the value printed in `02_final.png`. For the 105-day window, the figure prints 12.3568 instead of the recorded 12.3982; the discrepancy is preserved and reported rather than silently reconciled.

## Understanding the Generated Figures

Each experiment directory contains the same four outputs:

- `00_init.png` shows the untrained piecewise network at epoch 0, with $b$ initialized to 10.0. Its left panel compares total-tumor ground truth, measurement points, the initial PINN trajectory, and an ODE solution evaluated with $b=10.0$. Its right panel compares the living and dead internal states. Across the windows, the initial PINN curves are near zero or irregular and do not yet reproduce the reference dynamics.
- `01_phase1_end.png` shows the result after the 10,000-epoch warm-up with $b$ still fixed at 10.0. The total PINN trajectory has learned the main measured shape, and the internal-state curves are much closer to their reference counterparts. The green ODE curve still uses $b=10.0$ and visibly overpredicts total tumor volume.
- `02_final.png` shows the result after the 25,000-epoch parameter-discovery phase, at a displayed total epoch count of 35,000. The blue PINN trajectory is compared with ground truth and measurements, while the green dashed curve is an ODE solution using the final estimated $b$. The ODE curve remains above the ground truth in all five windows, whereas the PINN trajectory closely follows the measured total-tumor curve. This distinction matters: visual trajectory fit by the flexible PINN does not by itself prove exact parameter recovery.
- `03_b_convergence.png` plots the estimated $b$ during the discovery phase, with the red dashed horizontal line marking the true value. All five curves move upward from the neighborhood of 10 and flatten below the target. Some curves show a short initial dip or small late variations, but none displays large-scale divergence in the saved plot.

These descriptions are limited to features visible in the figures and behavior documented by the reference code. The PNG files do not provide saved loss histories, uncertainty intervals, or numerical trajectory-error metrics.

## Recorded Parameter Estimates

| Observation window | True $b$ | Recorded predicted $b$ | Direction relative to truth |
|---:|---:|---:|:---|
| 105 days | 13.112122459437 | 12.3982 | Underestimate |
| 100 days | 13.112122459437 | 12.4101 | Underestimate |
| 50 days | 13.112122459437 | 12.4604 | Underestimate |
| 30 days | 13.112122459437 | 12.4411 | Underestimate |
| 20 days | 13.112122459437 | 12.3405 | Underestimate |

The 105-day row uses the value recorded in the Markdown result logs. Its final PNG instead displays 12.3568, so readers should not treat the table and that particular figure as outputs proven to originate from the same training run.

## Quantitative Error Analysis

Absolute and relative errors are calculated as

$$
\text{absolute error}=|b_{\text{true}}-b_{\text{predicted}}|,
$$

$$
\text{relative error}=\frac{|b_{\text{true}}-b_{\text{predicted}}|}{b_{\text{true}}}\times100.
$$

The baseline change is $b_{\text{predicted, window}}-b_{\text{predicted, 105 days}}$. A positive value is closer to the true $b$ in this experiment because every estimate is below the target.

| Observation window | Recorded predicted $b$ | Absolute error | Relative error | Change from 105-day baseline | Percentage change from baseline |
|---:|---:|---:|---:|---:|---:|
| 105 days | 12.3982 | 0.713922 | 5.4448% | 0.0000 | 0.0000% |
| 100 days | 12.4101 | 0.702022 | 5.3540% | +0.0119 | +0.0960% |
| 50 days | 12.4604 | 0.651722 | 4.9704% | +0.0622 | +0.5017% |
| 30 days | 12.4411 | 0.671022 | 5.1176% | +0.0429 | +0.3460% |
| 20 days | 12.3405 | 0.771622 | 5.8848% | -0.0577 | -0.4654% |

The errors span a narrow range of approximately 4.97% to 5.88%. The 50-day estimate is numerically best, followed by 30, 100, 105, and 20 days. The ordering demonstrates that shortening the window does not produce a regular, monotonic loss of parameter accuracy in these recorded runs. It does not demonstrate that 50 days is intrinsically optimal, because each entry is a single stochastic training result.

## Individual Window Analysis

### 105-Day Window

The nominal 105-day experiment is the comparison baseline. The patched experiment design would include 29 dose events through day 105 and the longest intended tumor-response history. However, the stored figures show ground-truth and measurement points continuing to approximately day 119, while the PINN curve ends at day 105. This makes the exact data scope of this artifact different from the nominal label and from the consistently cropped shorter-window outputs.

The recorded estimate is $b=12.3982$, giving an absolute error of 0.713922 and a relative error of 5.4448%. In `02_final.png`, the blue PINN curve follows the two prominent early peaks and the later decay of the ground-truth total tumor within its plotted prediction range. The green ODE curve using the displayed estimate remains above the ground truth, particularly around the early peaks and the following decay. The internal-state predictions visually overlap their reference curves over most of the PINN range.

The convergence curve dips briefly below its starting neighborhood, rises rapidly, then continues upward more slowly and remains below the red target line. Its endpoint is consistent with the 12.3568 shown in the final PNG, not with the separately recorded 12.3982. The long visible trajectory provides varied dynamics, but the horizon and estimate mismatches prevent this folder from serving as a fully traceable reference run without clarification or a controlled rerun.

### 100-Day Window

The 100-day window contains 27 dose events through day 98 and retains the two strong early total-tumor peaks followed by a long decay. Its estimate is $b=12.4101$, with an absolute error of 0.702022 and a relative error of 5.3540%.

In the final figure, the blue PINN trajectory closely follows the ground truth and measurement markers throughout the window, including the peaks near the early dose-response region and the long approach toward zero. The ODE trajectory using the estimated $b$ is visibly higher than the ground truth after the early peaks but approaches it later in the decay. The internal-state predictions also visually track the reference living and dead components.

The $b$ curve rises smoothly and then continues with a shallow positive slope toward approximately 12.41, without reaching the true-value line. Its main strength is that it combines extensive dose and decay information with internally consistent displayed and recorded estimates. Its limitation is that the remaining parameter bias is still visible despite the strong total-trajectory fit.

### 50-Day Window

The 50-day window contains 13 dose events through day 49. It retains the first large response peak, the intervening decline, the second rise around the late twenties, and the subsequent downward trend, but excludes the later low-volume tail present in the longer windows.

The predicted value is $b=12.4604$. Its absolute error of 0.651722 and relative error of 4.9704% are the lowest among the five recorded runs. The final PINN curve closely follows the ground-truth total-tumor curve and measurement points across the window. The fitted-parameter ODE curve reproduces the broad timing of the peaks but stays above the ground truth, especially around the second peak and later decline. The living/dead PINN states visually follow the corresponding reference states.

The convergence trace rises toward about 12.45, forms a broad plateau, shows a small late decrease, and recovers slightly by the end. It stays below the true-value line but exhibits no large visible instability. The favorable parameter error is a useful observation, not evidence that 50 days is reliably superior across seeds.

### 30-Day Window

The 30-day window contains 7 dose events through day 28. It captures the initial rise, the first major peak, the following decline, and the beginning of the second increase near the end of the window. It does not include the subsequent long decay.

The predicted value is $b=12.4411$, with an absolute error of 0.671022 and a relative error of 5.1176%. In the final figure, the PINN curve follows the ground truth and measurement points across the window, including the late upturn. The ODE curve using the estimated $b$ remains moderately above the total-tumor ground truth after the main peak. The predicted internal states visually track the reference states, including rapid transitions around dose boundaries.

The convergence curve increases smoothly, levels near 12.4, dips slightly late in training, and ends with a small upward trend. It remains below the target without a visible runaway oscillation. This run is numerically closer to the true $b$ than the recorded 105- and 100-day estimates despite having less temporal information, underscoring the non-monotonic single-run result.

### 20-Day Window

The 20-day window contains 4 dose events through day 18. It captures the initial growth, the first major peak near the middle of the window, and only the beginning of the subsequent decline. It therefore contains the fewest measurements, dose boundaries, and distinct post-treatment regimes.

The predicted value is $b=12.3405$. Its absolute error of 0.771622 and relative error of 5.8848% are the highest in this comparison. The final PINN curve follows the visible total-tumor ground truth and measurement points over the short interval. The ODE curve using the estimated $b$ is higher than the ground truth after the peak, and the short horizon does not show whether longer-term decay would be reproduced. The internal-state predictions visually track the reference curves within the plotted interval.

The convergence trace first dips, then rises rapidly and settles near 12.34 with small late fluctuations. It does not approach the target as closely as the other recorded runs. The short run still provides a good visual total-trajectory fit, but its restricted number of doses and response phases gives less information for distinguishing the effect of $b$ from adjustments in the neural-network states and other loss terms.

## Cross-Window Comparison

All five recorded estimates are lower than the true $b$. The 50-day window has the lowest recorded error, while the 20-day window has the highest. The difference between their predicted values is 0.1199, and the difference between their relative errors is approximately 0.9144 percentage points.

There is no monotonic relationship between shorter observation duration and larger parameter error. Moving from 105 to 100 and then 50 days improves the recorded estimate; the error rises slightly at 30 days and more clearly at 20 days. In particular, the 50- and 30-day runs outperform both longer recorded estimates on this one parameter metric. The result should be described as non-monotonic rather than as a general short-window advantage.

The figures share several patterns. Initial PINN curves do not represent the reference trajectory, phase-one training learns the main total-tumor shape while $b$ remains fixed at 10.0, and final PINN curves closely follow the available ground truth and measurements. In every final plot, the ODE trajectory computed from the estimated $b$ lies above the ground-truth total-tumor trajectory over substantial portions of the window. Every convergence trace moves toward the target but ends below it. The main visual differences are the amount of the response history retained, the number of peaks and dose-driven transitions visible, and small differences in late convergence behavior. No saved figure alone establishes statistical stability.

## Interpretation of Results

The systematic underestimation of $b$ means that all five runs infer a weaker treatment effect than the value used to generate the synthetic ground truth. Because the PINN total-tumor curves nevertheless fit the observations closely, the network states and the combined loss terms can compensate for some parameter mismatch. This is an important distinction between trajectory reconstruction and parameter identifiability: a visually good fit to $x_1+x_2$ does not guarantee that the latent-state decomposition or $b$ is uniquely recovered.

Several plausible mechanisms could explain why the 50- and 30-day estimates are closer to the target than the longer single-run estimates:

- Early and intermediate dose-response transients may carry strong information about $b$, while a long low-volume tail may add many points with less parameter sensitivity.
- Longer windows create more dose-delimited subnetworks and continuity constraints, increasing optimization complexity and the number of ways the neural-network states can trade off data, physics, and continuity losses.
- The fixed loss weights give each window a different effective balance because the number of segments and measurements changes.
- Random initialization and Adam optimization can lead one run to a more favorable point in a non-convex loss landscape than another.
- Only total tumor volume is observed. Different living/dead decompositions may fit that total similarly, especially when the available response regimes do not uniquely constrain the latent states.

These are hypotheses consistent with the model and saved figures, not demonstrated causes. Testing them would require repeated seeds, sensitivity analyses, and saved loss and trajectory metrics.

The 20-day window has only four dose events and does not include a long post-peak response. This narrower excitation of the system can reduce practical identifiability: fewer distinct drug-concentration and tumor-response regimes are available to separate the influence of $b$ from the network's flexible state representation. Its larger recorded error is consistent with that possibility, but one run cannot show that reduced information caused the difference.

## Limitations

- Only one run is available for each observation window. The ranking may reflect run-to-run variation rather than a repeatable effect of window length.
- Random initialization and stochastic optimization can change both the learned trajectories and the inferred parameter.
- Means, standard deviations, confidence intervals, and seed-level distributions are unavailable.
- The results have not been validated against an independent real or open-source dataset. The plotted observations are synthetic and use zero added measurement noise in the reference script.
- The comparison evaluates only the recorded $b$ estimates and available figures. It does not include numerical trajectory errors, loss histories, uncertainty estimates, or independent checks of the latent states.
- Shorter windows contain fewer measurements, fewer dose events, and fewer response regimes, which may limit parameter identifiability even when the visible total-tumor fit is good.
- The recorded 105-day value, 12.3982, conflicts with the 12.3568 displayed in its final figure and suggested by its convergence endpoint. This prevents exact traceability between that table entry and the PNG artifacts.
- The nominal 105-day figures include ground-truth and measurement traces beyond day 105, whereas the shorter-window figures are cropped at their stated endpoints. This reduces strict comparability of the baseline and should be corrected in future controlled experiments.
- Visual agreement is judged from rasterized plots and should not be treated as a substitute for numerical metrics.

## Recommended Next Steps

The following are proposed future improvements; they have not been completed in the current project:

1. Repeat every observation window with multiple controlled random seeds while keeping all other settings fixed.
2. Report the mean, standard deviation, range, and seed-level values for estimated $b$ and trajectory metrics.
3. Add RMSE, MAE, or comparable trajectory errors for total tumor volume and, where ground truth is available, the internal states.
4. Save parameter-convergence histories, final losses, and phase-level summaries in machine-readable CSV or JSON files rather than relying only on PNGs and manually recorded values.
5. Validate the approach with independent real or open-source data, using an explicit train/evaluation protocol appropriate to that dataset.
6. Manage the observation horizon, seed, output directory, and other experiment settings through a configuration file or command-line arguments so that runs are reproducible and artifacts cannot be mixed across settings.
7. Rerun the 105-day baseline with dose times, generated data, measurements, model boundaries, and evaluation grids all clipped consistently to day 105, then reconcile the recorded and figure-displayed $b$ values.
8. Examine sensitivity to loss weights and record per-component losses to determine whether parameter bias is associated with data, physics, continuity, or initial-condition trade-offs.

## Conclusion

The model underestimates the true treatment-efficacy parameter $b$ in every recorded observation window. The 50-day window has the lowest recorded error, and the 20-day window has the highest. Shortening the observation period does not produce a regular, monotonic performance decline: the 50- and 30-day single runs are closer to the target than the longer recorded runs. However, one run per condition, stochastic training, and the provenance inconsistencies in the 105-day baseline prevent a definitive robustness conclusion. Repeated, controlled experiments with numerical trajectory metrics and machine-readable convergence records are required before drawing general conclusions about the best observation-window length.
