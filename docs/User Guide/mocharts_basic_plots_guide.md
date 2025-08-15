# MoCharts User Guide: Basic Plots

MoCharts provides quick, interactive plotting primitives with a consistent, chainable API (e.g., `set_xaxis`, `set_title`, `set_tooltip`, etc.). This guide shows installation, a minimal setup using the Iris dataset, and usage for seven plot types:

- Scatter
- Line
- Bar
- Box
- Histogram
- Density (KDE)
- Heatmap

_Source: Basic-Plots.ipynb in the MoCharts repo._

---

## Installation

```bash
pip install mocharts
```

---

## Quickstart (Iris dataset)

```python
from sklearn.datasets import load_iris
import numpy as np, pandas as pd

data = load_iris()
target = np.array([data['target_names'][int(i)] for i in data['target']]).reshape(-1,1)
df = pd.DataFrame(
    np.concatenate([data['data'], target.astype(object)], axis=1),
    columns=data['feature_names'] + ['Species']
)
for feature in data['feature_names']:
    df[feature] = df[feature].astype(float)
```

> **Display note (Jupyter):** MoCharts can return an embeddable HTML string. In the snippets below, `plot.show(return_html=True, silent=True)` returns HTML you can pass to `IPython.display.HTML`. If you’re running outside notebooks, use `plot.show()`.

---

## Common Pattern

All charts follow the same pattern:

1. Create the figure via `mc.scatterplot(...)` (or other constructor).  
2. Chain configuration methods (axes, legend, title, tooltip, size, etc.).  
3. Render with `show()` or return HTML.

Key setters:

- `set_xaxis(axis_name=..., axis_label_rotate=..., name_gap=...)`
- `set_yaxis(axis_name=...)`
- `set_legend(title=..., top=..., left=...)`
- `set_title('...')`
- `set_grid(right='15%')`
- `set_figsize((width, height))`
- `set_tooltip(precision=...)`
- Heatmap-specific: `set_visualmap(calculable=True, width=..., height=...)`

---

## 1) Scatter Plot

```python
import mocharts as mc
from IPython.display import HTML

plot = mc.scatterplot(
    x=df['sepal length (cm)'].values,
    y=df['sepal width (cm)'].values,
    label=df['Species'].values,
    allow_large=False,
    sampling=True
)
plot.set_xaxis(axis_name='sepal length (cm)')
plot.set_yaxis(axis_name='sepal width (cm)')
plot.set_legend(title='Species', top='13%', left='86%')
plot.set_title('Scatter Plot')
plot.set_grid(right='15%')
plot.set_figsize((6,6))

HTML(plot.show(return_html=True, silent=True))
```

---

## 2) Line Plot

```python
from IPython.display import HTML

category = pd.qcut(df['sepal width (cm)'].astype(float), 8).astype(object)
category.name = 'Sepal Width Range'
new_data = pd.concat([df, category], axis=1)
mean_val = new_data.groupby('Sepal Width Range').mean(numeric_only=True)['sepal length (cm)']

line = mc.lineplot(x=list(mean_val.index.astype(str)), y=mean_val.values)
line.set_tooltip(precision=4)
line.set_xaxis(axis_name='Sepal Width Range', axis_label_rotate=30, name_gap=50)
line.set_yaxis(axis_name='Mean sepal length (cm)')
line.set_title('Line Plot')
line.set_figsize((6,6))

HTML(line.show(return_html=True, silent=True))
```

---

## 3) Bar Plot

```python
from IPython.display import HTML

mean_length = df.groupby('Species').mean(numeric_only=True)['sepal length (cm)']

bar = mc.barplot(
    x=list(mean_length.index),
    y=np.round(mean_length.values, 4),
    show_label='top'
)
bar.set_xaxis(axis_name='Species')
bar.set_yaxis(axis_name='Mean sepal length (cm)')
bar.set_title('Bar Plot')
bar.set_tooltip(precision=4)
bar.set_figsize((6,6))

HTML(bar.show(return_html=True, silent=True))
```

---

## 4) Box Plot

```python
from IPython.display import HTML

box = mc.boxplot(
    x=df['Species'],
    y=df['sepal length (cm)'],
    orient='vertical'
)
box.set_xaxis(axis_name='Species')
box.set_yaxis(axis_name='sepal length (cm)')
box.set_title('Box Plot')
box.set_tooltip(precision=4)
box.set_figsize((6,6))

HTML(box.show(return_html=True, silent=True))
```

---

## 5) Histogram

```python
from IPython.display import HTML

hist = mc.histplot(
    x=df['sepal length (cm)'],
    label=df['Species'],
    bins=15
)
hist.set_xaxis(axis_name='sepal length (cm)')
hist.set_yaxis(axis_name='Count')
hist.set_legend()
hist.set_title('Histogram Plot')
hist.set_figsize((6,6))

HTML(hist.show(return_html=True, silent=True))
```

---

## 6) Density (KDE) Plot

```python
from IPython.display import HTML

kde = mc.kdeplot(
    x=df['sepal length (cm)'],
    label=df['Species']
)
kde.set_xaxis(axis_name='sepal length (cm)')
kde.set_yaxis(axis_name='Density')
kde.set_title('Density Plot')
kde.set_legend()
kde.set_tooltip(precision=4)
kde.set_figsize((6,6))

HTML(kde.show(return_html=True, silent=True))
```

---

## 7) Heatmap (Correlation)

```python
from IPython.display import HTML

convert = {'setosa':0, 'versicolor':1, 'virginica':2}
numeric_species = df['Species'].map(convert)
corr = pd.concat([df.drop('Species', axis=1), numeric_species], axis=1).corr().round(4)

heatmap = mc.heatmap(
    corr.values,
    row_names=list(corr.columns),
    col_names=list(corr.columns),
    show_label=True
)
heatmap.set_xaxis(axis_label_rotate=30)
heatmap.set_tooltip()
heatmap.set_title('Correlation Heatmap', left='55%')
heatmap.set_figsize((6,6))
heatmap.set_visualmap(calculable=True, width=15, height=520)

HTML(heatmap.show(return_html=True, silent=True))
```

---

## Tips & Conventions

- **Figure sizing:** `set_figsize((width, height))` in inches.
- **Tooltips:** `set_tooltip(precision=...)` controls numeric formatting in hover text.
- **Legends:** `set_legend(...)` to position and title.
- **Layout nudges:** `set_grid(right='15%')` prevents overlaps.
- **Sampling:** For large scatter datasets, keep `allow_large=False` and `sampling=True`.

---

## Troubleshooting

- **No output in notebooks:** Wrap `plot.show(return_html=True, silent=True)` in `IPython.display.HTML(...)`.
- **Overlapping labels/legend:** Rotate x-labels, adjust grid, or enlarge figure.
- **Heatmap cramped:** Increase `set_visualmap(..., height=...)` and rotate x-labels.
