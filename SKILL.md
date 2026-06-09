---
name: "dataviz-interactive-dashboard"
description: "创建 R 数据可视化交互看板（DataViz Studio）类网页。当用户需要生成含 ECharts 图表、R代码解析、CSV上传的交互式数据可视化网页时调用此 Skill。支持直方图、散点图、地图、流向图、桑基图、热力图、雷达图等。"
---

# DataViz Studio - R 数据可视化交互看板

## 概述

这是一个基于纯前端技术栈构建的 **R 数据可视化交互看板**，可在单个 HTML 文件中运行，无需后端服务器。核心能力是将 R ggplot2 风格的数据分析流程转化为可交互的 ECharts 网页图表。

## 适用场景

- 需要生成一个**可点击交互**的数据可视化网页（发送链接即可访问）
- 将 R 语言图表转为网页版交互图表
- 需要支持 CSV 数据上传和实时图表渲染的看板
- 展示中国地理数据（灯光强度、CFPS 样本等）
- 快速搭建数据分析演示原型

## 技术栈

| 类别 | 技术 | 用途 |
|------|------|------|
| CSS 框架 | Tailwind CSS (CDN) | 页面布局与样式 |
| 图表引擎 | ECharts 5.4.3 + ECharts GL | 核心图表渲染（2D + 3D） |
| 地图 | Leaflet 1.9.4 | 辅助地图展示 |
| 代码编辑器 | CodeMirror 5.65 | JavaScript 代码编辑 |
| 导出 | html2canvas + FileSaver | 图表截图导出 |
| 图标 | Font Awesome 6.4 | UI 图标 |
| 动画 | Animate.css 4.1 | 入场动画效果 |

## 核心功能模块

### 1. 图库（Gallery）- 4 个预设模板

| 模板 | 图表类型 | 数据来源 | 交互特性 |
|------|---------|---------|---------|
| 风速数据分析 | 直方图 + 箱线图（双面板） | airquality 内置数据 | Hover 查看数值分布 |
| 散点回归分析 | 散点图 + 线性回归线 | mtcars 内置数据 | 异常点红色三角标记，悬停查看车型详情 |
| 灯光强度地图 | 中国地图 + 气泡图 | chinaLightCities 模拟数据 | 滚轮缩放、拖拽漫游、气泡大小代表样本量 |
| 流量网络图 | 流向图 + 饼图 | 同上 | 动画粒子流向效果、饼图展示流量等级分布 |

### 2. 额外代码模板（Code Templates）

除图库 4 个模板外，代码中还预置了 3 个额外模板：

- **heatmap**：时空热力图（一周各时段灯光活跃度）
- **sankey**：CFPS 区域间样本流动桑基图
- **radar**：城市综合指标多维度雷达图

### 3. R 代码解析器（R Code Parser）

- 支持解析 R `ggplot2` 代码字符串
- 自动识别几何图层类型：`geom_point`、`geom_histogram`、`geom_boxplot`、`geom_bar`、`geom_line`、`geom_sf`、`geom_curve`、`geom_smooth`、`geom_density`、`geom_violin`、`geom_area` 等
- 提取美学映射（`aes`）、主题（`theme_*`）、标签（`labs`）、坐标系（`coord_flip`/`coord_polar`）、分面（`facet_wrap`/`facet_grid`）
- 自动将解析结果转换为 ECharts 配置并运行

### 4. CSV 数据上传

- 支持点击上传和拖拽上传 CSV 文件
- 自动解析表头和数据类型（数字/字符串）
- 数据预览面板显示前 5 行
- 上传的数据注入为全局变量 `window.uploadedData`

### 5. 代码编辑器（CodeMirror）

- JavaScrip t 语法高亮
- 支持 `Ctrl+Enter` 快捷键运行代码
- 内置 `renderChart(option)` 函数接收 ECharts 配置
- 注入的数据集：`airquality`、`mtcars`、`chinaLightCities`
- 注入的函数：`loadChinaGeoJSON(callback)` 加载中国地图 GeoJSON

### 6. 主题切换

- 亮色/暗色两种主题
- CSS 变量驱动的全局配色切换
- 包含毛玻璃效果背景动画

### 7. 图表导出

- PNG 图片导出（2x 像素比）
- SVG 矢量图导出
- JSON 配置数据导出

### 8. 响应式设计

- 桌面端三栏布局（侧边栏 + 编辑器 + 预览面板）
- 平板端隐藏侧边栏
- 移动端垂直布局

## 中国地图数据

- 优先尝试从阿里云 DataV API 加载完整 GeoJSON
- 网络不可用时使用内置的简化版 GeoJSON（34 个省/市/自治区简化边界）
- 加载机制：首次使用时 XHR 请求，后续调用使用缓存

## 内置数据集

### airquality（纽约空气质量，153 条）
```javascript
{ Month, Day, Ozone, Solar_R, Wind, Temp }
```

### mtcars（汽车性能，32 条）
```javascript
{ name, wt, disp, cyl, mpg }
```

### chinaLightCities（中国城市灯光数据，35 条）
```javascript
{ name, lon, lat, dn, cfps }
```

## 关键函数说明

### renderChart(option)
接收 ECharts option 配置对象，在预览面板中渲染图表。每次调用会先销毁之前的图表实例。

### loadChinaGeoJSON(callback)
异步加载中国地图 GeoJSON 数据。加载完成后调用 `callback(geoJson)`，内部使用 `echarts.registerMap('china', geoJson)` 注册。

### runCode()
从 CodeMirror 编辑器获取代码并使用 `new Function()` 执行，安全沙箱中注入数据集和工具函数。

### loadTemplate(name)
加载预设代码模板到编辑器并自动运行渲染图表。

## 创建网页时的要点

1. **CDN 依赖**必须保持可访问：Tailwind CSS、ECharts、Leaflet、CodeMirror、Font Awesome 均通过 CDN 加载
2. **GeoJSON 加载**需要设置合理的超时和回退机制
3. **图表容器**必须设置明确的宽高（`min-height: 450px`）
4. **响应式处理**：窗口 resize 时调用 `chartInstance.resize()`
5. **代码安全性**：使用 `new Function()` 执行用户代码，限制可访问的全局变量
6. **深色主题**：通过 `[data-theme="dark"]` 选择器和 CSS 变量实现

## 部署方式

- **单文件部署**：所有 CSS/JS 内联，复制 `data-visualization.html` + `assets/` 文件夹即可
- **本地服务器**：`python -m http.server 8080`
- **公网部署**：上传到 Netlify/Vercel/GitHub Pages 等静态托管服务
- **局域网分享**：同一 WiFi 下通过 IP 地址访问
- **公网隧道**：使用 cloudflared/ngrok/localtunnel 暴露本地服务