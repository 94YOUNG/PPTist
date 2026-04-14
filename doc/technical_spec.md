# PPTist 技术文档

## 目录

1. [项目概述](#1-项目概述)
2. [技术架构](#2-技术架构)
3. [核心模块设计](#3-核心模块设计)
4. [数据模型设计](#4-数据模型设计)
5. [接口设计](#5-接口设计)
6. [开发环境配置](#6-开发环境配置)
7. [部署流程](#7-部署流程)
8. [性能优化策略](#8-性能优化策略)
9. [常见问题与解决方案](#9-常见问题与解决方案)

---

## 1. 项目概述

### 1.1 项目定位

PPTist 是一个基于 Web 的在线演示文稿（幻灯片）应用，旨在为开发者提供一个可二次开发的 Web 幻灯片编辑/演示解决方案。项目还原了大部分 Microsoft Office PowerPoint 的常用功能，支持多种元素类型的编辑和演示。

### 1.2 核心特性

- **技术栈简洁**：基于 Vue 3 + TypeScript 构建，不依赖 UI 组件库
- **功能丰富**：支持文字、图片、形状、线条、图表、表格、视频、音频、公式等元素类型
- **导出多样**：支持导出 PPTX、JSON、图片、PDF 等多种格式
- **AI 辅助**：内置 AI 生成 PPT 功能
- **移动端支持**：提供移动端基础编辑和预览能力

### 1.3 项目目标用户

本项目主要面向有 Web 幻灯片开发需求的开发者，需要具备基础的 Web 开发经验。用户可以在此基础上定制个性化功能，打造不同于传统 Office PPT 的演示类产品。

---

## 2. 技术架构

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                         应用层 (Views)                           │
├─────────────────┬─────────────────────┬─────────────────────────┤
│   编辑器模块     │     播放器模块      │      移动端模块         │
│   (Editor)      │     (Screen)       │      (Mobile)          │
└────────┬────────┴──────────┬──────────┴────────────┬────────────┘
         │                  │                      │
         ▼                  ▼                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                      业务逻辑层 (Hooks)                         │
│  useCreateElement | useHistorySnapshot | useExport | ...      │
└─────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                       状态管理层 (Pinia)                        │
│  useMainStore | useSlidesStore | useScreenStore | ...         │
└─────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                        数据类型层 (Types)                       │
│  PPTElement | Slide | SlideTheme | PPTAnimation | ...         │
└─────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────┐
│                      工具层 (Utils)                             │
│  htmlParser | prosemirror | clipboard | image | ...            │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 技术栈选型

#### 2.2.1 前端框架

| 技术 | 版本 | 说明 |
|------|------|------|
| Vue | ^3.5.17 | 渐进式前端框架，组件化开发 |
| TypeScript | ~5.3.0 | 类型安全，提高代码质量 |
| Pinia | ^3.0.2 | 状态管理（替代 Vuex） |
| Vite | ^5.3.5 | 构建工具，开发体验优秀 |

#### 2.2.2 富文本编辑

| 技术 | 版本 | 说明 |
|------|------|------|
| prosemirror-state | ^1.4.3 | ProseMirror 状态管理 |
| prosemirror-view | ^1.33.9 | ProseMirror 视图层 |
| prosemirror-model | ^1.22.2 | ProseMirror 数据模型 |
| prosemirror-commands | ^1.6.0 | ProseMirror 命令集 |
| prosemirror-history | ^1.3.2 | 历史记录（撤销/重做） |
| prosemirror-keymap | ^1.2.2 | 键盘映射 |
| prosemirror-inputrules | ^1.4.0 | 输入规则 |
| prosemirror-schema-basic | ^1.2.3 | 基础 schema |
| prosemirror-schema-list | ^1.4.1 | 列表 schema |

#### 2.2.3 图表与公式

| 技术 | 版本 | 说明 |
|------|------|------|
| echarts | ^6.0.0 | 图表渲染 |
| hfmath | ^0.0.2 | LaTeX 公式渲染 |

#### 2.2.4 文件处理

| 技术 | 版本 | 说明 |
|------|------|------|
| pptxgenjs | ^3.12.0 | 生成 PPTX 文件 |
| pptxtojson | ^2.0.0 | 解析 PPTX 文件 |
| html-to-image | ^1.11.13 | HTML 转图片 |
| file-saver | ^2.0.5 | 文件下载 |
| jsonrepair | ^3.13.2 | JSON 修复 |

#### 2.2.5 其他依赖

| 技术 | 版本 | 说明 |
|------|------|------|
| axios | ^1.7.9 | HTTP 请求 |
| dexie | ^4.0.11 | IndexedDB 封装 |
| animate.css | ^4.1.1 | 动画库 |
| svg-pathdata | ^7.1.0 | SVG 路径处理 |
| nanoid | ^5.0.7 | ID 生成 |
| lodash | ^4.17.21 | 工具库 |
| tinycolor2 | ^1.6.0 | 颜色处理 |
| crypto-js | ^4.2.0 | 加密解密 |
| vuedraggable | ^4.1.0 | 拖拽排序 |

### 2.3 技术选型理由

#### 2.3.1 Vue 3 + TypeScript

- **Vue 3** 提供了 Composition API，更好地支持逻辑复用和代码组织
- **TypeScript** 提供静态类型检查，减少运行时错误，提高代码可维护性
- 两者结合使大型项目管理更加清晰

#### 2.3.2 不依赖 UI 组件库

项目刻意避免使用 UI 组件库（如 Element Plus、Ant Design），原因如下：

- **样式定制更轻松**：可完全自定义组件样式，无需覆盖第三方样式
- **功能扩展更方便**：不受第三方组件功能限制
- **减少依赖体积**：按需引入，减少打包体积
- **深度控制力**：对组件行为有完全控制权

#### 2.3.3 ProseMirror

- 专业级的富文本编辑器框架
- 高度可定制，支持复杂编辑场景
- 完善的撤销/重做机制
- 社区活跃，文档完善

#### 2.3.4 ECharts

- 图表类型丰富，满足各种数据可视化需求
- 性能优秀，支持大数据量渲染
- 配置灵活，支持深度定制

---

## 3. 核心模块设计

### 3.1 项目目录结构

```
PPTist/
├── src/
│   ├── assets/                    # 静态资源
│   │   ├── fonts/                 # 在线字体文件
│   │   └── styles/               # 全局样式
│   │
│   ├── components/                # 通用 UI 组件
│   │   ├── ColorPicker/          # 颜色选择器
│   │   ├── Contextmenu/           # 右键菜单
│   │   ├── LaTeXEditor/          # LaTeX 编辑器
│   │   └── ...
│   │
│   ├── configs/                   # 配置文件
│   │   ├── animation.ts           # 动画配置
│   │   ├── chart.ts               # 图表配置
│   │   ├── element.ts             # 元素配置
│   │   ├── font.ts                # 字体配置
│   │   ├── hotkey.ts              # 快捷键配置
│   │   ├── shapes.ts              # 预置形状
│   │   └── theme.ts               # 主题配置
│   │
│   ├── hooks/                     # 组合式函数
│   │   ├── useCreateElement.ts    # 创建元素
│   │   ├── useHistorySnapshot.ts  # 历史记录
│   │   ├── useExport.ts           # 导出功能
│   │   ├── useImport.ts           # 导入功能
│   │   └── ...
│   │
│   ├── store/                     # Pinia 状态管理
│   │   ├── main.ts               # 主状态（选中等）
│   │   ├── slides.ts             # 幻灯片数据
│   │   ├── snapshot.ts           # 快照管理
│   │   ├── screen.ts             # 放映状态
│   │   └── keyboard.ts           # 键盘状态
│   │
│   ├── types/                     # TypeScript 类型定义
│   │   └── slides.ts             # 幻灯片相关类型
│   │
│   ├── utils/                     # 工具函数
│   │   ├── htmlParser/           # HTML 解析
│   │   ├── prosemirror/          # 富文本编辑器
│   │   ├── clipboard.ts          # 剪贴板
│   │   ├── image.ts              # 图片处理
│   │   └── ...
│   │
│   ├── views/                     # 业务视图
│   │   ├── Editor/               # 编辑器
│   │   │   ├── Canvas/          # 画布
│   │   │   ├── CanvasTool/       # 插入工具
│   │   │   ├── EditorHeader/     # 顶部菜单
│   │   │   ├── ExportDialog/     # 导出对话框
│   │   │   ├── Thumbnails/       # 缩略图
│   │   │   ├── Toolbar/           # 工具栏
│   │   │   └── ...
│   │   │
│   │   ├── Screen/               # 播放器
│   │   │   ├── AudienceView.vue  # 观众视图
│   │   │   ├── PresenterView.vue # 演讲者视图
│   │   │   ├── ScreenSlide.vue   # 放映页面
│   │   │   └── ...
│   │   │
│   │   ├── Mobile/               # 移动端
│   │   │   ├── MobileEditor/     # 移动端编辑器
│   │   │   ├── MobilePreview.vue # 移动端预览
│   │   │   └── ...
│   │   │
│   │   └── components/           # 业务组件
│   │       ├── ThumbnailSlide/   # 缩略图组件
│   │       └── element/          # 元素组件
│   │           ├── TextElement/  # 文本元素
│   │           ├── ImageElement/ # 图片元素
│   │           ├── ShapeElement/ # 形状元素
│   │           ├── ChartElement/ # 图表元素
│   │           └── ...
│   │
│   ├── services/                  # 网络服务
│   │   ├── axios.ts              # Axios 配置
│   │   └── fetch.ts              # Fetch 封装
│   │
│   ├── directive/                 # Vue 指令
│   │   ├── clickOutside.ts       # 点击外部
│   │   ├── contextmenu.ts        # 右键菜单
│   │   └── loading.ts            # 加载指令
│   │
│   ├── main.ts                   # 入口文件
│   └── App.vue                   # 根组件
│
├── public/                        # 公共资源
│   ├── mocks/                    # 模拟数据
│   └── imgs/                    # 图片资源
│
├── doc/                          # 项目文档
├── package.json                  # 依赖配置
├── vite.config.ts               # Vite 配置
└── tsconfig.json                # TypeScript 配置
```

### 3.2 编辑器模块

编辑器是 PPTist 的核心模块，负责幻灯片的编辑功能。

#### 3.2.1 编辑器结构

```
编辑器
├── 顶部菜单栏 (EditorHeader)
│   ├── 文件操作（新建、打开、保存）
│   ├── 导出功能
│   ├── 快捷键文档
│   └── 主题设置
│
├── 左侧导航栏 (Thumbnails)
│   ├── 幻灯片缩略图列表
│   ├── 幻灯片模板
│   └── 幻灯片分节管理
│
├── 中上部插入/工具栏 (CanvasTool)
│   ├── 插入元素工具
│   ├── 形状库
│   ├── 图表类型选择
│   └── 表格生成器
│
├── 画布 (Canvas)
│   ├── 可视区域 (Viewport)
│   │   ├── 可编辑元素
│   │   └── 鼠标选框
│   │
│   └── 画布工具
│       ├── 参考线
│       ├── 标尺
│       ├── 元素操作节点
│       └── 吸附对齐线
│
├── 右侧工具栏 (Toolbar)
│   ├── 元素样式面板
│   ├── 幻灯片设计面板
│   ├── 动画面板
│   └── 选择面板
│
└── 底部演讲者备注 (Remark)
```

#### 3.2.2 画布原理

画布采用相对坐标系设计：

- **可视区域基准**：默认以 宽1000px × 高562.5px 为基础比例（16:9）
- **缩放机制**：根据可视区域实际宽度计算缩放比例
- **统一坐标**：所有元素坐标都基于基准尺寸，无论画布实际大小如何

```typescript
// 缩放比例计算示例
const scale = viewportWidth / VIEWPORT_SIZE  // 1200 / 1000 = 1.2
// 元素实际渲染尺寸 = 元素基准尺寸 × 缩放比例
```

### 3.3 播放器模块

播放器负责幻灯片的演示功能。

#### 3.3.1 播放模式

| 模式 | 说明 |
|------|------|
| 演讲者视图 | 演讲者可见当前页、下一页、备注、计时器 |
| 观众视图 | 观众可见幻灯片内容 |
| 自动放映 | 定时自动切换幻灯片 |

#### 3.3.2 演示工具

- **画笔标注**：画笔、形状、箭头、荧光笔
- **橡皮擦除**：清除标注
- **黑板模式**：模拟黑板效果
- **激光笔**：指针指示
- **计时器**：演讲计时

### 3.4 状态管理

#### 3.4.1 MainStore

管理编辑器的主状态：

```typescript
interface MainState {
  handleElementId: string | null      // 当前操作的元素 ID
  handleSlideId: string | null       // 当前操作的幻灯片 ID
  selectedElementIds: string[]        // 选中的元素 ID 列表
  clipboard: PPTElement[]             // 剪贴板数据
  isScaling: boolean                  // 是否正在缩放
  // ...其他状态
}
```

#### 3.4.2 SlidesStore

管理幻灯片数据：

```typescript
interface SlidesState {
  title: string                       // 幻灯片标题
  slides: Slide[]                     // 幻灯片列表
  theme: SlideTheme                   // 主题配置
  viewportSize: number                // 可视区域基准宽度
  viewportRatio: number               // 宽高比
  templates: SlideTemplate[]         // 模板列表
}
```

#### 3.4.3 SnapshotStore

管理历史记录快照：

```typescript
interface SnapshotState {
  snapshots: Snapshot[]               // 快照列表
  currentIndex: number                 // 当前快照索引
}
```

### 3.5 元素系统

#### 3.5.1 支持的元素类型

| 类型 | 说明 | 核心属性 |
|------|------|----------|
| text | 文本 | content, fontSize, color |
| image | 图片 | src, filters, clip |
| shape | 形状 | shapeType, fill, stroke |
| line | 线条 | points, style |
| chart | 图表 | chartType, data, options |
| table | 表格 | rows, cols, cells |
| latex | 公式 | latex, color |
| video | 视频 | src, poster |
| audio | 音频 | src, autoplay |

#### 3.5.2 元素组成结构

每个元素由以下部分组成：

```
元素
├── 基础属性 (PPTBaseElement)
│   ├── id: string
│   ├── left: number
│   ├── top: number
│   ├── width: number
│   ├── height: number
│   ├── rotate: number
│   ├── lock: boolean
│   └── groupId: string
│
├── 样式属性
│   ├── fill: string                  // 填充色
│   ├── outline: PPTElementOutline    // 边框
│   ├── shadow: PPTElementShadow     // 阴影
│   └── opacity: number              // 透明度
│
└── 特殊属性（根据元素类型不同）
```

---

## 4. 数据模型设计

### 4.1 核心数据类型

#### 4.1.1 幻灯片页面 (Slide)

```typescript
interface Slide {
  id: string                          // 唯一标识
  type: SlideType                     // 页面类型
  elements: PPTElement[]              // 元素列表
  background: SlideBackground         // 背景
  notes: Note                         // 演讲者备注
  animation: PPTAnimation[]           // 页面动画
  turningMode: TurningMode            // 翻页方式
  sectionTag?: SectionTag             // 章节标签
}
```

#### 4.1.2 主题配置 (SlideTheme)

```typescript
interface SlideTheme {
  background: SlideBackground          // 背景色
  themeColor: string                   // 主题色
  fontColor: string                    // 字体颜色
  fontName: string                     // 字体
  fontSize: number                     // 基础字号
}
```

#### 4.1.3 元素动画 (PPTAnimation)

```typescript
interface PPTAnimation {
  type: AnimationType                  // 动画类型（入场/退场/强调）
  animationType: string                 // 具体动画效果
  trigger: AnimationTrigger            // 触发方式
  duration: number                     // 持续时间
  delay: number                        // 延迟时间
}
```

### 4.2 数据流转

```
用户操作
    │
    ▼
┌─────────────────┐
│   Hook 处理      │  useCreateElement / useDeleteElement / ...
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Store 更新    │  slidesStore.updateElement / ...
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   组件响应      │  Vue 响应式系统触发重新渲染
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   历史记录      │  snapshotStore.addSnapshot
└─────────────────┘
```

---

## 5. 接口设计

### 5.1 内部 API 设计原则

项目主要使用内部方法调用，遵循以下原则：

#### 5.1.1 Hook 设计模式

```typescript
// 创建元素的 Hook 示例
const { createTextElement } = useCreateElement()

// 导出功能的 Hook 示例
const { exportPPTX, exportImage, exportPDF } = useExport()
```

#### 5.1.2 Store 方法

```typescript
// 更新元素
slidesStore.updateElement(id: string, props: Partial<PPTElement>)

// 添加元素
slidesStore.addElement(element: PPTElement)

// 删除元素
slidesStore.deleteElement(id: string)

// 撤销/重做
snapshotStore.undo()
snapshotStore.redo()
```

### 5.2 外部 API（可选）

如需集成后端，可通过以下方式扩展：

#### 5.2.1 文件保存 API

```typescript
interface SaveAPI {
  save(data: SlideData): Promise<Result>
  load(id: string): Promise<SlideData>
  list(): Promise<SlideMeta[]>
}
```

#### 5.2.2 AI 生成 API

```typescript
interface AIPPTAPI {
  generateOutline(topic: string): Promise<AIPPTOutline>
  generateSlide(outline: AIPPTOutline): Promise<SlideData>
}
```

---

## 6. 开发环境配置

### 6.1 环境要求

| 环境 | 要求 |
|------|------|
| Node.js | >= 20.0.0 |
| npm | >= 9.0.0 |
| 浏览器 | Chrome 90+ / Firefox 90+ / Safari 15+ / Edge 90+ |

### 6.2 本地开发

#### 6.2.1 安装依赖

```bash
npm install
```

#### 6.2.2 启动开发服务器

```bash
npm run dev
```

访问地址：`http://127.0.0.1:5173/`

### 6.3 构建生产版本

```bash
npm run build
```

构建产物输出到 `dist` 目录。

### 6.4 代码检查

```bash
# 类型检查
npm run type-check

# 代码规范检查
npm run lint

# 自动修复
npm run lint -- --fix
```

### 6.5 IDE 配置

项目已配置 VS Code 推荐扩展，详见 `.vscode/extensions.json`。

推荐安装的扩展：
- Vue - Official
- TypeScript Vue Plugin (Volar)
- ESLint
- Prettier

---

## 7. 部署流程

### 7.1 静态部署

PPTist 作为单页应用（SPA），可部署到任何静态文件服务器。

#### 7.1.1 构建

```bash
npm run build
```

#### 7.1.2 部署方式

**方式一：Nginx**

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /var/www/pptist/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

**方式二：GitHub Pages**

```bash
# 构建后推送到 gh-pages 分支
npm run build
cd dist
git init
git add .
git commit -m "Deploy"
git push origin main:gh-pages
```

**方式三：Vercel / Netlify**

连接 GitHub 仓库后自动部署。

### 7.2 Docker 部署（可选）

```dockerfile
FROM nginx:alpine
COPY dist/ /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## 8. 性能优化策略

### 8.1 渲染优化

#### 8.1.1 虚拟列表

对于大量元素的幻灯片，使用虚拟列表技术，只渲染可视区域内的元素。

#### 8.1.2 懒加载

- 图片懒加载：非可视区域的图片延迟加载
- 组件懒加载：按需加载非核心组件

```typescript
// 示例：路由懒加载
const Editor = () => import('./views/Editor/index.vue')
```

### 8.2 状态优化

#### 8.2.1 细粒度响应

避免在 Store 中存储大量不必要的数据，只保存必要状态。

#### 8.2.2 计算属性缓存

使用 `computed` 缓存复杂计算结果，避免重复计算。

### 8.3 构建优化

#### 8.3.1 代码分割

利用 Vite 的自动代码分割，按路由和组件分割代码块。

#### 8.3.2 依赖优化

- 避免引入完整库，按需引入
- 使用替代方案（如 lodash-es 替代 lodash）

### 8.4 运行时优化

#### 8.4.1 事件委托

对于大量相似元素，使用事件委托减少事件监听器数量。

#### 8.4.2 防抖/节流

对于频繁触发的事件（如 resize、scroll），使用防抖或节流。

---

## 9. 常见问题与解决方案

### 9.1 开发相关

#### Q1: 如何添加新的元素类型？

**解决方案**：
1. 在 `types/slides.ts` 中定义新的元素接口
2. 在 `configs/element.ts` 中添加元素配置
3. 创建元素组件（编辑版和基础版）
4. 在 `EditableElement.vue` 中注册组件
5. 创建对应的样式面板
6. 在 `useCreateElement.ts` 中添加创建方法

详见 [自定义元素文档](./CustomElement.md)

#### Q2: 如何修改默认画布尺寸？

**解决方案**：
修改 `src/store/slides.ts` 中的 `viewportSize` 和 `viewportRatio`：

```typescript
state: (): SlidesState => ({
  viewportSize: 1000,      // 修改基准宽度
  viewportRatio: 16 / 9,   // 修改宽高比
  // ...
})
```

#### Q3: 如何添加新的导出格式？

**解决方案**：
1. 在 `hooks/useExport.ts` 中添加新的导出方法
2. 引入对应的库（如 pdf-lib 用于 PDF）
3. 实现导出逻辑

### 9.2 功能相关

#### Q4: 导入 PPTX 文件不完整？

**解决方案**：
项目使用 `pptxtojson` 解析 PPTX，由于格式复杂性，解析能力有限（约 60% 还原度）。建议：
- 重点关注核心内容（文本、图片、形状）
- 复杂动画和特殊效果可能丢失
- 可参考 `pptxtojson` 项目自行扩展

#### Q5: 移动端功能受限？

**解决方案**：
项目提供了移动端基础编辑能力，但功能相比桌面端有限：
- 支持基础元素（文本、图片、矩形、圆形）
- 支持基础操作（移动、缩放、旋转、复制、删除）
- 不支持复杂样式编辑

### 9.3 部署相关

#### Q6: 部署后刷新 404？

**解决方案**：
SPA 应用需要服务端配置 URL 重定向到 index.html：

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

#### Q7: 字体加载失败？

**解决方案**：
1. 检查字体文件路径是否正确
2. 检查 CORS 配置
3. 可将字体文件转换为 Base64 内联

---

## 附录

### A. 相关资源

- [项目在线演示](https://pipipi-pikachu.github.io/PPTist/)
- [PPTist GitHub](https://github.com/pipipi-pikachu/PPTist)
- [PPTist Gitee（国内镜像）](https://gitee.com/pptist/PPTist)

### B. 参考项目

- [pptxtojson](https://github.com/pipipi-pikachu/pptxtojson) - PPTX 解析
- [svgPathCreator](https://github.com/pipipi-pikachu/svgPathCreator) - SVG 形状绘制

### C. 开源协议

AGPL-3.0 License，详见 [LICENSE](../LICENSE)

---

*文档版本：2.0.0*
*最后更新：2026-04-11*
