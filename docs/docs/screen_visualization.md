---
title: 前端大屏可视化项目适配方案
createTime: 2024/10/03 22:38:17
permalink: /article/w28ukmwf/
---
#### echarts在vue或者react中使用存在的问题

- 每个图表需要从头到尾写一遍完整的option配置，十分冗余
- 在同一个项目中，各类图表设计十分相似，甚至是相同，没必要一直做重复工作
- 窗口缩放时的适应问题
- 在项目中如何按需引入echarts图表 减少打包体积的大小

####  解决方案

一种方案

- 使用了scale方案
  通过 scale 属性，根据屏幕大小，对图表进行整体的等比缩放
  其实是有一定的问题的：
  **1.因为是根据 ui 稿等比缩放，当大屏跟 ui 稿的比例不一样时，会出现周边留白情况 
  ****2.当缩放比例过大时候，字体会有一点点模糊，就一点点** 

​      **3.当缩放比例过大时候，事件热区会偏移。**

```html
<div className="screen-wrapper">
  <div className="screen" id="screen">
  </div>
</div>
<script>
  export default {
    mounted() {
      // 初始化自适应  ----在刚显示的时候就开始适配一次
      handleScreenAuto();
      // 绑定自适应函数   ---防止浏览器栏变化后不再适配
      window.onresize = () => handleScreenAuto();
    },
    deleted() {
      window.onresize = null;
    },
    methods: {
      // 数据大屏自适应函数
      handleScreenAuto() {
        const designDraftWidth = 1920; //设计稿的宽度
        const designDraftHeight = 1080; //设计稿的高度
        // 根据屏幕的变化适配的比例
        const scale =
          document.documentElement.clientWidth /
          document.documentElement.clientHeight <
          designDraftWidth / designDraftHeight
          ? document.documentElement.clientWidth / designDraftWidth
          : document.documentElement.clientHeight / designDraftHeight;
        // 缩放比例
        document.querySelector(
          '#screen',
        ).style.transform = `scale(${scale}) translate(-50%, -50%)`;
      },
    },
  };
</script>

/*
除了设计稿的宽高是根据您自己的设计稿决定以外，其他复制粘贴就完事
*/  
.screen-root {
  height: 100%;
  width: 100%;
  .screen {
    display: inline-block;
    width: 1920px;  //设计稿的宽度
    height: 1080px;  //设计稿的高度
    transform-origin: 0 0;
    position: absolute;
    left: 50%;
    top: 50%;
  }
}
```

采用了另外的一直方式

- vw vh方案：
  实现方式：
  按照设计稿的尺寸，将px按比例计算转为vw和vh
  优点：
- 1.可以动态计算图表的宽高，字体等，灵活性较高 
- 2.当屏幕比例跟 ui 稿不一致时，不会出现两边留白情况
  缺点：每个图表都需要单独做字体、间距、位移的适配，比较麻烦


```javascript
import * as echarts from 'echarts/core'
import customedTheme from './theme/customed.json'

// 引入柱状图图表，图表后缀都为 Chart
import { BarChart, LineChart, PieChart } from 'echarts/charts'
// 引入提示框，标题，直角坐标系，数据集，内置数据转换器组件，组件后缀都为 Component
import { TitleComponent, TooltipComponent, LegendComponent, GridComponent, GraphicComponent, DataZoomComponent } from 'echarts/components'
// 标签自动布局、全局过渡动画等特性
import { LabelLayout, UniversalTransition } from 'echarts/features'
// 引入 Canvas 渲染器，注意引入 CanvasRenderer 或者 SVGRenderer 是必须的一步
import { CanvasRenderer } from 'echarts/renderers'

echarts.use([
  TitleComponent,
  TooltipComponent,
  LegendComponent,
  GridComponent,
  GraphicComponent,
  DataZoomComponent,
  BarChart,
  LineChart,
  PieChart,
  LabelLayout,
  UniversalTransition,
  CanvasRenderer
])

echarts.registerTheme('customed', customedTheme)
export default echarts
```

 

**1. Canvas 渲染**

​	**优点**：

​	•**性能优越**：Canvas 渲染是基于位图的，绘制速度较快，适合处理大量数据，尤其是当图表中元素数量非常多时。

​	•**动画流畅**：Canvas 能够很好地支持高性能的动态渲染，适合展示实时数据更新和复杂动画效果。

​	•**内存占用低**：相对于 SVG，Canvas 对内存的使用更高效，尤其是当图表中元素数量庞大时。

​	**缺点**：

​	•**不可编辑性**：Canvas 渲染的图形是一个位图，一旦绘制完成，无法像 SVG 那样直接修改元素（例如动态添加、删除图形）。

​	•**SEO 和可访问性差**：Canvas 内容不容易被搜索引擎抓取和解析，也不适合需要无障碍访问的场景。

​	**适用场景**：

​	•大量数据（如散点图、热力图、实时数据流图等）。

​	•高性能需求场景（如实时动态更新、动画效果等）。

​	•需要绘制复杂图形、覆盖面积较大的图表。

**2. SVG 渲染**

​	**优点**：

​	•**高质量的图形**：SVG 是矢量图形，具有无限缩放的特性，不会失真，适合用于图表的精准显示。

​	•**易于交互和修改**：SVG 可以通过 DOM 操作直接修改图形元素，支持动态交互和元素的编辑、删除等操作。

​	•**良好的可访问性**：SVG 图形可以通过屏幕阅读器等工具进行访问，适用于需要无障碍支持的场景。

​	•**SEO 支持**：SVG 是文本文件，能够被搜索引擎解析，适用于需要被索引的图表内容。

​	**缺点**：

​	•**性能较差**：SVG 是基于 XML 的，对于图形较复杂或数据量庞大的图表，性能不如 Canvas。

​	•**内存占用较高**：每个图形元素都是 DOM 节点，元素数量增多时会占用更多内存，渲染速度也可能变慢。

​	**适用场景**：

​	•少量数据和静态图表（如饼图、条形图等）。

​	•需要交互、动画和实时更新的场景。

​	•需要较高质量图形的场景（如响应式设计、需要高精度显示的图表）。

**总结**

​	•**Canvas** 适合**数据量大**、**需要高性能**和复杂动画的图表。

​	•**SVG** 适合**图形较简单**、**需要交互**和**注重质量**的图表，特别是数据量较小、精确度要求高的场景。