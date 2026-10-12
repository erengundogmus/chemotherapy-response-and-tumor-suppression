# Observation Window Experiment Plan

## Goal

Test how the model behaves when the available observation period is shortened.

## Planned Observation Windows

- 105 days
- 100 days
- 50 days
- 30 days
- 20 days

## Current Evidence

The article defines the maximum study horizon as 105 days. In the reference code, this appears in the time span, segment boundaries, and evaluation grid.

Relevant code patterns to check:

- `t_span = [0, 105]`
- `boundaries = sorted(list(set([0.0] + dose_times + [105.0])))`
- `t_eval = np.linspace(0, 105, 500)`

## Output to Record

For each observation window, we plan to record:

- Estimated parameters
- Tumor volume prediction curve
- Error values if available
- Notes about whether the run was stable