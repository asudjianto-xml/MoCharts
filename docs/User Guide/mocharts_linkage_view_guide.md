# MoCharts User Guide: Linkage View

MoCharts supports **linked views** for multiple charts—interacting with one chart can control or update another related chart.

This guide demonstrates:
1. Linking a bar chart to a histogram using click events.
2. Linking a scatterplot to a bar chart using brush selection.

---

## Installation

```bash
pip install mocharts
```
> In Jupyter, you can also run `!pip install mocharts`.

---

## 1. Load the Iris Dataset

```python
from sklearn.datasets import load_iris
import numpy as np
import pandas as pd

data = load_iris()
target = np.array([data['target_names'][int(i)] for i in data['target']]).reshape(-1,1)
df = pd.DataFrame(
    np.concatenate([data['data'], target.astype(object)], axis=1),
    columns=data['feature_names'] + ['Species']
)

for feature in data['feature_names']:
    df[feature] = df[feature].astype(float)
```

---

## 2. Linking a Bar Chart to a Histogram (Click Event)

**Goal:** Clicking a species in the bar chart updates the histogram to show sepal length for that species.

```python
import mocharts as mc
from IPython.display import HTML

def func1(js_input, py_input):  # for use by card2
    idx = js_input['selected']
    fig2 = mc.histplot(x=df.iloc[:, 0][df.Species == idx].values, bins=8)
    fig2.chart_id = py_input['link_id']
    fig2.set_title(f"Histogram of species {idx}")
    fig2.set_tooltip(precision=4)
    fig2.set_figsize(figsize=(5, 5))
    fig2.set_yaxis(axis_name='Count')
    fig2.set_xaxis(axis_name=df.columns[0])
    return fig2.show(return_html=True, silent=True)

# Bar chart
tmp = df.Species.value_counts()
fig = mc.barplot(x=list(tmp.index.astype(str)), y=tmp.values)
fig.set_event(type_='click', func=func1, link_id='card2')    # Link to ChartID = card2
fig.set_yaxis(axis_name='Count', name_gap=20)
fig.set_xaxis(axis_name='Species')
fig.set_figsize((4, 4))
fig.set_title('Histogram of species')
html1 = fig.show(return_html=True, silent=True)

# Histogram placeholder
fig2 = mc.histplot(x=df.iloc[:,0].values, bins=8)
fig2.chart_id = 'card2'
fig2.set_title("Histogram")
fig2.set_tooltip(precision=4)
fig2.set_figsize(figsize=(4, 4))
fig2.set_yaxis(axis_name='Count')
fig2.set_xaxis(axis_name=df.columns[0])
html2 = fig2.show(return_html=True, silent=True)

# Layout side-by-side
html_content = f"""
<div style="display: flex; justify-content: space-around;">
    <div><p>{html1}</p></div>
    <div><p>{html2}</p></div>
</div>
"""

HTML(html_content)
```

**Key API points:**
- `set_event(type_='click', func=func1, link_id='card2')` → attaches an event handler that updates the chart with `chart_id='card2'`.
- The `func1` callback rebuilds and returns the updated histogram HTML.
- `link_id` in `set_event` matches the target chart’s `chart_id`.

---

## 3. Linking a Scatterplot to a Bar Chart (Brush Selection)

**Goal:** Brushing points in a scatterplot updates the bar chart to show species counts for the selected points.

```python
def func1(js_input, py_input):  # for use by card2
    selected = js_input['selected']
    if len(selected) != 0:
        link_id = py_input['link_id']
        data_selected = df.iloc[selected, list(df.columns).index('Species')]
        tmp = data_selected.value_counts()
        fig = mc.barplot(list(tmp.index), tmp.values)
        fig.link_id = link_id
        fig.set_xaxis(axis_name='Species')
        fig.set_tooltip(precision=4)
        fig.set_title('Species of selected samples')
        fig.set_figsize(figsize=(4,4))
        return fig.show(silent=True, return_html=True)

# Scatterplot with brush
sca1 = mc.scatterplot(x=df['sepal length (cm)'], y=df['sepal width (cm)'], allow_large=False)
sca1.set_xaxis(axis_name='sepal length (cm)')
sca1.set_yaxis(axis_name='sepal width (cm)')
sca1.set_event(type_='brushselected', func=func1, link_id='related_counts')
sca1.set_brush(toolbox=['rect', 'polygon', 'keep', 'clear'])
sca1.set_title('Scatter for samples selection')
sca1.set_figsize((4, 4))
html3 = sca1.show(return_html=True, silent=True)

# Bar chart placeholder
tmp = df['Species'].value_counts()
fig = mc.barplot(list(tmp.index), tmp.values)
fig.chart_id='related_counts'
fig.set_figsize(figsize=(4,4))
fig.set_xaxis(axis_name='Species')
fig.set_tooltip(precision=4)
fig.set_title('Species of selected samples')
html4 = fig.show(return_html=True, silent=True)

# Layout side-by-side
html_content2 = f"""
<div style="display: flex; justify-content: space-around;">
    <div><p>{html3}</p></div>
    <div><p>{html4}</p></div>
</div>
"""

HTML(html_content2)
```

**Key API points:**
- `set_event(type_='brushselected', func=func1, link_id='related_counts')` → links brush selection to another chart.
- `set_brush(toolbox=[...])` → configures brush tools: `'rect'`, `'polygon'`, `'keep'`, `'clear'`.
- The callback reads `selected` indices, computes counts, and returns the updated linked chart.

---

## Tips for Linkage Views

- Always match `link_id` (in `set_event`) with the target chart’s `chart_id`.
- In callbacks, use `js_input['selected']` to get the selection (string for click, list of indices for brush).
- Ensure callback returns `show(return_html=True, silent=True)` to update inline in notebooks.
- Use side-by-side HTML `<div>`s for simple linked dashboard layouts.

---

Happy linking!
