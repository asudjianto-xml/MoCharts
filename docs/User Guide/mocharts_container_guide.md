# MoCharts User Guide: Containers, Figures, and SubPlots

MoCharts supports flexible **layout options** for displaying one or many plots. This guide shows how to:

- Render a **single plot** in a figure
- **Combine multiple plots** into one figure
- Arrange plots with a **SubPlots** container (custom grid/positions)

The examples below use the exact code you provided, plus brief explanations and tips.

---

## Installation

```bash
pip install mocharts
```

> In notebooks you can also run: `!pip install mocharts`

---

## Figure (Single Plot)

Create a `Figure`, then add a single chart (e.g., scatter plot). In notebooks, use `IPython.display.HTML` with `return_html=True` for clean inline output.

```python
from IPython.display import HTML
import mocharts as mc
from sklearn.datasets import make_circles

pos, label = make_circles(n_samples=100, shuffle=True, noise=None, random_state=None, factor=0.8)
x, y = pos[:, 0], pos[:, 1]

fig = mc.Figure()
plot = mc.scatterplot(x=x, y=y, label=label)
fig.add(plot)
# fig.show()
HTML(fig.show(return_html=True, silent=True))
```

**Notes**
- `mc.Figure()` is a container for one or more plots.
- `.add(plot)` attaches a chart to the figure.
- `.show(return_html=True, silent=True)` returns embeddable HTML without printing extra logs.

---

## Figure (Multiple Plots)

You can layer multiple charts onto one figure—e.g., plot points **and** overlay a line. Title the combined figure with `set_title`.

```python
fig = mc.Figure()

scatter = mc.scatterplot(x=x, y=y)
fig.add(scatter)

line = mc.lineplot([-1, 1], [-1, 1])
fig.add(line)
fig.set_title('Combine Plot')

# fig.show()
HTML(fig.show(return_html=True, silent=True))
```

**Tips**
- Order matters: later `add(...)` calls render on top of earlier ones.
- Use chart-level methods (e.g., `set_title`, `set_xaxis`) or figure-level methods (`set_title`) to control appearance.
- For legends, tooltips, and sizes, configure each plot (e.g., `scatter.set_legend(...)`, `line.set_figsize(...)`).

---

## SubPlots

`SubPlots` lets you place multiple charts within the same canvas based on a grid with explicit sub-rectangle positions.

```python
scatter = mc.scatterplot(x=x, y=y)
line = mc.lineplot([-1, 1], [-1, 1])
line.set_title('Line')
scatter.set_title('Scatter')
sub = mc.SubPlots(
    [scatter, line],
    subplots_grid=[10, 1],
    subplots_position=[[[0, 5], [0, 1]], [[6, 10], [0, 1]]]
)

# sub.show()
HTML(sub.show(return_html=True, silent=True))
```

### How the grid and positions work

- **`subplots_grid=[R, C]`** defines a coarse grid with **R rows** and **C columns** (here `10 × 1`).
- **`subplots_position`** is a list with one entry per subplot. Each entry has two spans: `[[row_start, row_end], [col_start, col_end]]`.
  - Indices are **grid boundaries** (not counts). The subplot spans rows `[row_start, row_end)` and columns `[col_start, col_end)` using half-open intervals.
  - In the example:
    - The first subplot (`scatter`) occupies rows `0–5` and cols `0–1` (top block).
    - The second subplot (`line`) occupies rows `6–10` and cols `0–1` (bottom block).

**Layout tips**
- Use finer `subplots_grid` (e.g., `[24, 24]`) for more precise placement.
- Keep some spacing between row/column boundaries to avoid axis label overlap.
- Each child plot can still use its own setters (`set_title`, `set_xaxis`, etc.).

---

## Rendering Notes

- **Jupyter:** Prefer `HTML(obj.show(return_html=True, silent=True))` for clean inline output.
- **Scripts:** Call `obj.show()` to render (exact behavior depends on your environment; you can also save HTML if the API supports it).

---

## Troubleshooting

- **No output in notebook:** Ensure you either print the returned HTML or wrap with `IPython.display.HTML(...)` when using `return_html=True`.
- **Overlapping labels or legends:** Increase figure size on individual charts, reduce font sizes, or adjust subplot spans so axes have more room.
- **Subplot not visible:** Check the `subplots_position` ranges; ensure each child occupies a non-empty region within the `subplots_grid`.

---

## Copy-Paste Summary

**Single plot**
```python
fig = mc.Figure(); fig.add(mc.scatterplot(x=x, y=y)); HTML(fig.show(return_html=True, silent=True))
```

**Multiple plots in one figure**
```python
fig = mc.Figure()
fig.add(mc.scatterplot(x=x, y=y))
fig.add(mc.lineplot([-1,1], [-1,1]))
fig.set_title('Combine Plot')
HTML(fig.show(return_html=True, silent=True))
```

**SubPlots with custom grid**
```python
p1 = mc.scatterplot(x=x, y=y).set_title('Scatter')
p2 = mc.lineplot([-1,1], [-1,1]).set_title('Line')
sub = mc.SubPlots([p1, p2], subplots_grid=[10,1], subplots_position=[[[0,5],[0,1]], [[6,10],[0,1]]])
HTML(sub.show(return_html=True, silent=True))
```

---

Happy charting!
