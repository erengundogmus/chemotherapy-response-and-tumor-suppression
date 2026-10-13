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
- Notes about whether the run was stable# Observation Window Experiment Plan

## Goal

Test how the model behaves when the available observation period is shortened.

## Observation Windows

- 105 days
- 100 days
- 50 days
- 30 days
- 20 days

## Current Evidence

The article defines the maximum study horizon as 105 days. In the reference code, this value appears in the time span, segment boundaries, and evaluation grid.

Relevant code patterns:

```python
t_span = [0, 105]
boundaries = sorted(list(set([0.0] + dose_times + [105.0])))
t_eval = np.linspace(0, 105, 500)
```

For the experiment, these fixed values were parameterized with `STUDY_END_DAY` so that the same script could be tested with shorter observation windows.

## Output to Record

For each observation window, we record:

- Predicted `b` parameter
- Tumor volume prediction figures
- Parameter convergence figure
- Notes about whether the run was stable

## Completed Runs

| Observation Window | Status    |
| 105 days           | Completed |
| 100 days           | Completed |
| 50 days            | Completed |
| 30 days            | Completed |
| 20 days            | Completed |