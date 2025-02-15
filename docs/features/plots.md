<script setup>

import * as Plot from "@observablehq/plot";
import * as d3 from "d3";
import * as htl from "htl";
import {computed, ref, shallowRef, onMounted} from "vue";
import alphabet from "../data/alphabet.ts";

const marginTop = ref(20);
const marginRight = ref(20);
const marginBottom = ref(30);
const marginLeft = ref(40);
const fixed = ref(true);
const aapl = shallowRef([]);
const goog = shallowRef([]);
const penguins = shallowRef([]);
const stocks = computed(() => [...aapl.value.map((d) => ({...d, Symbol: "AAPL"})), ...goog.value.map((d) => ({...d, Symbol: "GOOG"}))]);

onMounted(() => {
  d3.csv("../data/aapl.csv", d3.autoType).then((data) => (aapl.value = data));
  d3.csv("../data/goog.csv", d3.autoType).then((data) => (goog.value = data));
  d3.csv("../data/penguins.csv", d3.autoType).then((data) => (penguins.value = data));
});

</script>

# Plots

To render a **plot** in Observable Plot, call [plot](#plot) (typically as `Plot.plot`), passing in the desired *options*. This function returns an SVG or HTML figure element.

:::plot https://observablehq.com/@observablehq/plot-hello-world
```js
Plot.plot({
  marks: [
    Plot.frame(),
    Plot.text(["Hello, world!"], {frameAnchor: "middle"})
  ]
})
```
:::

:::tip
The returned plot element is detached; it must be inserted into the page to be visible. For help, see the [getting started guide](../getting-started.md).
:::

## Marks option

The **marks** option specifies an array of [marks](./marks.md) to render. Above, there are two marks: a [frame](../marks/frame.md) to draw the outline of the plot frame, and a [text](../marks/text.md) to say hello. 👋

Each mark supplies its own tabular data. For example, the table below shows the first five rows of a daily dataset of Apple stock price (`aapl`).

| Date       | Open      | High      | Low       | Close     | Volume    |
| ---------- | --------: | --------: | --------: | --------: | --------: |
| 2013-05-13 | 64.501427 | 65.414284 | 64.500000 | 64.962860 |  79237200 |
| 2013-05-14 | 64.835716 | 65.028572 | 63.164288 | 63.408573 | 111779500 |
| 2013-05-15 | 62.737144 | 63.000000 | 60.337143 | 61.264286 | 185403400 |
| 2013-05-16 | 60.462856 | 62.549999 | 59.842857 | 62.082859 | 150801000 |
| 2013-05-17 | 62.721428 | 62.869999 | 61.572857 | 61.894287 | 106976100 |

In JavaScript, we can represent tabular data as an array of objects. Each object records a daily observation, with properties *Date*, *Open*, *High*, and so on. This is known as a “row-based” format since each object corresponds to a row in the table.

```js-vue
aapl = [
  {Date: new Date("2013-05-13"), Open: 64.501427, High: 65.414284, Low: 64.500000, Close: 64.962860, Volume: 79237200},
  {Date: new Date("2013-05-14"), Open: 64.835716, High: 65.028572, Low: 63.164288, Close: 63.408573, Volume: 111779500},
  {Date: new Date("2013-05-15"), Open: 62.737144, High: 63.000000, Low: 60.337143, Close: 61.264286, Volume: 185403400},
  {Date: new Date("2013-05-16"), Open: 60.462856, High: 62.549999, Low: 59.842857, Close: 62.082859, Volume: 150801000},
  {Date: new Date("2013-05-17"), Open: 62.721428, High: 62.869999, Low: 61.572857, Close: 61.894287, Volume: 106976100}
]
```

:::tip
Rather than baking data into JavaScript, use [JSON](https://en.wikipedia.org/wiki/JSON) or [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) files to store data. You can use [d3.json](https://d3js.org/d3-fetch#json), [d3.csv](https://d3js.org/d3-fetch#csv), or [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) to load a file. On Observable, you can also use a [file attachment](https://observablehq.com/@observablehq/file-attachments) or [SQL cell](https://observablehq.com/@observablehq/sql-cell).
:::

To use data with Plot, pass the data as the first argument to the mark constructor. We can then assign columns of data such as *Date* and *Close* to visual properties of the mark (or “channels”) such as horizontal↔︎ position **x** and vertical↕︎ position **y**.

:::plot defer https://observablehq.com/@observablehq/plot-first-line-chart
```js
Plot.plot({
  marks: [
    Plot.lineY(aapl, {x: "Date", y: "Close"})
  ]
})
```
:::

A plot can have multiple marks, and each mark has its own data. For example, say we had a similar table `goog` representing the daily price of Google stock for the same period. Below, the <span style="border-bottom: solid 2px var(--vp-c-red);">red</span> line represents Google stock, while the <span style="border-bottom: solid 2px var(--vp-c-blue);">blue</span> line represents Apple stock.

:::plot defer https://observablehq.com/@observablehq/plot-layered-marks
```js
Plot.plot({
  marks: [
    Plot.ruleY([0]),
    Plot.lineY(goog, {x: "Date", y: "Close", stroke: "red"}),
    Plot.lineY(aapl, {x: "Date", y: "Close", stroke: "blue"})
  ]
})
```
:::

:::tip
When comparing the performance of different stocks, we typically want to normalize the return relative to a purchase price. See the [normalize transform](../transforms/normalize.md) for an example.
:::

Alternatively, the tables can be combined, say with a *Symbol* column to distinguish AAPL from GOOG. This allows the use of a categorical *color* scale and legend.

:::plot defer https://observablehq.com/@observablehq/plot-stocks-multiline-chart
```js
Plot.plot({
  color: {legend: true},
  marks: [
    Plot.ruleY([0]),
    Plot.lineY(stocks, {x: "Date", y: "Close", stroke: "Symbol"})
  ]
})
```
:::

Each mark has its own options, and different mark types support different options. See the respective mark type (such as [bar](../marks/bar.md) or [dot](../marks/dot.md)) for details.

Marks are drawn in the given order, with the last mark drawn on top. For example, below <span style="border-bottom: solid 2px var(--vp-c-green);">green</span> bars are drawn on top of <span style="border-bottom: solid 2px;">{{$dark ? "white" : "black"}}</span> bars.

:::plot https://observablehq.com/@observablehq/plot-marks-z-order
```js
Plot.plot({
  x: {padding: 0.4},
  marks: [
    Plot.barY(alphabet, {x: "letter", y: "frequency", dx: 2, dy: 2}),
    Plot.barY(alphabet, {x: "letter", y: "frequency", fill: "green", dx: -2, dy: -2})
  ]
})
```
:::

## Layout options

The layout options determine the overall size of the plot; all are specified as numbers in pixels:

* **marginTop** - the top margin
* **marginRight** - the right margin
* **marginBottom** - the bottom margin
* **marginLeft** - the left margin
* **margin** - shorthand for the four margins
* **width** - the outer width of the plot (including margins)
* **height** - the outer height of the plot (including margins)

Experiment with the margins by adjusting the sliders below. Note that because the *x* scale is a *band* scale, the **round** option defaults to true, so the bars may jump when you adjust the horizontal margins to snap to crisp edges.

<p>
  <label class="label-input" style="display: flex;">
    <span style="display: inline-block; width: 7em;">marginTop:</span>
    <input type="range" v-model.number="marginTop" min="0" max="60" step="1">
    <span style="font-variant-numeric: tabular-nums;">{{marginTop}}</span>
  </label>
  <label class="label-input" style="display: flex;">
    <span style="display: inline-block; width: 7em;">marginRight:</span>
    <input type="range" v-model.number="marginRight" min="0" max="60" step="1">
    <span style="font-variant-numeric: tabular-nums;">{{marginRight}}</span>
  </label>
  <label class="label-input" style="display: flex;">
    <span style="display: inline-block; width: 7em;">marginBottom:</span>
    <input type="range" v-model.number="marginBottom" min="0" max="60" step="1">
    <span style="font-variant-numeric: tabular-nums;">{{marginBottom}}</span>
  </label>
  <label class="label-input" style="display: flex;">
    <span style="display: inline-block; width: 7em;">marginLeft:</span>
    <input type="range" v-model.number="marginLeft" min="0" max="60" step="1">
    <span style="font-variant-numeric: tabular-nums;">{{marginLeft}}</span>
  </label>
</p>

:::plot hidden defer
```js
Plot.plot({
  marginTop,
  marginRight,
  marginBottom,
  marginLeft,
  grid: true,
  marks: [
    Plot.frame({
      stroke: "var(--vp-c-text-2)",
      strokeOpacity: 0.5,
      insetTop: -marginTop,
      insetRight: -marginRight,
      insetBottom: -marginBottom,
      insetLeft: -marginLeft,
    }),
    Plot.barY(alphabet, {x: "letter", y: "frequency", fill: "green"}),
    Plot.frame()
  ]
})
```
:::

```js-vue
Plot.plot({
  marginTop: {{marginTop}},
  marginRight: {{marginRight}},
  marginBottom: {{marginBottom}},
  marginLeft: {{marginLeft}},
  grid: true,
  marks: [
    Plot.barY(alphabet, {x: "letter", y: "frequency", fill: "green"}),
    Plot.frame()
  ]
})
```

:::info
To assist the explanation, the plot above is drawn with a light gray border.
:::

The default **width** is 640. On Observable, the width can be set to the [standard width](https://github.com/observablehq/stdlib/blob/main/README.md#width) to make responsive plots. The default **height** is chosen automatically based on the plot’s associated scales; for example, if *y* is linear and there is no *fy* scale, it might be 396. The default margins depend on the maximum margins of the plot’s constituent [marks](#marks-option). While most marks default to zero margins (because they are drawn inside the chart area), Plot’s [axis mark](../marks/axis.md) has non-zero default margins.

:::tip
Plot does not adjust margins automatically to make room for long tick labels. If your *y* axis labels are too long, you can increase the **marginLeft** to make more room. Also consider using a different **tickFormat** for short labels (*e.g.*, `s` for SI prefix notation), or a scale **transform** (say to convert units to millions or billions).
:::

The **aspectRatio** option<a id="aspectRatio" class="header-anchor" href="#aspectRatio" aria-label="Permalink to &quot;aspectRatio&quot;"></a> <VersionBadge version="0.6.4" />, if not null, computes a default **height** such that a variation of one unit in the *x* dimension is represented by the corresponding number of pixels as a variation in the *y* dimension of one unit.

<p>
  <label class="label-input">
    Use fixed aspect ratio:
    <input type="checkbox" v-model="fixed">
  </label>
</p>

:::plot defer https://observablehq.com/@observablehq/plot-intro-to-aspectratio
```js
Plot.plot({
  grid: true,
  inset: 10,
  aspectRatio: fixed ? 1 : undefined,
  color: {legend: true},
  marks: [
    Plot.frame(),
    Plot.dot(penguins, {x: "culmen_length_mm", y: "culmen_depth_mm", stroke: "species"})
  ]
})
```
:::

:::tip
When using facets, set the *fx* and *fy* scales’ **round** option to false if you need an exact aspect ratio.
:::

## Other options

By default, [plot](#plot) returns an SVG element; however, if the plot includes a title, subtitle, [legend](./legends.md), or caption, plot wraps the SVG element with an HTML figure element. You can also force Plot to generate a figure element by setting the **figure** option <VersionBadge version="0.6.10" pr="1761" /> to true.

The **title** & **subtitle** options <VersionBadge version="0.6.10" pr="1761" /> and the **caption** option accept either a string or an HTML element. If given an HTML element, say using the [`html` tagged template literal](http://github.com/observablehq/htl), the title and subtitle are used as-is while the caption is wrapped in a figcaption element; otherwise, the specified text will be escaped and wrapped in an h2, h3, or figcaption, respectively.

:::plot https://observablehq.com/@observablehq/plot-caption
```js
Plot.plot({
  title: "For charts, an informative title",
  subtitle: "Subtitle to follow with additional context",
  caption: "Figure 1. A chart with a title, subtitle, and caption.",
  marks: [
    Plot.frame(),
    Plot.text(["Titles, subtitles, captions, and annotations assist inter­pretation by telling the reader what’s interesting. Don’t make the reader work to find what you already know."], {lineWidth: 30, frameAnchor: "middle"})
  ]
})
```
:::

The **style** option allows custom styles to override Plot’s defaults. It may be specified either as a string of inline styles (*e.g.*, `"color: red;"`, in the same fashion as assigning [*element*.style](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style)) or an object of properties (*e.g.*, `{color: "red"}`, in the same fashion as assigning [*element*.style properties](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration)). By default, the returned plot has a white background, a max-width of 100%, and the system-ui font. Plot’s marks and axes default to [currentColor](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value#currentcolor_keyword), meaning that they will inherit the surrounding content’s color.

:::warning CAUTION
Unitless numbers ([quirky lengths](https://www.w3.org/TR/css-values-4/#deprecated-quirky-length)) such as `{padding: 20}` are not supported by some browsers; you should instead specify a string with units such as `{padding: "20px"}`.
:::

The generated SVG element has a class name which applies a default stylesheet. Use the top-level **className** option to specify that class name.

The **clip** option <VersionBadge version="0.6.10" pr="1792" /> determines the default clipping behavior if the [mark **clip** option](./marks.md#mark-options) is not specified; set it to true to enable clipping. This option does not affect [axis](../marks/axis.md), [grid](../marks/grid.md), and [frame](../marks/frame.md) marks, whose **clip** option defaults to false.

The **document** option specifies the [document](https://developer.mozilla.org/en-US/docs/Web/API/Document) used to create plot elements. It defaults to window.document, but can be changed to another document, say when using a virtual DOM implementation for server-side rendering in Node.

## plot(*options*) {#plot}

```js
Plot.plot({
  height: 200,
  marks: [
    Plot.barY(alphabet, {x: "letter", y: "frequency"})
  ]
})
```

Renders a new plot with the specified *options*, returning a SVG or HTML figure element. This element can then be inserted into the page as described in the [getting started guide](../getting-started.md).

## *mark*.plot(*options*) {#mark_plot}

```js
Plot.barY(alphabet, {x: "letter", y: "frequency"}).plot({height: 200})
```

Given a [*mark*](./marks.md), this is a convenience shorthand for calling [plot](#plot) where the **marks** option includes this *mark*. Any additional **marks** in *options* are drawn on top of this *mark*.

## *plot*.scale(*name*) {#plot_scale}

```js
const plot = Plot.plot(options); // render a plot
const color = plot.scale("color"); // get the color scale
console.log(color.range); // inspect the scale’s range
```

Returns the [scale object](./scales.md#scale-options) for the scale with the specified *name* (such as *x* or *color*) on the given *plot*, where *plot* is a rendered plot element returned by [plot](#plot). If the associated *plot* has no scale with the given *name*, returns undefined.

## *plot*.legend(*name*, *options*) {#plot_legend}

```js
const plot = Plot.plot(options); // render a plot
const legend = plot.legend("color"); // render a color legend
```

Renders a standalone legend for the scale with the specified *name* (such as *x* or *color*) on the given *plot*, where *plot* is a rendered plot element returned by [plot](#plot), returning a SVG or HTML figure element. This element can then be inserted into the page as described in the [getting started guide](../getting-started.md). If the associated *plot* has no scale with the given *name*, returns undefined. Legends are currently only supported for *color*, *opacity*, and *symbol* scales.
