# PPTist 项目文档

> 文档生成模型：GLM-5.1 | 版本：2.0.0 | 生成日期：2026-04-11

---

## 目录

- [1. 项目概述](#1-项目概述)
- [2. 功能说明](#2-功能说明)
  - [2.1 基础功能](#21-基础功能)
  - [2.2 幻灯片页面编辑](#22-幻灯片页面编辑)
  - [2.3 幻灯片元素编辑](#23-幻灯片元素编辑)
  - [2.4 幻灯片放映](#24-幻灯片放映)
  - [2.5 移动端功能](#25-移动端功能)
  - [2.6 AI 生成 PPT](#26-ai-生成-ppt)
- [3. 使用方法](#3-使用方法)
  - [3.1 环境要求](#31-环境要求)
  - [3.2 安装与运行](#32-安装与运行)
  - [3.3 快捷键参考](#33-快捷键参考)
  - [3.4 导入导出操作](#34-导入导出操作)
  - [3.5 自定义元素开发](#35-自定义元素开发)
- [4. 技术架构](#4-技术架构)
  - [4.1 架构总览](#41-架构总览)
  - [4.2 技术栈选型](#42-技术栈选型)
  - [4.3 目录结构](#43-目录结构)
  - [4.4 核心模块设计](#44-核心模块设计)
  - [4.5 数据模型设计](#45-数据模型设计)
  - [4.6 状态管理设计](#46-状态管理设计)
  - [4.7 画布与元素原理](#47-画布与元素原理)
  - [4.8 历史快照机制](#48-历史快照机制)
- [5. API 接口规范](#5-api-接口规范)
  - [5.1 服务端接口](#51-服务端接口)
  - [5.2 Store Actions 接口](#52-store-actions-接口)
  - [5.3 Hooks 接口](#53-hooks-接口)
  - [5.4 类型定义规范](#54-类型定义规范)
- [6. 注意事项](#6-注意事项)
  - [6.1 项目定位与适用场景](#61-项目定位与适用场景)
  - [6.2 功能边界与限制](#62-功能边界与限制)
  - [6.3 开源协议与商用](#63-开源协议与商用)
  - [6.4 性能优化建议](#64-性能优化建议)
  - [6.5 常见问题](#65-常见问题)
  - [6.6 部署说明](#66-部署说明)

---

## 1. 项目概述

**PPTist**（PowerPoint-ist /'pauəpɔintist/）是一个基于 Web 的在线演示文稿（幻灯片）应用，使用 Vue 3 + TypeScript 构建，还原了大部分 Office PowerPoint 常用功能。项目支持文字、图片、形状、线条、图表、表格、视频、音频、公式九种元素类型，可在 Web 浏览器中完成幻灯片的编辑与演示。

### 项目特色

| 特色 | 说明 |
|------|------|
| 易开发 | 基于 Vue 3 + TypeScript，不依赖 UI 组件库，样式定制轻松、功能扩展方便 |
| 易使用 | 右键菜单、数十种快捷键、精细化编辑体验，力求还原桌面应用级体验 |
| 功能丰富 | 支持 PPT 大部分常用元素和功能，支持 AI 生成 PPT、多格式导出、移动端基础编辑 |

### 在线体验

- 官方地址：https://pipipi-pikachu.github.io/PPTist/
- 国内镜像：[Gitee](https://gitee.com/pptist/PPTist)、[GitCode](https://gitcode.com/pipipi-pikachu/PPTist)

---

## 2. 功能说明

### 2.1 基础功能

| 功能 | 说明 |
|------|------|
| 历史记录 | 支持撤销（Ctrl+Z）和重做（Ctrl+Y），最多保留 20 步快照 |
| 快捷键 | 覆盖通用操作、幻灯片编辑、元素操作、文本编辑、表格编辑等场景 |
| 右键菜单 | 根据当前选中对象动态展示相关操作菜单 |
| 导出文件 | 支持 PPTX、JSON、图片（PNG/JPEG）、PDF 格式导出 |
| 导入文件 | 支持 PPTX、JSON、.pptist 特有格式导入 |
| 打印 | 支持浏览器原生打印功能 |
| AI 生成 PPT | 基于模板的 AI 自动生成演示文稿 |

### 2.2 幻灯片页面编辑

| 功能分类 | 操作说明 |
|----------|----------|
| 页面管理 | 添加、删除、复制粘贴、顺序调整（拖拽排序） |
| 页面分节 | 通过 SectionTag 标记幻灯片分节信息 |
| 背景设置 | 纯色、渐变（线性/径向）、图片背景，支持 cover/contain/repeat 模式 |
| 画布尺寸 | 默认 1000×562.5（16:9），支持 16:10、4:3、A3 等比例 |
| 辅助工具 | 网格线、标尺、画布缩放与移动 |
| 主题设置 | 背景色、主题色、字体颜色、字体、阴影、边框等全局主题 |
| 风格提取 | 从已有幻灯片中提取主题风格 |
| 演讲者备注 | 富文本格式的备注编辑 |
| 幻灯片模板 | 内置 8 套模板，支持自定义模板 |
| 翻页动画 | 无、随机、左右推移、上下推移、3D推移、淡入淡出、旋转、缩放等 12 种 |
| 元素动画 | 入场动画（8 类 40+ 种）、退场动画（8 类 30+ 种）、强调动画（2 类 12 种） |
| 选择面板 | 隐藏/显示元素、层级排序、元素命名 |
| 类型标注 | 页面类型标注（封面/目录/过渡/内容/结束）和节点类型标注，用于模板与 AIPPT |
| 查找替换 | 全局文本查找与替换 |
| 批注 | 支持添加和回复批注 |

### 2.3 幻灯片元素编辑

#### 2.3.1 通用操作

| 操作 | 说明 |
|------|------|
| 添加/删除 | 通过工具栏插入或快捷键创建，Delete/Backspace 删除 |
| 复制粘贴 | Ctrl+C/V 复制粘贴，Ctrl+D 快速复制，支持跨应用粘贴图片和文本 |
| 拖拽移动 | 鼠标拖拽移动元素位置，方向键微调 |
| 旋转 | 旋转手柄或属性面板设置旋转角度 |
| 缩放 | 八个缩放控制点，按住 Ctrl/Shift 锁定宽高比 |
| 多选 | 框选或 Ctrl/Shift 点选多元素 |
| 组合 | Ctrl+G 组合，Ctrl+Shift+G 取消组合 |
| 锁定 | Ctrl+L 锁定元素防止误操作 |
| 对齐吸附 | 移动和缩放时自动吸附对齐线 |
| 层级调整 | 置顶/置底/上移/下移 |
| 对齐分布 | 对齐到画布、对齐到其他元素、均匀分布 |
| 超链接 | 链接到网页或链接到其他幻灯片页面 |
| 坐标尺寸 | 精确设置元素 left/top/width/height/rotate |

#### 2.3.2 文字元素

| 属性 | 说明 |
|------|------|
| 富文本编辑 | 颜色、高亮、字体、字号、加粗、斜体、下划线、删除线、角标、行内代码、引用、超链接、对齐方式、序号、项目符号、段落缩进、清除格式 |
| 排版 | 行高、字间距、段间距、首行缩进 |
| 外观 | 填充色、边框、阴影、透明度 |
| 竖向文本 | 支持竖排文字 |
| AI 辅助 | AI 改写/扩写/缩写 |

#### 2.3.3 图片元素

| 属性 | 说明 |
|------|------|
| 裁剪 | 自定义裁剪、按形状裁剪、按纵横比裁剪 |
| 样式 | 圆角、滤镜（模糊/亮度/对比度/灰度/饱和度/色相旋转/不透明度/棕褐色/反色）、着色（蒙版） |
| 变换 | 翻转（水平/垂直）、边框、阴影 |
| 操作 | 替换图片、重置图片、设置为背景图 |

#### 2.3.4 形状元素

| 属性 | 说明 |
|------|------|
| 绘制 | 任意多边形绘制、任意线条（未封闭形状模拟） |
| 填充 | 纯色、渐变（线性/径向）、图片填充、图案填充 |
| 样式 | 边框、阴影、透明度、翻转 |
| 文字 | 支持富文本编辑（与文字元素近似） |
| 格式刷 | 形状格式刷快速复制样式 |
| 替换 | 替换为其他形状 |

#### 2.3.5 线条元素

| 属性 | 说明 |
|------|------|
| 类型 | 直线、基础折线（单折/双折）、曲线（二次/三次贝塞尔） |
| 样式 | 颜色、宽度、线型（实线/虚线/点线） |
| 端点 | 无、箭头、圆点 |

#### 2.3.6 图表元素

| 图表类型 | 说明 |
|----------|------|
| 柱状图/条形图 | 支持堆积模式 |
| 折线图/面积图 | 支持平滑曲线、堆积模式 |
| 饼图/环形图 | - |
| 散点图/雷达图 | - |

通用属性：图表类型转换、数据编辑、背景填充、主题色设置、坐标轴/文字颜色、网格颜色

#### 2.3.7 表格元素

| 属性 | 说明 |
|------|------|
| 结构 | 行列添加删除、合并单元格 |
| 主题 | 主题色、表头行、汇总行、第一列、最后一列 |
| 单元格样式 | 填充色、文字颜色、加粗、斜体、下划线、删除线、对齐方式 |
| 边框 | 全局边框设置 |

#### 2.3.8 视频元素

| 属性 | 说明 |
|------|------|
| 播放 | 预览封面设置、自动播放 |
| 格式 | 支持通过 MSE 播放特殊格式视频 |

#### 2.3.9 音频元素

| 属性 | 说明 |
|------|------|
| 播放 | 图标颜色、自动播放、循环播放 |

#### 2.3.10 公式元素

| 属性 | 说明 |
|------|------|
| 编辑 | LaTeX 语法编辑，内置符号面板 |
| 样式 | 颜色设置、公式线条粗细设置 |

### 2.4 幻灯片放映

| 功能 | 说明 |
|------|------|
| 画笔工具 | 画笔/形状/箭头/荧光笔标注、橡皮擦除、黑板模式 |
| 幻灯片预览 | 全部幻灯片缩略图预览 |
| 缩略图导航 | 触底显示缩略图快速导航 |
| 计时器 | 演讲计时工具 |
| 激光笔 | 演示激光笔效果 |
| 自动放映 | 自动按设定时间切换页面 |
| 演讲者视图 | 双屏模式，演讲者可查看备注和控制进度 |
| 观众视图 | 观众仅看到幻灯片内容 |

### 2.5 移动端功能

| 功能 | 说明 |
|------|------|
| 基础编辑 | 页面添加/删除/复制/备注/撤销重做 |
| 插入元素 | 文字、图片、矩形、圆形 |
| 元素操作 | 移动、缩放、旋转、复制、删除、层级调整、对齐 |
| 元素样式 | 文字（加粗/斜体/下划线/删除线/字号/颜色/对齐）、填充色 |
| 预览 | 基础预览与播放预览 |

### 2.6 AI 生成 PPT

AIPPT 采用模板式生成方案，流程如下：

1. **定义结构**：确定 PPT 中页面类型（封面/目录/过渡/内容/结束）
2. **定义数据格式**：AI 生成结构化 PPT 数据（见 `src/types/AIPPT.ts`）
3. **制作模板**：在 PPTist 中制作模板页面并标注类型
4. **AI 生成数据**：调用 AI 接口生成符合结构的数据
5. **生成配图**：通过 AI 文生图或图库搜索匹配
6. **数据匹配**：将 AI 数据、配图与模板结合生成最终 PPT

**模板标记类型**：

| 页面类型 | 节点标记 |
|----------|----------|
| 封面页 | 标题、正文、图片 |
| 目录页 | 目录标题（列表项目）、图片 |
| 过渡页 | 标题、正文、节编号、图片 |
| 内容页 | 标题、内容项标题/正文/编号、图片 |
| 结束页 | 图片 |

---

## 3. 使用方法

### 3.1 环境要求

| 要求 | 版本 |
|------|------|
| Node.js | >= 20 |
| 包管理器 | npm |

### 3.2 安装与运行

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 类型检查
npm run type-check

# 代码检查
npm run lint

# 预览构建结果
npm run preview
```

开发服务器启动后访问：http://127.0.0.1:5173/

### 3.3 快捷键参考

#### 通用操作

| 快捷键 | 功能 |
|--------|------|
| Ctrl + X | 剪切 |
| Ctrl + C | 复制 |
| Ctrl + V | 粘贴 |
| Ctrl + Shift + V | 粘贴为纯文本 |
| Ctrl + D | 快速复制粘贴 |
| Ctrl + A | 全选 |
| Ctrl + Z | 撤销 |
| Ctrl + Y | 恢复 |
| Delete / Backspace | 删除 |
| Ctrl / Shift + 点击 | 多选 |
| Ctrl + F | 打开搜索替换 |
| Ctrl + P | 打印 |
| ESC | 关闭弹窗 |

#### 幻灯片放映

| 快捷键 | 功能 |
|--------|------|
| F5 | 从头开始放映 |
| Shift + F5 | 从当前页开始放映 |
| ↑ / ← / PgUp | 上一页 |
| ↓ / → / PgDown / Enter / Space | 下一页 |
| ESC | 退出放映 |

#### 幻灯片编辑

| 快捷键 | 功能 |
|--------|------|
| Enter | 新建幻灯片 |
| Space + 拖拽 | 移动画布 |
| Ctrl + 滚轮 | 缩放画布 |
| Ctrl + = | 放大画布 |
| Ctrl + - | 缩小画布 |
| Ctrl + 0 | 画布适应当前屏幕 |
| 双击空白处 / T | 快速创建文本 |
| R | 快速创建矩形 |
| O | 快速创建圆形 |
| L | 快速创建线条 |

#### 元素操作

| 快捷键 | 功能 |
|--------|------|
| 方向键 | 移动元素 |
| Ctrl + L | 锁定元素 |
| Ctrl + G | 组合元素 |
| Ctrl + Shift + G | 取消组合 |
| Alt + F | 置顶层 |
| Alt + B | 置底层 |
| Tab | 切换焦点元素 |

#### 文本编辑

| 快捷键 | 功能 |
|--------|------|
| Ctrl + B | 加粗 |
| Ctrl + I | 斜体 |
| Ctrl + U | 下划线 |
| Ctrl + E | 行内代码 |
| Ctrl + ; | 上角标 |
| Ctrl + ' | 下角标 |

### 3.4 导入导出操作

#### 导出格式

| 格式 | 说明 | 实现方式 |
|------|------|----------|
| PPTX | 导出为 PowerPoint 文件 | pptxgenjs，支持文字/图片/形状/线条/图表/表格/公式/音视频 |
| JSON | 导出为 JSON 数据文件 | 原始数据序列化 |
| .pptist | 加密的特有格式文件 | CryptoJS AES 加密后保存 |
| PNG/JPEG | 导出为图片 | html-to-image 逐页截图 |
| PDF | 导出为 PDF | 打印方式导出 |

#### 导入格式

| 格式 | 说明 | 实现方式 |
|------|------|----------|
| PPTX | 导入 PowerPoint 文件 | pptxtojson 解析，约 60% 还原度 |
| JSON | 导入 JSON 数据文件 | 直接反序列化 |
| .pptist | 导入特有格式文件 | CryptoJS AES 解密 |

#### 导出 PPTX 支持的元素属性

| 元素类型 | 支持导出的属性 |
|----------|----------------|
| 文字 | 位置/尺寸/旋转/内容/字体/颜色/填充/边框/阴影/行高/字间距/段间距/竖排/超链接/列表 |
| 图片 | 位置/尺寸/旋转/翻转/裁剪/圆角/透明度/超链接 |
| 形状 | 位置/尺寸/旋转/翻转/填充/渐变/边框/阴影/文字/超链接 |
| 线条 | 位置/尺寸/颜色/宽度/线型/端点样式/阴影 |
| 图表 | 位置/尺寸/类型/数据/主题色/文字颜色/网格颜色/堆积/平滑 |
| 表格 | 位置/尺寸/数据/主题/边框/合并单元格 |
| 公式 | 转为 SVG 图片导出 |
| 音视频 | 位置/尺寸/路径/封面（仅支持特定格式） |

### 3.5 自定义元素开发

添加新元素类型需完成以下步骤：

**步骤一：定义元素类型与数据结构**

在 `src/types/slides.ts` 中：
1. 在 `ElementTypes` 枚举中添加新类型
2. 定义元素接口，继承 `PPTBaseElement`
3. 将新类型加入 `PPTElement` 联合类型

**步骤二：添加元素配置**

在 `src/configs/element.ts` 中：
1. 在 `ELEMENT_TYPE_ZH` 中添加中文名
2. 在 `MIN_SIZE` 中添加最小尺寸

**步骤三：编写元素组件**

在 `src/views/components/element/` 下创建组件目录，需编写：
- 可编辑版组件（`index.vue`）—— 用于编辑器画布
- 基础版组件（`BaseXxxElement.vue`）—— 用于缩略图和放映

**步骤四：注册元素组件**

在以下文件中注册新元素组件：
- `src/views/components/ThumbnailSlide/ThumbnailElement.vue` —— 缩略图
- `src/views/Screen/ScreenElement.vue` —— 放映
- `src/views/Editor/Canvas/EditableElement.vue` —— 编辑器
- `src/views/Mobile/MobileEditor/MobileEditableElement.vue` —— 移动端

**步骤五：编写操作节点**

在 `src/views/Editor/Canvas/Operate/index.vue` 中注册操作节点组件

**步骤六：编写样式面板**

在 `src/views/Editor/Toolbar/ElementStylePanel/` 下创建面板组件，并在 `index.vue` 中注册

**步骤七：创建元素方法**

在 `src/hooks/useCreateElement.ts` 中添加创建方法，并在工具栏中调用

---

## 4. 技术架构

### 4.1 架构总览

```
┌─────────────────────────────────────────────────────────────┐
│                        Views（视图层）                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Editor   │  │  Screen  │  │  Mobile  │  │Components │   │
│  │  编辑器   │  │  播放器   │  │  移动端   │  │  公共组件  │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │
│       │              │              │              │         │
├───────┼──────────────┼──────────────┼──────────────┼─────────┤
│       │         Hooks（业务逻辑层）  │              │         │
│  ┌────┴─────────────────────────────┴──────────────┴─────┐  │
│  │  useExport  useImport  useAIPPT  useCreateElement ... │  │
│  └──────────────────────┬───────────────────────────────┘  │
│                         │                                    │
├─────────────────────────┼────────────────────────────────────┤
│                    Store（状态管理层）                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   Main   │  │  Slides  │  │ Snapshot │  │ Keyboard │   │
│  │  编辑器   │  │  幻灯片  │  │  历史快照  │  │  键盘状态  │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│                    基础设施层                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Types   │  │  Configs │  │  Utils   │  │ Services │   │
│  │  类型定义  │  │  配置文件  │  │  工具方法  │  │  API服务  │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└──────────────────────────────────────────────────────────────┘
```

### 4.2 技术栈选型

| 分类 | 技术 | 版本 | 选型理由 |
|------|------|------|----------|
| 框架 | Vue | 3.5.17 | Composition API + 响应式系统，适合复杂交互应用 |
| 语言 | TypeScript | ~5.3.0 | 类型安全，提升代码可维护性和开发体验 |
| 构建 | Vite | 5.3.5 | 快速 HMR，高效构建，原生 ESM 支持 |
| 状态管理 | Pinia | 3.0.2 | Vue 3 官方推荐，轻量、类型友好 |
| 富文本 | ProseMirror | 多包 | 高度可定制的富文本编辑框架 |
| 图表 | ECharts | 6.0.0 | 功能丰富的数据可视化库 |
| 公式 | hfmath | 0.0.2 | LaTeX 公式渲染为 SVG |
| PPTX导出 | pptxgenjs | 3.12.0 | 客户端生成 PowerPoint 文件 |
| PPTX导入 | pptxtojson | 2.0.0 | 将 PPTX 文件解析为 JSON 数据 |
| 图片导出 | html-to-image | 1.11.13 | DOM 节点转图片 |
| 本地存储 | Dexie | 4.0.11 | IndexedDB 封装，用于历史快照存储 |
| 动画 | Animate.css | 4.1.1 | CSS 动画库，用于元素动画效果 |
| 图标 | Iconify | - | 按需加载图标，支持 icon-park 图标集 |
| 样式 | SCSS | 1.69.6 | CSS 预处理器，变量/混入/嵌套 |
| HTTP | Axios | 1.7.9 | HTTP 请求库 |
| 加密 | CryptoJS | 4.2.0 | .pptist 文件加密/解密 |
| 拖拽 | vuedraggable | 4.1.0 | 基于 SortableJS 的 Vue 拖拽组件 |
| 提示 | Tippy.js | 6.3.7 | Tooltip/Popover 提示组件 |

### 4.3 目录结构

```
src/
├── assets/                     # 静态资源
│   ├── fonts/                  # 在线字体文件（28种字体 .woff2）
│   ├── icons/                  # 自定义 SVG 图标
│   └── styles/                 # 全局样式
│       ├── font.scss           # 在线字体定义
│       ├── global.scss         # 通用全局样式
│       ├── mixin.scss          # SCSS 全局混入
│       ├── variable.scss       # SCSS 全局变量
│       └── prosemirror.scss    # ProseMirror 富文本默认样式
├── components/                 # 与业务无关的通用组件
│   ├── ColorPicker/            # 颜色选择器（饱和度/色相/透明度）
│   ├── Contextmenu/            # 右键菜单
│   ├── LaTeXEditor/            # LaTeX 编辑器
│   ├── Button.vue              # 按钮
│   ├── Modal.vue               # 模态框
│   ├── Popover.vue             # 弹出层
│   ├── WritingBoard.vue        # 画板
│   └── ...                     # 其他 30+ 通用组件
├── configs/                    # 配置文件
│   ├── animation.ts            # 动画配置（入场/退场/强调/翻页）
│   ├── chart.ts                # 图表配置
│   ├── element.ts              # 元素配置（类型中文名/最小尺寸）
│   ├── font.ts                 # 字体配置
│   ├── hotkey.ts               # 快捷键配置
│   ├── imageClip.ts            # 图片裁剪形状配置
│   ├── latex.ts                # LaTeX 配置
│   ├── lines.ts                # 线条配置
│   ├── shapes.ts               # 形状配置（预置形状路径/公式）
│   ├── storage.ts              # 本地存储键名配置
│   └── theme.ts                # 主题配置
├── directive/                  # 自定义指令
│   ├── contextmenu.ts          # 右键菜单指令
│   ├── loading.ts              # 加载指令
│   └── tooltip.ts              # 提示指令
├── hooks/                      # 业务 Hooks（30+）
│   ├── useAIPPT.ts             # AI 生成 PPT
│   ├── useCreateElement.ts     # 创建元素
│   ├── useExport.ts            # 导出功能
│   ├── useImport.ts            # 导入功能
│   ├── useHistorySnapshot.ts   # 历史快照
│   ├── useScreening.ts         # 幻灯片放映
│   └── ...                     # 其他 hooks
├── services/                   # API 服务
│   ├── axios.ts                # Axios 实例
│   ├── fetch.ts                # Fetch 封装（支持流式）
│   └── index.ts                # API 方法定义
├── store/                      # Pinia 状态管理
│   ├── main.ts                 # 编辑器主状态
│   ├── slides.ts               # 幻灯片数据
│   ├── snapshot.ts             # 历史快照
│   ├── screen.ts               # 放映状态
│   └── keyboard.ts             # 键盘状态
├── types/                      # TypeScript 类型定义
│   ├── slides.ts               # 幻灯片核心类型
│   ├── AIPPT.ts                # AIPPT 类型
│   ├── edit.ts                 # 编辑器类型
│   ├── export.ts               # 导出类型
│   ├── mobile.ts               # 移动端类型
│   ├── toolbar.ts              # 工具栏类型
│   └── injectKey.ts            # Provide/Inject 键
├── utils/                      # 工具方法
│   ├── prosemirror/            # ProseMirror 配置
│   │   ├── commands/           # 自定义命令
│   │   ├── plugins/            # 自定义插件
│   │   └── schema/             # Schema 定义
│   ├── htmlParser/             # HTML 解析器
│   ├── clipboard.ts            # 剪贴板工具
│   ├── crypto.ts               # 加密解密
│   ├── database.ts             # IndexedDB 数据库
│   ├── element.ts              # 元素工具方法
│   ├── image.ts                # 图片处理
│   ├── svgPathParser.ts        # SVG 路径解析
│   └── ...                     # 其他工具
└── views/                      # 业务视图
    ├── Editor/                 # 编辑器模块
    │   ├── Canvas/             # 画布（核心）
    │   ├── CanvasTool/         # 画布工具栏
    │   ├── EditorHeader/       # 顶部菜单栏
    │   ├── ExportDialog/       # 导出对话框
    │   ├── Remark/             # 备注
    │   ├── Thumbnails/         # 缩略图导航
    │   ├── Toolbar/            # 右侧工具栏
    │   └── *.vue               # 面板组件
    ├── Screen/                 # 播放器模块
    │   ├── PresenterView.vue   # 演讲者视图
    │   ├── AudienceView.vue    # 观众视图
    │   └── hooks/              # 播放器 hooks
    ├── Mobile/                 # 移动端模块
    │   ├── MobileEditor/       # 移动端编辑器
    │   ├── MobilePlayer.vue    # 移动端播放器
    │   └── MobilePreview.vue   # 移动端预览
    └── components/             # 公共业务组件
        ├── ThumbnailSlide/     # 缩略图组件
        └── element/            # 元素组件（9种元素类型）
```

### 4.4 核心模块设计

#### 4.4.1 编辑器模块（Editor）

编辑器是项目核心模块，由以下区域组成：

```
┌──────────────────────────────────────────────┐
│              EditorHeader（顶部菜单栏）         │
├──────┬───────────────────────┬───────────────┤
│      │                       │               │
│  Th  │                       │               │
│  um  │      Canvas           │   Toolbar     │
│  bn  │      （画布）          │  （右侧工具栏） │
│  ai  │                       │               │
│  ls  │                       │               │
│      │                       │               │
│（缩  │                       │               │
│ 略图 │                       │               │
│ 导航）│                       │               │
│      ├───────────────────────┤               │
│      │   Remark（备注）       │               │
├──────┴───────────────────────┴───────────────┤
│          CanvasTool（插入/工具栏）              │
└──────────────────────────────────────────────┘
```

画布（Canvas）内部结构：

```
Canvas
├── ViewportBackground     # 可视区域背景
├── GridLines              # 网格线
├── Ruler                  # 标尺
├── EditableElement        # 可编辑元素（9种类型）
├── MouseSelection         # 鼠标选框
├── ElementCreateSelection # 元素创建选框
├── ShapeCreateCanvas      # 形状绘制画布
├── AlignmentLine          # 吸附对齐线
└── Operate                # 元素操作节点层
    ├── ResizeHandler      # 缩放控制点
    ├── RotateHandler      # 旋转控制点
    ├── BorderLine         # 边框线
    └── LinkHandler        # 超链接处理
```

#### 4.4.2 播放器模块（Screen）

播放器支持三种视图模式：

| 视图 | 组件 | 说明 |
|------|------|------|
| 演讲者视图 | PresenterView | 双屏模式，含备注、计时器、画笔工具 |
| 观众视图 | AudienceView | 仅展示幻灯片内容 |
| 基础视图 | BaseView | 单屏全屏放映 |

播放器核心 Hooks：

| Hook | 功能 |
|------|------|
| useExecPlay | 控制动画执行序列 |
| useFullscreen | 全屏控制 |
| useSlideSize | 幻灯片尺寸适配 |
| useSlidesWithTurningMode | 翻页动画处理 |

#### 4.4.3 元素系统

每种元素类型包含三个层级的组件：

| 层级 | 组件 | 用途 |
|------|------|------|
| 可编辑版 | `XxxElement/index.vue` | 编辑器画布中使用，支持选中/拖拽/缩放等交互 |
| 基础版 | `BaseXxxElement.vue` | 缩略图和放映模式，仅渲染展示 |
| 放映版 | `ScreenXxxElement.vue` | 放映模式专用（音视频元素） |

元素通用 Hooks：

| Hook | 功能 |
|------|------|
| useElementFill | 元素填充处理 |
| useElementFlip | 元素翻转处理 |
| useElementOutline | 元素边框处理 |
| useElementShadow | 元素阴影处理 |

### 4.5 数据模型设计

#### 4.5.1 核心数据结构

```
SlidesState
├── title: string                    # 幻灯片标题
├── theme: SlideTheme                # 主题样式
│   ├── backgroundColor: string      # 背景颜色
│   ├── themeColors: string[]        # 主题色数组
│   ├── fontColor: string            # 字体颜色
│   ├── fontName: string             # 字体名称
│   ├── outline: PPTElementOutline   # 默认边框
│   └── shadow: PPTElementShadow     # 默认阴影
├── slides: Slide[]                  # 幻灯片页面数组
│   └── Slide
│       ├── id: string               # 页面ID
│       ├── elements: PPTElement[]   # 元素集合
│       ├── notes?: Note[]           # 批注
│       ├── remark?: string          # 备注
│       ├── background?: SlideBackground  # 页面背景
│       ├── animations?: PPTAnimation[]   # 动画集合
│       ├── turningMode?: TurningMode     # 翻页方式
│       ├── sectionTag?: SectionTag       # 分节标记
│       └── type?: SlideType              # 页面类型
├── slideIndex: number               # 当前页面索引
├── viewportSize: number             # 画布宽度基数（默认1000）
├── viewportRatio: number            # 画布宽高比（默认0.5625即16:9）
└── templates: SlideTemplate[]       # 模板列表
```

#### 4.5.2 元素类型体系

```
PPTElement (联合类型)
├── PPTTextElement          # 文字元素
├── PPTImageElement         # 图片元素
├── PPTShapeElement         # 形状元素
├── PPTLineElement          # 线条元素
├── PPTChartElement         # 图表元素
├── PPTTableElement         # 表格元素
├── PPTLatexElement         # 公式元素
├── PPTVideoElement         # 视频元素
└── PPTAudioElement         # 音频元素

PPTBaseElement (基础属性)
├── id: string              # 元素ID
├── left: number            # 水平位置
├── top: number             # 垂直位置
├── width: number           # 宽度
├── height: number          # 高度
├── rotate: number          # 旋转角度
├── lock?: boolean          # 是否锁定
├── groupId?: string        # 组合ID
├── link?: PPTElementLink   # 超链接
└── name?: string           # 元素名称
```

#### 4.5.3 各元素特有属性

| 元素类型 | 特有属性 |
|----------|----------|
| Text | content, defaultFontName, defaultColor, lineHeight, wordSpace, paragraphSpace, fill, outline, shadow, opacity, vertical, textType |
| Image | src, fixedRatio, filters, clip, flipH, flipV, outline, shadow, radius, colorMask, imageType |
| Shape | viewBox, path, fixedRatio, fill, gradient, pattern, outline, opacity, flipH, flipV, shadow, special, text, pathFormula, keypoints |
| Line | start, end, style, color, points, shadow, broken, broken2, curve, cubic |
| Chart | chartType, data, options, fill, outline, themeColors, textColor, lineColor |
| Table | outline, theme, colWidths, cellMinHeight, data |
| Latex | latex, path, color, strokeWidth, viewBox, fixedRatio |
| Video | src, autoplay, poster, ext |
| Audio | src, fixedRatio, color, loop, autoplay, ext |

#### 4.5.4 辅助数据结构

| 结构 | 说明 |
|------|------|
| Gradient | 渐变（type: linear/radial, colors, rotate） |
| PPTElementOutline | 边框（style, width, color） |
| PPTElementShadow | 阴影（h, v, blur, color） |
| PPTElementLink | 超链接（type: web/slide, target） |
| SlideBackground | 背景（type: solid/image/gradient） |
| PPTAnimation | 动画（id, elId, effect, type, duration, trigger） |
| TableCell | 表格单元格（id, colspan, rowspan, text, style） |
| ChartData | 图表数据（labels, legends, series） |

### 4.6 状态管理设计

项目使用 Pinia 进行状态管理，共 5 个 Store：

#### MainStore（编辑器主状态）

| 状态 | 类型 | 说明 |
|------|------|------|
| activeElementIdList | string[] | 被选中的元素ID集合 |
| handleElementId | string | 正在操作的元素ID |
| activeGroupElementId | string | 组合中被选中可独立操作的元素ID |
| hiddenElementIdList | string[] | 被隐藏的元素ID集合 |
| canvasPercentage | number | 画布可视区域百分比（默认90） |
| canvasScale | number | 画布缩放比例 |
| canvasDragged | boolean | 画布是否被拖拽 |
| creatingElement | CreatingElement \| null | 正在插入的元素信息 |
| creatingCustomShape | boolean | 是否正在绘制任意多边形 |
| toolbarState | ToolbarStates | 右侧工具栏状态 |
| richTextAttrs | TextAttrs | 富文本编辑状态 |
| selectedTableCells | string[] | 选中的表格单元格 |
| textFormatPainter | TextFormatPainter \| null | 文字格式刷 |
| shapeFormatPainter | ShapeFormatPainter \| null | 形状格式刷 |
| showSelectPanel | boolean | 选择面板开关 |
| showSearchPanel | boolean | 搜索面板开关 |
| showAIPPTDialog | boolean \| 'running' | AIPPT 对话框开关 |

#### SlidesStore（幻灯片数据）

| 核心 Action | 说明 |
|-------------|------|
| setSlides(slides, themeProps?) | 设置全部幻灯片数据 |
| addSlide(slide) | 添加幻灯片页面 |
| updateSlide(props, slideId?) | 更新幻灯片属性 |
| deleteSlide(slideId) | 删除幻灯片页面 |
| updateSlideIndex(index) | 切换当前页面 |
| addElement(element) | 添加元素 |
| updateElement(data) | 更新元素属性 |
| deleteElement(elementId) | 删除元素 |
| removeElementProps(data) | 移除元素属性 |
| setTheme(themeProps) | 设置主题 |
| setViewportRatio(ratio) | 设置画布比例 |

| 核心 Getter | 说明 |
|-------------|------|
| currentSlide | 当前页面数据 |
| currentSlideAnimations | 当前页有效动画列表 |
| formatedAnimations | 格式化的动画序列（合并同时/自动触发） |

#### SnapshotStore（历史快照）

| 状态 | 说明 |
|------|------|
| snapshotCursor | 快照指针位置（-1 为初始值） |
| snapshotLength | 快照总长度 |

| 核心 Action | 说明 |
|-------------|------|
| initSnapshotDatabase() | 初始化快照数据库 |
| addSnapshot() | 添加新快照 |
| unDo() | 撤销 |
| reDo() | 重做 |

| 核心 Getter | 说明 |
|-------------|------|
| canUndo | 是否可撤销 |
| canRedo | 是否可重做 |

### 4.7 画布与元素原理

#### 相对坐标系统

画布采用相对坐标系统，以 `viewportSize`（默认 1000）为宽度基数：

- 元素的 `left`、`top` 表示距离画布左上角的位置
- 元素的 `width`、`height` 表示元素尺寸
- 一个 `{ width: 1000, height: 562.5, left: 0, top: 0 }` 的元素恰好铺满整个画布

#### 缩放计算

实际渲染时根据画布容器大小计算缩放比：

```
scale = 实际容器宽度 / viewportSize
```

所有元素按此缩放比进行等比缩放，缩略图和放映页面本质上是不同大小的可视区域。

#### 元素定位

所有元素使用 `position: absolute` 定位，通过 `transform: rotate()` 实现旋转，通过 `transform: scale(-1, 1)` / `scale(1, -1)` 实现翻转。

### 4.8 历史快照机制

历史快照基于 IndexedDB（Dexie）实现：

1. **初始化**：应用启动时创建 IndexedDB 数据库，写入初始快照
2. **添加快照**：每次编辑操作后，将当前 slides 数据深拷贝后存入 IndexedDB
3. **快照限制**：最多保留 20 条快照，超出时删除最早的记录
4. **撤销**：指针前移，从 IndexedDB 读取对应快照恢复数据
5. **重做**：指针后移，从 IndexedDB 读取对应快照恢复数据
6. **分支处理**：撤销后进行新操作时，删除指针之后的所有快照
7. **数据库清理**：应用关闭时记录数据库 ID，下次启动时清理过期数据库（超过 12 小时）

数据库命名规则：`PPTist_{databaseId}_{timestamp}`

---

## 5. API 接口规范

### 5.1 服务端接口

服务端基础地址：

- 开发环境：`/api`（代理到 https://server.pptist.cn）
- 生产环境：`https://server.pptist.cn`

#### 5.1.1 图片搜索

```
POST /tools/img_search
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| query | string | 是 | 搜索关键词 |
| orientation | string | 否 | 方向：landscape/portrait/square/all |
| locale | string | 否 | 语言：zh/en |
| order | string | 否 | 排序：popular/latest |
| size | string | 否 | 尺寸：large/medium/small |
| image_type | string | 否 | 类型：all/photo/illustration/vector |
| page | number | 否 | 页码 |
| per_page | number | 否 | 每页数量 |

#### 5.1.2 AIPPT 大纲生成

```
POST /tools/aippt_outline
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| content | string | 是 | PPT 主题内容 |
| language | string | 是 | 语言 |
| model | string | 是 | AI 模型名称 |
| stream | boolean | - | 固定为 true（流式输出） |

返回：Server-Sent Events（SSE）流式数据

#### 5.1.3 AIPPT 内容生成

```
POST /tools/aippt
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| content | string | 是 | 大纲内容 |
| language | string | 是 | 语言 |
| style | string | 是 | 风格 |
| model | string | 是 | AI 模型名称 |
| stream | boolean | - | 固定为 true（流式输出） |

返回：Server-Sent Events（SSE）流式数据

#### 5.1.4 AI 写作辅助

```
POST /tools/ai_writing
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| content | string | 是 | 原始文本 |
| command | string | 是 | 指令（改写/扩写/缩写） |
| model | string | - | 固定为 GLM-4.5-Flash |
| stream | boolean | - | 固定为 true（流式输出） |

返回：Server-Sent Events（SSE）流式数据

#### 5.1.5 Mock 数据获取

```
GET ./mocks/{filename}.json
```

用于获取本地 Mock 数据，如幻灯片数据、模板数据、AIPPT 示例等。

### 5.2 Store Actions 接口

#### SlidesStore Actions

| 方法签名 | 说明 |
|----------|------|
| `setTitle(title: string)` | 设置幻灯片标题 |
| `setTheme(themeProps: Partial<SlideTheme>)` | 更新主题属性 |
| `setViewportSize(size: number)` | 设置画布宽度基数 |
| `setViewportRatio(ratio: number)` | 设置画布宽高比 |
| `setSlides(slides: Slide[], themeProps?)` | 设置全部幻灯片 |
| `addSlide(slide: Slide \| Slide[])` | 添加页面（插入到当前页之后） |
| `updateSlide(props: Partial<Slide>, slideId?)` | 更新页面属性 |
| `deleteSlide(slideId: string \| string[])` | 删除页面 |
| `updateSlideIndex(index: number)` | 切换当前页面索引 |
| `addElement(element: PPTElement \| PPTElement[])` | 添加元素到当前页 |
| `deleteElement(elementId: string \| string[])` | 删除元素 |
| `updateElement(data: UpdateElementData)` | 更新元素属性 |
| `removeElementProps(data: RemovePropData)` | 移除元素指定属性 |

#### MainStore Actions

| 方法签名 | 说明 |
|----------|------|
| `setActiveElementIdList(ids: string[])` | 设置选中元素 |
| `setCanvasScale(scale: number)` | 设置画布缩放 |
| `setCreatingElement(element: CreatingElement \| null)` | 设置正在创建的元素 |
| `setToolbarState(state: ToolbarStates)` | 切换工具栏面板 |
| `setRichtextAttrs(attrs: TextAttrs)` | 更新富文本状态 |
| `setTextFormatPainter(painter)` | 设置文字格式刷 |
| `setShapeFormatPainter(painter)` | 设置形状格式刷 |

#### SnapshotStore Actions

| 方法签名 | 说明 |
|----------|------|
| `initSnapshotDatabase()` | 初始化快照数据库 |
| `addSnapshot()` | 添加新快照 |
| `unDo()` | 撤销操作 |
| `reDo()` | 重做操作 |

### 5.3 Hooks 接口

| Hook | 返回值 | 说明 |
|------|--------|------|
| `useExport()` | `{ exporting, exportImage, exportImagePPTX, exportJSON, exportSpecificFile, exportPPTX }` | 导出功能 |
| `useImport()` | `{ exporting, importSpecificFile, importJSON, importPPTXFile }` | 导入功能 |
| `useAIPPT()` | `{ presetImgPool, AIPPT, getMdContent }` | AI 生成 PPT |
| `useHistorySnapshot()` | `{ addHistorySnapshot }` | 历史快照 |
| `useCreateElement()` | `{ createElement, createTextElement, createImageElement, createShapeElement, ... }` | 创建元素 |
| `useScreening()` | `{ enterScreening, exitScreening }` | 幻灯片放映 |
| `useSlideHandler()` | `{ isEmptySlide, copySlide, deleteSlide, ... }` | 页面操作 |
| `useSelectElement()` | `{ selectElement }` | 元素选择 |
| `useCopyAndPasteElement()` | `{ copyElement, pasteElement, ... }` | 复制粘贴 |
| `useDeleteElement()` | `{ deleteElement }` | 删除元素 |
| `useMoveElement()` | `{ dragElement }` | 元素移动 |
| `useAlignActiveElement()` | `{ alignActiveElement }` | 元素对齐 |
| `useCombineElement()` | `{ combineElements, uncombineElements }` | 组合/取消组合 |
| `useLockElement()` | `{ lockElement, unlockElement }` | 锁定/解锁 |
| `useOrderElement()` | `{ orderElement }` | 层级调整 |
| `useGlobalHotkey()` | - | 全局快捷键 |
| `useScaleCanvas()` | `{ scaleCanvas, resetCanvas }` | 画布缩放 |
| `useSearch()` | `{ searchKey, searchResults, ... }` | 查找替换 |
| `useSlideBackgroundStyle()` | `{ backgroundStyle }` | 背景样式计算 |
| `useSlideTheme()` | `{ applyTheme }` | 主题应用 |
| `useTextFormatPainter()` | `{ startTextFormatPainter }` | 文字格式刷 |
| `useShapeFormatPainter()` | `{ startShapeFormatPainter }` | 形状格式刷 |

### 5.4 类型定义规范

#### 元素类型枚举

```typescript
export const enum ElementTypes {
  TEXT = 'text',
  IMAGE = 'image',
  SHAPE = 'shape',
  LINE = 'line',
  CHART = 'chart',
  TABLE = 'table',
  LATEX = 'latex',
  VIDEO = 'video',
  AUDIO = 'audio',
}
```

#### 动画类型

```typescript
export type AnimationType = 'in' | 'out' | 'attention'
export type AnimationTrigger = 'click' | 'meantime' | 'auto'
```

#### 翻页方式

```typescript
export type TurningMode = 'no' | 'fade' | 'slideX' | 'slideY' | 'random' |
  'slideX3D' | 'slideY3D' | 'rotate' | 'scaleY' | 'scaleX' | 'scale' | 'scaleReverse'
```

#### 页面类型

```typescript
export type SlideType = 'cover' | 'contents' | 'transition' | 'content' | 'end'
```

#### AIPPT 数据类型

```typescript
export type AIPPTSlide = AIPPTCover | AIPPTContents | AIPPTTransition | AIPPTContent | AIPPTEnd
```

新增元素类型时，需同步更新 `ElementTypes` 枚举、`PPTElement` 联合类型、`ELEMENT_TYPE_ZH` 配置和 `MIN_SIZE` 配置。

---

## 6. 注意事项

### 6.1 项目定位与适用场景

| 场景 | 推荐度 | 说明 |
|------|--------|------|
| Web 幻灯片编辑/演示应用 | ⭐⭐⭐⭐⭐ | 最推荐场景，编辑能力和体验是核心优势 |
| AIPPT 生成工具 | ⭐⭐ | 提供基础模板式 AIPPT，需自行实现/优化生成流程 |
| Office PPT 制作工具 | ⭐⭐ | 导出无法 100% 还原，导入能力有限 |
| PPTX 文件预览工具 | ⭐ | 导入还原度约 60%，仅适合基本内容展示 |
| 低代码/H5/图片编辑器/白板 | 不推荐 | 建议选择匹配度更高的开源项目 |

### 6.2 功能边界与限制

| 限制项 | 说明 |
|--------|------|
| PPTX 导入还原度 | 约 60%，复杂形状/动画/排版可能丢失 |
| PPTX 导出还原度 | 部分属性无法完全还原（如渐变文字、图案填充形状导出为图片） |
| 历史快照上限 | 最多保留 20 步操作记录 |
| 数据库生命周期 | IndexedDB 数据库超过 12 小时自动清理 |
| 服务端依赖 | 图片搜索、AIPPT、AI 写作依赖外部服务，需自行部署或使用官方服务 |
| 移动端功能 | 仅支持基础编辑和预览，功能远少于桌面端 |
| 协同编辑 | 不支持多人实时协同编辑 |
| 后端能力 | 纯前端项目，不提供后端服务，需自行接入 |

### 6.3 开源协议与商用

- 项目采用 **AGPL-3.0 协议**开源
- **禁止闭源商用**，商用需严格遵循 AGPL-3.0 协议
- 无法执行 AGPL-3.0 时可选择：
  1. 使用早期 Apache 2.0 协议版本（2022年5月后停止维护）
  2. 成为项目重要贡献者
  3. 付费获取独立商业授权（一年 2999 元 / 永久 5499 元）
- 详见项目 [LICENSE](/LICENSE) 文件

### 6.4 性能优化建议

| 优化方向 | 建议 |
|----------|------|
| 元素渲染 | 避免单页过多元素（建议 < 50 个），大量元素时考虑虚拟滚动 |
| 图片资源 | 使用 WebP 格式，控制图片尺寸，避免加载超大图片 |
| 快照存储 | 快照采用深拷贝，大量页面时注意内存占用 |
| 动画性能 | 减少同时执行的动画数量，使用 CSS transform 代替 top/left 动画 |
| 字体加载 | 字体文件采用 woff2 格式，按需加载 |
| 构建优化 | 使用 Vite 代码分割，懒加载非首屏组件 |

### 6.5 常见问题

| 问题 | 解决方案 |
|------|----------|
| Node 版本不兼容 | 确保 Node.js >= 20 |
| 安装依赖失败 | 删除 node_modules 和 package-lock.json 后重新安装 |
| 画布显示异常 | 检查 viewportSize 和 viewportRatio 配置 |
| 导出 PPTX 样式偏差 | 部分属性无法完全还原，属于已知限制 |
| 导入 PPTX 内容丢失 | pptxtojson 还原度约 60%，复杂元素可能无法解析 |
| AIPPT 功能不可用 | 需要配置后端服务，开发环境通过 Vite 代理访问 |
| 字体不显示 | 确保字体文件已正确加载到 assets/fonts 目录 |
| 快照数据丢失 | IndexedDB 数据在浏览器清理时可能丢失，建议定期导出 |

### 6.6 部署说明

#### 构建生产版本

```bash
npm run build
```

构建产物输出到 `dist/` 目录。

#### Nginx 部署

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /path/to/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # API 代理（如需后端服务）
    location /api/ {
        proxy_pass https://server.pptist.cn/;
        proxy_set_header Host $host;
    }
}
```

#### GitHub Pages 部署

项目已内置 GitHub Actions 部署配置（`.github/workflows/deploy.yml`），推送到主分支后自动部署。

#### Docker 部署

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

#### 环境变量

| 变量 | 说明 | 默认值 |
|------|------|--------|
| VITE_SERVER_URL | API 服务地址 | 开发: /api, 生产: https://server.pptist.cn |

---

> 本文档由 GLM-5.1 模型自动生成，内容基于 PPTist v2.0.0 源码分析。如有疑问请参考项目 [Issues](https://github.com/pipipi-pikachu/PPTist/issues) 或现有文档。
