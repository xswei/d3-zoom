# d3-zoom

视图移动以及缩放是一种将用户注意力聚焦在感兴趣区域的一种流行的交互技术。操作直接，容易理解: 点击并拖拽平移，使用滚轮进行缩放，当然也可以通过触摸进行。平移和缩放被广泛的应用在地图中，但是也可被应用到其他的可视化比如时间序列以及散点图中。

缩放行为通过 `d3-zoom` 模块实现，能方便且灵活到 [selections](https://github.com/d3/d3-selection) 上。它处理了许多 [input events](#api-reference ) 以及浏览器的怪异模式。缩放行为与 `DOM` 无关，可以应用于 `SVG`, `HTML` 或者 `Canvas`。

[<img alt="Canvas Zooming" src="https://raw.githubusercontent.com/d3/d3-zoom/master/img/dots.png" width="420" height="219">](https://bl.ocks.org/mbostock/d1f7b58631e71fbf9c568345ee04a60e)[<img alt="SVG Zooming" src="https://raw.githubusercontent.com/d3/d3-zoom/master/img/dots.png" width="420" height="219">](https://bl.ocks.org/mbostock/4e3925cdc804db257a86fdef3a032a45)

缩放行为可也以与 [d3-scale](https://github.com/xswei/d3-scale) 和 [d3-axis](https://github.com/xswei/d3-axis) 一起工作; 参考 [*transform*.rescaleX](#transform_rescaleX) and [*transform*.rescaleY](#transform_rescaleY). 你可以使用 [*zoom*.scaleExtent](#zoom_scaleExtent) 显示平移范围或使用 [*zoom*.translateExtent](#zoom_translateExtent) 来显示缩放尺度。

[<img alt="Axis Zooming" src="https://raw.githubusercontent.com/d3/d3-zoom/master/img/axis.png" width="420" height="219">](https://bl.ocks.org/mbostock/db6b4335bf1662b413e7968910104f0f)

缩放行为可以用来和其他的交互结合，比如用以拖拽的 [d3-drag](https://github.com/xswei/d3-drag) 和用以刷取的 [d3-brush](https://github.com/xswei/d3-brush)。

[<img alt="Drag & Zoom II" src="https://raw.githubusercontent.com/d3/d3-zoom/master/img/dots.png" width="420" height="219">](https://bl.ocks.org/mbostock/3127661b6f13f9316be745e77fdfb084)[<img alt="Brush & Zoom" src="https://raw.githubusercontent.com/d3/d3-zoom/master/img/brush.png" width="420" height="219">](https://bl.ocks.org/mbostock/34f08d5e11952a80609169b7917d4172)

缩放行为可以通使用 [*zoom*.transform](#zoom_transform) 以编程的方式控制，允许使用用户操作面板以及根据数据来驱动显示状态的变化。平滑缩放过渡基于 `Jarke J. van Wijk` 和 `Wim A.A. Nuij` 的 [“Smooth and efficient zooming and panning”](http://www.win.tue.nl/~vanwijk/zoompan.pdf)。

[<img alt="Zoom Transitions" src="https://raw.githubusercontent.com/d3/d3-zoom/master/img/transition.png" width="420" height="219">](https://bl.ocks.org/mbostock/b783fbb2e673561d214e09c7fb5cedee)

参考 [d3-tile](https://github.com/d3/d3-tile) 获取关于平移和缩放地图的例子.

## Installing

`NPM` 安装: `npm install d3-zoom`. 此外还可以下载 [latest release](https://github.com/d3/d3-zoom/releases/latest). 可以直接从 [d3js.org](https://d3js.org), 以 [standalone library](https://d3js.org/d3-zoom.v1.min.js) 或作为 [D3 4.0](https://github.com/d3/d3) 的一部分引入. 支持 `AMD`, `CommonJS` 以及基本的标签引入形式，如果使用标签引入则会暴露全局 `d3` 变量:

```html
<script src="https://d3js.org/d3-color.v1.min.js"></script>
<script src="https://d3js.org/d3-dispatch.v1.min.js"></script>
<script src="https://d3js.org/d3-ease.v1.min.js"></script>
<script src="https://d3js.org/d3-interpolate.v1.min.js"></script>
<script src="https://d3js.org/d3-selection.v1.min.js"></script>
<script src="https://d3js.org/d3-timer.v1.min.js"></script>
<script src="https://d3js.org/d3-transition.v1.min.js"></script>
<script src="https://d3js.org/d3-drag.v1.min.js"></script>
<script src="https://d3js.org/d3-zoom.v1.min.js"></script>
<script>

var zoom = d3.zoom();

</script>
```

[在浏览器中测试 `d3-zoom`.](https://tonicdev.com/npm/d3-zoom)

## API Reference

下面这个表描述了缩放行为如何利用原生事件:

| Event        | Listening Element | Zoom Event  | Default Prevented? |
| ------------ | ----------------- | ----------- | ------------------ |
| mousedown⁵   | selection         | start       | no¹                |
| mousemove²   | window¹           | zoom        | yes                |
| mouseup²     | window¹           | end         | yes                |
| dragstart²   | window            | -           | yes                |
| selectstart² | window            | -           | yes                |
| click³       | window            | -           | yes                |
| dblclick     | selection         | *multiple*⁶ | yes                |
| wheel⁸       | selection         | zoom⁷       | yes                |
| touchstart   | selection         | *multiple*⁶ | no⁴                |
| touchmove    | selection         | zoom        | yes                |
| touchend     | selection         | end         | no⁴                |
| touchcancel  | selection         | end         | no⁴                |

所有事件的传播会被 [immediately stopped](https://dom.spec.whatwg.org/#dom-event-stopimmediatepropagation)

¹ 在 `iframe` 之外捕获事件的必要条件; 参考 [d3-drag#9](https://github.com/d3/d3-drag/issues/9).
<br>² 只适用于以鼠标为基础的活动; 参考 [d3-drag#9](https://github.com/d3/d3-drag/issues/9).
<br>³ 只在一些基于鼠标的手势之后立即应用; 参考 [*zoom*.clickDistance](#zoom_clickDistance).
<br>⁴ 在触摸输入上允许 [click emulation(点击模拟)](https://developer.apple.com/library/ios/documentation/AppleApplications/Reference/SafariWebContent/HandlingEvents/HandlingEvents.html#//apple_ref/doc/uid/TP40006511-SW7) 的必要条件; 参考 [d3-drag#9](https://github.com/d3/d3-drag/issues/9).
<br>⁵ 忽略触摸事件结束后的 `500ms` 之内的事件; 假设 [click emulation(点击模拟)](https://developer.apple.com/library/ios/documentation/AppleApplications/Reference/SafariWebContent/HandlingEvents/HandlingEvents.html#//apple_ref/doc/uid/TP40006511-SW7).
<br>⁶ 双击和双击启动一个转换，它会发出开始、缩放和结束事件。
<br>⁷ 第一次滚轮事件发出一个启动事件; 当在 `150ms` 之内没有再次收到滚轮事件时，就会发出一个结束事件.
<br>⁸ 如果已经达到了 [scale extent](#zoom_scaleExtent) 的限制，则忽略.

<a href="#zoom" name="zoom">#</a> d3.<b>zoom</b>() [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js "Source")

创建一个新的缩放行为，并返回该行为。[*zoom*](#_drag) 既是一个对象又是一个函数，通过情况下通过 [*selection*.call](https://github.com/d3/d3-selection#selection_call) 来应用到元素上。

<a href="#_zoom" name="_zoom">#</a> <i>zoom</i>(<i>selection</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L62 "Source")

将缩放行为应用到指定的 [*selection*](https://github.com/d3/d3-selection)，并绑定必要的事件监听器用来允许平移和缩放，如果元素上没有定义 [zoom transform](#zoom-transforms) 的话会将其初始化为 `identity transform`。这个函数通常不会被直接调用，而是通过 [*selection*.call](https://github.com/d3/d3-selection#selection_call) 来调用。例如，初始化一个缩放行为并将其应用到指定的选择集:

```js
selection.call(d3.zoom().on("zoom", zoomed));
```

在内部，缩放行为使用 [*selection*.on](https://github.com/d3/d3-selection#selection_on) 去绑定缩放必需的一些事件句柄。事件句柄会以 `.zoom` 为 `name`，因此你可以使用如下方式来取消元素上所有的缩放行为的事件句柄:

```js
selection.on(".zoom", null);
```

如果仅仅想将滚动滚轮事件取消(不干扰原生滚轮事件)，则可以单独将缩放的滚轮事件取消:

```js
selection
    .call(zoom)
    .on("wheel.zoom", null);
```

此外还可以使用 [*zoom*.filter](#zoom_filter) 来控制哪些事件可以开始缩放手势.

应用缩放行为也会将 [-webkit-tap-highlight-color](https://developer.apple.com/library/mac/documentation/AppleApplications/Reference/SafariWebContent/AdjustingtheTextSize/AdjustingtheTextSize.html#//apple_ref/doc/uid/TP40006510-SW5) 设置为透明，禁用 `iOS` 上的高亮显示功能. 如果你需要一个不一样的高亮颜色则可以在应用缩放行为之后重新设置这个样式。

<a href="#zoom_transform" name="zoom_transform">#</a> <i>zoom</i>.<b>transform</b>(<i>selection</i>, <i>transform</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L76 "Source")

如果 *selection* 为选择集，则设置已选中元素的 [current zoom transform](#zoomTransform) 为指定的 *transform*, 并立即触发 `start`, `zoom` 以及 `end` [events](#zoom-events)。如果 *selection* 为过渡则使用 [d3.interpolateZoom](https://github.com/xswei/d3-interpolate#interpolateZoom) 定义一个 “zoom” 补间并在过渡开始时触发 `start` 事件，过渡中的每一帧都触发一次 `zoom` 事件，结束或中断时触发 `end` 事件。*transform* 可以是 [zoom transform(缩放变换)](#zoom-transforms) 也可以是返回缩放变换的函数。如果是函数则会为选择集中的每一个元素调用，并传递当前元素数据 `d` 以及索引 `i`, 函数内部 `this` 指向当前 `DOM` 元素。

这个函数通常不会直接调用，而是通过 [*selection*.call](https://github.com/d3/d3-selection#selection_call) 或 [*transition*.call](https://github.com/d3/d3-transition#transition_call) 调用的。比如重置缩放变换为 [identity transform](#zoomIdentity):

```js
selection.call(zoom.transform, d3.zoomIdentity);
```

或者平滑的重置缩放(`750ms`)：

```js
selection.transition().duration(750).call(zoom.transform, d3.zoomIdentity);
```

这个方法要求指定新的缩放变换，不会受 [scale extent](#zoom_scaleExtent) 和 [translate extent](#zoom_translateExtent) 的影响。如果要从已有的变化出发并考虑缩放限制则可以使用 [*zoom*.translateBy](#zoom_translateBy), [*zoom*.scaleBy](#zoom_scaleBy) 和 [*zoom*.scaleTo](#zoom_scaleTo)。

<a href="#zoom_translateBy" name="zoom_translateBy">#</a> <i>zoom</i>.<b>translateBy</b>(<i>selection</i>, <i>x</i>, <i>y</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L110 "Source")

如果 *selection* 为选择集，则将选中元素上的 [current zoom transform](#zoomTransform) [translates](#transform_translate) *x* 和 *y*(相对于当前位置)，新的 *t<sub>x1</sub>* = *t<sub>x0</sub>* + *kx* ，新的 *t<sub>y1</sub>* = *t<sub>y0</sub>* + *ky*. 如果 *selection* 则会创建一个 “zoom” 补间。这个方法是 [*zoom*.transform](#zoom_transform) 的方便用法。*x* 和 *y* 可以是数字或者返回数字的函数。如果是函数则会为每个选中的元素调用，并传递当前元素绑定的数据 `d` 以及索引 `i`，函数内部 `this` 指向当前 `DOM` 元素。

<a href="#zoom_translateTo" name="zoom_translateTo">#</a> <i>zoom</i>.<b>translateTo</b>(<i>selection</i>, <i>x</i>, <i>y</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L119 "Source")

如果 *selection* 为选择集，则将选中元素上的 [current zoom transform](#zoomTransform) [translates](#transform_translate) 设置为指定的 ⟨*x*,*y*⟩ (不考虑当前变换), 并且移动之后 [viewport extent](#zoom_extent) 中心为指定的坐标。新的 *t<sub>x</sub>* = *c<sub>x</sub>* - *kx*，新的 *t<sub>y</sub>* = *c<sub>y</sub>* - *ky*，其中 ⟨*c<sub>x</sub>*,*c<sub>y</sub>*⟩ 为视图中心。如果 *selection* 为过渡，则会定义一个 “zoom” 补间来过渡当前变换。这个方法是 [*zoom*.transform](#zoom_transform) 的便利用法。*x* 和 *y* 坐标必须为数值或者返回数值的函数，如果是函数则会为每个已选中的元素调用，并传递当前元素绑定的数据 `d` 以及索引 `i`，函数内部 `this` 指向当前 `DOM` 元素。

<a href="#zoom_scaleBy" name="zoom_scaleBy">#</a> <i>zoom</i>.<b>scaleBy</b>(<i>selection</i>, <i>k</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L91 "Source")

如果 *selection* 为选择集，则将已经选中的元素的 [current zoom transform](#zoomTransform) [scales](#transform_scale) 指定的倍数(在当前缩放基础上叠加缩放)，新的 *k₁* = *k₀k*。如果 *selection* 为过渡，则会为当前变换定义一个 “zoom” 补间。这个方法是 [*zoom*.transform](#zoom_transform) 的便捷用法。*t* 必须为数值或者返回数值的函数。如果是函数则会为每个已选中的元素调用，并传递当前元素绑定的数据 `d` 以及索引 `i`，函数内部 `this` 指向当前 `DOM` 元素。

<a href="#zoom_scaleTo" name="zoom_scaleTo">#</a> <i>zoom</i>.<b>scaleTo</b>(<i>selection</i>, <i>k</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L99 "Source")

如果 *selection* 为选择集，则将已经选中的元素的 [current zoom transform](#zoomTransform) [scales](#transform_scale) 到指定的倍数(不考虑当前缩放)，新的 *k₁* = *k*。如果 *selection* 为过渡，则会为当前变换定义一个 “zoom” 补间。这个方法是 [*zoom*.transform](#zoom_transform) 的便捷用法。*t* 必须为数值或者返回数值的函数。如果是函数则会为每个已选中的元素调用，并传递当前元素绑定的数据 `d` 以及索引 `i`，函数内部 `this` 指向当前 `DOM` 元素。

<a href="#zoom_constrain" name="zoom_constrain">#</a> <i>zoom</i>.<b>constrain</b>([<i>constrain</i>]) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#403 "Source")

如果指定了 *constrain* 则将变换限制设置为指定的函数并返回缩放行为。如果没有指定 *constrain* 则返回当前的限制函数，默认为:

```js
function constrain(transform, extent, translateExtent) {
  var dx0 = transform.invertX(extent[0][0]) - translateExtent[0][0],
      dx1 = transform.invertX(extent[1][0]) - translateExtent[1][0],
      dy0 = transform.invertY(extent[0][1]) - translateExtent[0][1],
      dy1 = transform.invertY(extent[1][1]) - translateExtent[1][1];
  return transform.translate(
    dx1 > dx0 ? (dx0 + dx1) / 2 : Math.min(0, dx0) || Math.max(0, dx1),
    dy1 > dy0 ? (dy0 + dy1) / 2 : Math.min(0, dy0) || Math.max(0, dy1)
  );
}
```

限制函数必须给定当前 *transform*, [viewport extent](#zoom_extent) 和 [translate extent](#zoom_translateExtent) 并返回 [*transform*](#zoom-transforms)。默认的实现是试图确保 `viewport` 的范围不会超出缩放的平移区间范围。

<a href="#zoom_filter" name="zoom_filter">#</a> <i>zoom</i>.<b>filter</b>([<i>filter</i>]) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L386 "Source")

如果指定了 *filter*，则将过滤器设置为指定的函数并返回缩放行为。如果没有指定 *filter* 则返回当前过滤器，默认为:

```js
function filter() {
  return !d3.event.button;
}
```

如果过滤器返回假值，则初始事件会被忽略并不会触发缩放手势的开始。因此过滤器可以用来设置哪些事件被忽略。默认的过滤器忽略了次要按钮上的 `mousedown` 事件，因为这些按钮通常被用作其他用途，比如菜单。

<a href="#zoom_touchable" name="zoom_touchable">#</a> <i>zoom</i>.<b>touchable</b>([<i>touchable</i>]) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L390 "Source")

如果指定了 *touchable* 则将触摸支持检测器设置为指定的函数并返回缩放行为。如果没有指定 *touchable* 则返回当前触摸支持检测器，默认为:

```js
function touchable() {
  return "ontouchstart" in this;
}
```

触摸事件通常在触摸支持检测器返回真的时候才会在缩放被 [applied](#_zoom) 被注册到对应的元素。对于大多数能够触摸输入的浏览器来说，默认的检测器很好用，但不是全部; 例如，`Chrome` 的移动设备模拟器无法有效检测。

<a href="#zoom_wheelDelta" name="zoom_wheelDelta">#</a> <i>zoom</i>.<b>wheelDelta</b>([<i>delta</i>]) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L382 "Source")

如果指定了 *delta* 则将滚轮的 `delta` 函数设置为指定的函数并返回缩放行为。如果没有指定 *delta* 则返回滚轮当前的 `delta` 函数，默认为:

```js
function wheelDelta() {
  return -d3.event.deltaY * (d3.event.deltaMode ? 120 : 1) / 500;
}
```

`delata` 函数返回的值 *Δ* 定义了 [WheelEvent](https://developer.mozilla.org/en-US/docs/Web/API/WheelEvent) 与缩放量的影响。缩放因子 [*transform*.k](#zoomTransform) 会乘以 2<sup>*Δ*</sup>。例如 *Δ* + 1 会使缩放因子增加一倍而 *Δ* - 1 会使缩放因子减少一倍。

<a href="#zoom_extent" name="zoom_extent">#</a> <i>zoom</i>.<b>extent</b>([<i>extent</i>]) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L394 "Source")

如果指定了 *extent* 则将当前视口范围设置为指定的数组 [[*x0*, *y0*], [*x1*, *y1*]]，其中 [*x0*, *y0*] 是视口的左上角坐标，[*x1*, *y1*] 是视口的右下角坐标，并返回缩放行为。*extent* 也可以是一个返回数组的函数。如果为函数，则会为每个选中的元素调用，并传递当前数据 `d` 以及索引 `i`，其中 `this` 指向当前 `DOM` 元素。

如果没有指定 *extent* 则返回去当前 `extent` 访问器，默认为 [[0, 0], [*width*, *height*]], 其中 *width* 为元素的 [client width](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientWidth) 而 *height* 为元素的 [client height](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientHeight)；对于 `SVG` 元素(属于 `SVG` 的元素)来说会使用最近的祖先 `SVG` 元素的 [width](https://www.w3.org/TR/SVG/struct.html#SVGElementWidthAttribute) 和 [height](https://www.w3.org/TR/SVG/struct.html#SVGElementHeightAttribute)。在这种情况下，所属 `SVG` 元素必须包含 [width](https://www.w3.org/TR/SVG/struct.html#SVGElementWidthAttribute) 和 [height](https://www.w3.org/TR/SVG/struct.html#SVGElementHeightAttribute) 而不是依赖 `CSS` 样式或者 `viewBox` 属性；`SVG` 不提供检索 [initial viewport size(初始视口大小)](https://www.w3.org/TR/SVG/coords.html#ViewportSpace) 的编程方法。可以考虑使用 [*element*.getBoundingClientRect](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect) (在火狐中，`SVG` 的 [*element*.clientWidth](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientWidth) 和 [*element*.clientHeight](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientHeight) 为零)

视口的范围影响以下几个函数: 由 [*zoom*.scaleBy](#zoom_scaleBy) 和 [*zoom*.scaleTo](#zoom_scaleTo) 引起的改变则视口中心会保持不变；视口中心和维度会影响 [d3.interpolateZoom](https://github.com/d3/d3-interpolate#interpolateZoom) 的路径；为了执行 [translate extent](#zoom_translateExtent) 视口范围是需要的。

<a href="#zoom_scaleExtent" name="zoom_scaleExtent">#</a> <i>zoom</i>.<b>scaleExtent</b>([<i>extent</i>]) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L398 "Source")

如果指定了 *extent* 则将缩放范围设置为指定的 [*k0*, *k1*]，其中 *k0* 为允许缩放的最小因子而 *k1* 为缩放的最大缩放因子，并返回缩放行为。如果没有指定 *extent* 则返回当前缩放范围，默认为 [0, ∞]。缩放范围约束了缩小和放大。在手动交互阶段或者使用 [*zoom*.scaleBy](#zoom_scaleBy), [*zoom*.scaleTo](#zoom_scaleTo) 和 [*zoom*.translateBy](#zoom_translateBy) 的时候执行；然而如果使用 [*zoom*.transform](#zoom_transform) 来明确的指定变换则不会被执行。

如果用户在已经达到缩放边界的情况下继续缩放，则滚轮事件会被忽略并不会初始化缩放手势。这样就允许用户在缩放后滚动到可缩放的区域，或者在缩小后向上滚动。如果你希望无论缩放尺度大小都始终避免上下滚动，则在可以在选择集上注册一个事件监听器来阻止浏览器的默认行为:

```js
selection
    .call(zoom)
    .on("wheel", function() { d3.event.preventDefault(); });
```

<a href="#zoom_translateExtent" name="zoom_translateExtent">#</a> <i>zoom</i>.<b>translateExtent</b>([<i>extent</i>]) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L402 "Source")

如果指定了 *extent* 则将当前的平移区间设置为指定的数组: [[*x0*, *y0*], [*x1*, *y1*]], 其中 [*x0*, *y0*] 为世界的左上角而 [*x1*, *y1*] 为世界的右下角，并返回缩放行为。如果没有指定 *extent* 则返回当前的平移范围，默认为 [[-∞, -∞], [+∞, +∞]]。平移范围限制了视口的移动以及因为缩小引起的平移。它在交互以及使用 [*zoom*.scaleBy](#zoom_scaleBy), [*zoom*.scaleTo](#zoom_scaleTo) 和 [*zoom*.translateBy](#zoom_translateBy) 的时候执行；然而如果使用 [*zoom*.transform](#zoom_transform) 来明确的指定变换则不会被执行。

<a href="#zoom_clickDistance" name="zoom_clickDistance">#</a> <i>zoom</i>.<b>clickDistance</b>([<i>distance</i>]) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L419 "Source")

如果指定了 *distance* 则将`click` 事件的触发条件: `mousedown` 和 `mouseup` 之间鼠标移动的距离设置为指定的距离。如果鼠标按下时的坐标与鼠标抬起时的坐标之间的距离大于或等于 *distance* 则不会触发随后的 `click` 事件。如果没有指定 *distance* 则返回当前的默认值，默认为 0。距离阈值通过坐标系统 ([*event*.clientX](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/clientX) 和 [*event*.clientY](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent/clientY)) 测量得到。

<a href="#zoom_duration" name="zoom_duration">#</a> <i>zoom</i>.<b>duration</b>([<i>duration</i>]) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L406 "Source")

如果指定了 *duration* 则将双击放大的过渡事件设置为指定的数值(毫秒)并返回缩放行为，如果没有指定 *duration* 则返回当前的过渡时长，默认为 `250ms`。如果过渡时长小于等于 `0` 则在双击放大时不会有平滑过渡效果。

如果要禁用双击放大则可以在应用缩放行为之后移除选择集的缩放行为的双击事件:

```js
selection
    .call(zoom)
    .on("dblclick.zoom", null);
```

<a href="#zoom_interpolate" name="zoom_interpolate">#</a> <i>zoom</i>.<b>interpolate</b>([<i>interpolate</i>]) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L410 "Source")

如果指定了 *interpolate* 则设置缩放过渡插值工厂为指定的函数。如果没有指定 *interpolate* 则返回当前的插值器工厂，默认为实现平滑缩放过渡的 [d3.interpolateZoom](https://github.com/d3/d3-interpolate#interpolateZoom)。如果在两个视图之间直接插值，则可以尝试使用 [d3.interpolate](https://github.com/d3/d3-interpolate#interpolate) 代替。

<a href="#zoom_on" name="zoom_on">#</a> <i>zoom</i>.<b>on</b>(<i>typenames</i>[, <i>listener</i>]) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/zoom.js#L414 "Source")

如果指定了 *listener* 则将 *listener* 设置为指定 *typenames* 的监听器并返回缩放新闻。如果已经注册了同类型以及同名的事件监听器则会被覆盖。如果 *listener* 为 `null` 则表示移除指定 *typenames* 的事件监听器(如果存在的话)。如果没有指定 *listener* 则返回当前第一个已经被分配的并且匹配 *typenames* 的事件监听器。当指定的事件被触发的时候，对应的 *listener* 会被执行，参数与上下文与 [*selection*.on](https://github.com/xswei/d3-selection#selection_on) 一致: 当前绑定的数据 `d` 以及索引 `i`，`this` 指向当前 `DOM` 元素。

*typenames* 是一个包含一个或多个由空格隔开的 *typename* 的字符串。每个 *typename* 都是一个 *type*，后面可以跟一个由 `.` 隔开的 *name*, 比如 `zoom.foo` 和 `zoom.bar`；通过 `name` 可以为同一种 *type* 同时指定多个事件监听器，*type* 为以下几种: 

* `start` - `zooming` 开始之后(比如`mousedown`)
* `zoom` - `zoom` 的变换发生了变化(比如 `mousemove`)
* `end` - `zoom` 结束(比如 `mouseup`)

参考 [*dispatch*.on](https://github.com/xswei/d3-dispatch#dispatch_on) 获取更多信息.

### Zoom Events

当 [zoom event listener](#zoom_on) 被调用时，[d3.event](https://github.com/xswei/d3-selection#event) 被设置为当前缩放事件。*event* 对象暴露以下几个字段:

* *event*.target - 关联的 [zoom behavior](#zoom).
* *event*.type - 字符串 “start”, “zoom” 或 “end”; 参考 [*zoom*.on](#zoom_on).
* *event*.transform - 当前 [zoom transform](#zoom-transforms).
* *event*.sourceEvent - 底层输入事件，比如 `mousemove` 或 `touchmove`.

### Zoom Transforms

缩放行为载缩放被 [applied](#_zoom) 的时候将缩放状态存储在元素上，而不是存储在缩放行为自身。这是因为缩放行为可能同时应用在多个元素上，并且每个元素的缩放都是独立的。缩放状态可以通过交互或 通过 [*zoom*.transform](#zoom_transform) 编程的形式改变。

为了检索缩放的状态，请在当前 [zoom event](#zoom-events) 事件监听器内使用 *event*.transform 来获取(参考 [*zoom*.on](#zoom_on))，或者使用为给定节点使用 [d3.zoomTransform](#zoomTransform)。后者在通过编程的形式缩放状态时很有用，比如要实现缩小和放大的按钮。

<a href="#zoomTransform" name="zoomTransform">#</a> d3.<b>zoomTransform</b>(<i>node</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/transform.js "Source")

返回指定 *node* 上当前缩放变换。*node* 通常是 `DOM` 元素而不是 *selection*。(选择集中可能包含多个状态不同的元素，而这个方法会返回一个单一的变换)。如果你有一个选择集则可以先调用 [*selection*.node](https://github.com/xswei/d3-selection#selection_node):

```js
var transform = d3.zoomTransform(selection.node());
```

在 [event listener](https://github.com/d3/d3-selection#selection_on) 的上下文中，*node* 通常是接收事件输入的元素(应该等价于 [*event*.transform](#zoom-events))，*this*:

```js
var transform = d3.zoomTransform(this);
```

在内部，元素的变换存储在 *element*.\_\_zoom；但是应该通过方法去修改它，而最好不要直接修改。如果给定的 *node* 没有定义变换，则返回 [identity transformation](#zoomIdentity)。返回的变换表示了一个变换矩阵:

*k* 0 *t<sub>x</sub>*
<br>0 *k* *t<sub>y</sub>*
<br>0 0 1

(这个矩阵只能表示缩放和平移，未来可能会支持旋转，尽管不会向后兼容)。坐标 ⟨*x*,*y*⟩ 被变换到 ⟨*xk* + *t<sub>x</sub>*,*yk* + *t<sub>y</sub>*⟩。变换对象暴露以下属性:

* *transform*.x - 在*x*-轴上的平移量 *t<sub>x</sub>*
* *transform*.y - 在*y*-轴上的平移量 *t<sub>y</sub>*
* *transform*.k - 缩放因子 *k*

这些属性应该被认为是只读的；不要直接去修改变换，而是通过 [*transform*.scale](#transform_scale) 和 [*transform*.translate](#transform_translate) 去获取一个新的转换。参考 [*zoom*.scaleBy](#zoom_scaleBy), [*zoom*.scaleTo](#zoom_scaleTo) 和 [*zoom*.translateBy](#zoom_translateBy) 这些更方便的方法。根据给定的 *k*, *t<sub>x</sub>*, 和 *t<sub>y</sub>* 创建一个变换:

```js
var t = d3.zoomIdentity.translate(x, y).scale(k);
```

将变换应用到 [Canvas 2D context](https://www.w3.org/TR/2dcontext/)，则使用在 [*context*.translate](https://www.w3.org/TR/2dcontext/#dom-context-2d-translate) 之后使用 [*context*.scale](https://www.w3.org/TR/2dcontext/#dom-context-2d-scale):

```js
context.translate(transform.x, transform.y);
context.scale(transform.k, transform.k);
```

类似的将变换通过 [CSS](https://www.w3.org/TR/css-transforms-1/) 应用到 `HTML` 元素:

```js
div.style("transform", "translate(" + transform.x + "px," + transform.y + "px) scale(" + transform.k + ")");
div.style("transform-origin", "0 0");
```

将变换应用到 [SVG](https://www.w3.org/TR/SVG/coords.html#TransformAttribute):

```js
g.attr("transform", "translate(" + transform.x + "," + transform.y + ") scale(" + transform.k + ")");
```

更方便的是，使用 [*transform*.toString](#transform_toString) 的优势:

```js
g.attr("transform", transform);
```

要注意变换的顺序，平移一定要在缩放前。

<a href="#transform_scale" name="transform_scale">#</a> <i>transform</i>.<b>scale</b>(<i>k</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/transform.js#L9 "Source")

返回一个 *k₁* 等于 *k₀k* 的变换，其中 *k₀* 是当前变换的缩放因子(也就是将当前变换缩小或放大一个倍数)。

<a href="#transform_translate" name="transform_translate">#</a> <i>transform</i>.<b>translate</b>(<i>x</i>, <i>y</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/transform.js#L12 "Source")

返回一个平移量 *t<sub>x1</sub>* 和 *t<sub>y1</sub>* 分别等于 *t<sub>x0</sub>* + *x* 和 *t<sub>y0</sub>* + *y* 的变换，其中 *t<sub>x0</sub>* 和 *t<sub>y0</sub>* 当前变换的平移量。

<a href="#transform_apply" name="transform_apply">#</a> <i>transform</i>.<b>apply</b>(<i>point</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/transform.js#L15 "Source")

返回指定的点经过当前变换变换后的坐标，其中 *point* 由二元数字数组 [*x*, *y*] 表示。返回的坐标等价于 [*xk* + *t<sub>x</sub>*, *yk* + *t<sub>y</sub>*].

<a href="#transform_applyX" name="transform_applyX">#</a> <i>transform</i>.<b>applyX</b>(<i>x</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/transform.js#L18 "Source")

返回指 *x* 坐标经过变换后的 *x* 坐标，等于 *xk* + *t<sub>x</sub>*。

<a href="#transform_applyY" name="transform_applyY">#</a> <i>transform</i>.<b>applyY</b>(<i>y</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/transform.js#L21 "Source")

返回指 *y* 坐标经过变换后的 *y* 坐标，等于 *yk* + *t<sub>y</sub>*。

<a href="#transform_invert" name="transform_invert">#</a> <i>transform</i>.<b>invert</b>(<i>point</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/transform.js#L24 "Source")

返回指定 *point* 的逆变换，*point* 由二元数组 [*x*, *y*] 表示。返回的点 *point* 等价于 [(*x* - *t<sub>x</sub>*) / *k*, (*y* - *t<sub>y</sub>*) / *k*]。

<a href="#transform_invertX" name="transform_invertX">#</a> <i>transform</i>.<b>invertX</b>(<i>x</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/transform.js#L27 "Source")

返回指定 *x* 坐标的逆变换，等于 (*x* - *t<sub>x</sub>*) / *k*。

<a href="#transform_invertY" name="transform_invertY">#</a> <i>transform</i>.<b>invertY</b>(<i>y</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/transform.js#L30 "Source")

返回指定 *y* 坐标的逆变换，等于 (*y* - *t<sub>y</sub>*) / *k*。

<a href="#transform_rescaleX" name="transform_rescaleX">#</a> <i>transform</i>.<b>rescaleX</b>(<i>x</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/transform.js#L33 "Source")

返回指定 [continuous scale](https://github.com/xswei/d3-scale#continuous-scales) *x*，且 [domain](https://github.com/xswei/d3-scale#continuous_domain) 结果变换的拷贝。这是通过首先在刻度范围内应用 [inverse *x*-transform](#transform_invertX) ，然后使用 [inverse scale](https://github.com/xswei/d3-scale#continuous_invert) 来计算响应的值域实现的:

```js
function rescaleX(x) {
  var range = x.range().map(transform.invertX, transform),
      domain = range.map(x.invert, x);
  return x.copy().domain(domain);
}
```

比例尺 *x* 必须使用 [d3.interpolateNumber](https://github.com/xswei/d3-interpolate#interpolateNumber)；而不能使用 [*continuous*.rangeRound](https://github.com/xswei/d3-scale#continuous_rangeRound)，因为会降低 [*continuous*.invert](https://github.com/d3/d3-scale#continuous_invert) 的准确性并且可能导致缩放之后 `domain` 的错乱。这个方法不会修改输入比例尺 *x*；因此 *x* 表示未变换的比例尺，而返回的比例尺则表示经过缩放变换之后的比例尺。

<a href="#transform_rescaleY" name="transform_rescaleY">#</a> <i>transform</i>.<b>rescaleY</b>(<i>y</i>) [<源码>](https://github.com/d3/d3-zoom/blob/master/src/transform.js#L36 "Source")

返回指定 [continuous scale](https://github.com/xswei/d3-scale#continuous-scales) *y*，且 [domain](https://github.com/xswei/d3-scale#continuous_domain) 结果变换的拷贝。这是通过首先在刻度范围内应用 [inverse *y*-transform](#transform_invertY) ，然后使用 [inverse scale](https://github.com/xswei/d3-scale#continuous_invert) 来计算响应的值域实现的:

```js
function rescaleY(y) {
  var range = y.range().map(transform.invertY, transform),
      domain = range.map(y.invert, y);
  return y.copy().domain(domain);
}
```

比例尺 *y* 必须使用 [d3.interpolateNumber](https://github.com/d3/d3-interpolate#interpolateNumber)；而不能使用 [*continuous*.rangeRound](https://github.com/d3/d3-scale#continuous_rangeRound)，因为会降低 [*continuous*.invert](https://github.com/d3/d3-scale#continuous_invert) 的准确性并且可能导致缩放之后 `domain` 的错乱。这个方法不会修改输入比例尺 *y*；因此 *y* 表示未变换的比例尺，而返回的比例尺则表示经过缩放变换之后的比例尺。

<a href="#transform_toString" name="transform_toString">#</a> <i>transform</i>.<b>toString</b>() [<源码>](https://github.com/d3/d3-zoom/blob/master/src/transform.js#L39 "Source")

返回一个能直接被 [SVG transform](https://www.w3.org/TR/SVG/coords.html#TransformAttribute) 识别的表示变换的字符串，通过一下方式实现:

```js
function toString() {
  return "translate(" + this.x + "," + this.y + ") scale(" + this.k + ")";
}
```

<a href="#zoomIdentity" name="zoomIdentity">#</a> d3.<b>zoomIdentity</b> [<源码>](https://github.com/d3/d3-zoom/blob/master/src/transform.js#L44 "Source")

恒等变换，其中 *k* = 1, *t<sub>x</sub>* = *t<sub>y</sub>* = 0.