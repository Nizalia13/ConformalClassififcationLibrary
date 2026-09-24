# Conformal Envelopes

A Python library for classification using a seperate conformal envelope per class.

Available envelope methods:
- **Collapsed:** averages observed nonconformity scores.
- **Radial:** learns a direction-dependent boundary.
- **Strip:** learns constraints between pairs of observed score dimensions.

## Input data

Training data can be a pandas DataFrame or a CSV file:

| ID | predictor_1 | predictor_2 | label |
|---|---|---|---|
| sample_1 | 0.85 | 0.72 | A |
| sample_2 | 0.31 | NaN | B |

The library takes predictor scores as input; it does not train the predictors.

Missing scores can be represented by NaN. Handling depends on the method and on which scores are available. 
Entirely missing samples require particular care.

By default, higher scores mean better agreement with a class, and scores must lie between 0 and 1. The library 
converts these to nonconformity scores using 1 - score.

Use score_direction="lower_is_better" for nonnegative scores where smaller values already mean better agreement.

## Installation
From the project directory:

```bash
python -m pip install -e .
```

To include testing and notebook example dependencies:

```bash
python -m pip install -e ".[test,examples]"
```

## Fit and predict

```python
from conformal_envelopes import ConformalSetModel

model = ConformalSetModel(method="strip", alpha=0.10, random_state=13, force_nonempty=False,
                          n_bins=8, min_samples=3,)

model.fit(training, id_col="ID", label_col="label", score_cols=["predictor_1", "predictor_2"],)

predictions = model.predict(test_data)
envelopes = model.get_envelopes()
```

Here, training and test_data are DataFrames or CSV paths. Test labels are not needed for prediction.

Each prediction set contains the classes whose envelopes accept the sample. Sets can contain multiple 
classes or be empty.

For a dictionary mapping sample IDs to prediction sets:

```python
prediction_sets = model.predict(test_data, output="dict")
```

## Class-specific predictor columns

By default, all classes use the same predictor columns.

When predictors provide separate scores for each candidate class, supply label_to_columns to fit:

```python
model.fit(training, label_to_columns={"A": ["A_predictor_1", "A_predictor_2"],
                                      "B": ["B_predictor_1", "B_predictor_2"],},)
```

## Method parameters

- Collapsed: no additional geometry parameters.
- Radial: n_directions, smoothing, angle_deg, neighbor_fraction.
- Strip: n_bins, min_samples.

Common parameters include alpha, shape_fraction, random_state,
score_direction, and force_nonempty.

## Visualisation

There are three plotting options:

- `model.plot(...)`: fits a separate two-dimensional envelope using
  the two selected predictors. Test-point categories describe decisions
  from this separate 2D envelope.

- `model.plot_slice(...)`: displays a cross-section of the existing
  fitted envelope without refitting. Hidden coordinates default to zero.
  Use `slice_values` to specify their fixed nonconformity values.
  The plot displays these values above the axes.

- `plot_inclusion_heatmap(...)`: shows how frequently each candidate
  class appears in prediction sets, grouped by the true class.
  Diagonal entries give per-class coverage. Rows need not sum to one
  because prediction sets can contain multiple labels.

Envelope plots use nonconformity coordinates. With
`score_direction="higher_is_better"`, these equal `1 - score`.

In slice plots, test-point categories use each sample's actual
full-dimensional scores. Their acceptance may therefore differ from
what their displayed position relative to the slice suggests.

Use `test_data` to show test samples, `show_training` to display
training samples, and `show_missing` to display dotted lines for
samples missing one plotted coordinate.

Pass `ax` to arrange plots in separate figures or a shared figure.


## Tests

```bash
python -m pytest test_library.py -v
```

The tests check software behaviour, including prediction formats, input validation, saving/loading, and whether 
plotting preserves predictions.

## Coverage and limitations

alpha=0.10 specifies a nominal coverage target of 90%.

Conformal coverage relies on assumptions such as exchangeability between calibration and future samples, with the score 
construction fixed independently of calibration data. It does not guarantee exactly 90% coverage on every observed test set.

Distribution changes, candidate filtering, and missing-score handling need to be considered when interpreting results.

Small class-specific calibration sets can produce infinite thresholds and consequently broad prediction sets.

For strip envelopes, zero or very small shape limits can produce large calibration scaling factors, weakening other constraints.