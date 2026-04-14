# PPTist 项目规则

## 项目概述
PPTist 是一个基于 Vue 3 + TypeScript 的在线演示文稿编辑器，支持创建和编辑 PPT。

## 核心架构原则

### 1. 技术栈
- **框架**: Vue 3 (Composition API)
- **状态管理**: Pinia
- **语言**: TypeScript 5.3+
- **构建工具**: Vite 5
- **样式**: SCSS
- **富文本编辑**: ProseMirror

### 2. 目录结构约定
```
src/
├── components/     # 可复用组件
├── configs/        # 配置文件（形状、线条、图表等）
├── hooks/          # 组合式函数（业务逻辑）
├── store/          # Pinia 状态管理
├── types/          # TypeScript 类型定义
├── utils/          # 工具函数
├── views/          # 页面组件
│   ├── Editor/     # 编辑器主界面
│   └── Screen/     # 演示放映界面
└── services/       # API 服务
```

### 3. 命名规范
- **文件名**: 小写kebab-case (如: `use-create-element.ts`)
- **组件名**: PascalCase (如: `ColorPicker.vue`)
- **Hook函数**: use开头 (如: `useCreateElement`)
- **Store**: use开头 (如: `useMainStore`)
- **类型**: PascalCase，接口以I开头可选 (如: `PPTElement`, `MainState`)
- **常量**: UPPER_SNAKE_CASE (如: `ELEMENT_TYPES`)

### 4. 代码风格

#### TypeScript
- 优先使用 `interface` 定义类型，复杂类型使用 `type`
- 避免使用 `any`，必要时使用 `unknown`
- 使用 `const enum` 定义常量枚举
- 函数参数和返回值必须有类型注解

#### Vue 组件
- 使用 `<script setup lang="ts">` 语法
- Props 使用 `defineProps` with TypeScript
- Emits 使用 `defineEmits` with TypeScript
- 组件逻辑复杂时，提取到 hooks 中

#### 样式
- 使用 SCSS 预处理器
- 遵循 BEM 命名规范（可选）
- 响应式设计使用 mixin
- 颜色值使用变量定义

### 5. 状态管理
- 使用 Pinia 的 Composition API 风格
- Store 按功能模块划分（main, slides, snapshot, keyboard, screen）
- 复杂状态逻辑封装在 actions 中
- 使用 `storeToRefs` 解构响应式状态

### 6. Hooks 设计原则
- 单一职责：每个 hook 只负责一个功能
- 返回对象包含状态和方法
- 使用 TypeScript 泛型增强类型推导
- 复杂逻辑拆分为多个内部函数

### 7. 性能优化
- 大列表使用虚拟滚动
- 图片懒加载
- 防抖和节流处理高频事件
- 避免不必要的响应式数据
- 使用 `shallowRef` 和 `shallowReactive` 优化大数据

### 8. 错误处理
- 异步操作使用 try-catch
- 用户操作提供友好错误提示
- 边界条件检查
- 类型守卫确保类型安全

### 9. 测试要求
- 核心工具函数需要单元测试
- 复杂 hooks 需要测试
- E2E 测试覆盖关键用户流程

### 10. Git 提交规范
- 遵循 Conventional Commits
- 格式: `type(scope): subject`
- 类型: feat, fix, docs, style, refactor, test, chore
- 使用 Husky + Commitlint 自动校验

## 开发流程

### 启动项目
```bash
npm run dev          # 开发模式
npm run build        # 生产构建
npm run type-check   # 类型检查
npm run lint         # ESLint 检查
npm run lint:style   # Stylelint 检查
```

### 代码质量检查
- 提交前自动运行 lint-staged
- PR 前确保 type-check 通过
- 遵循 ESLint 和 Prettier 配置

## 关键技术点

### 1. 元素系统
- 支持多种元素类型：文本、图片、形状、线条、图表、表格、LaTeX、视频、音频
- 元素通用属性：位置、大小、旋转、锁定、组合、超链接
- 元素操作：拖拽、缩放、旋转、对齐、组合

### 2. 画布系统
- 视口缩放和平移
- 网格线和标尺
- 对齐吸附线
- 元素层级管理

### 3. 富文本编辑
- 基于 ProseMirror 实现
- 支持文本格式化、列表、对齐等
- 自定义 schema 和 commands

### 4. 历史记录
- 使用快照机制实现撤销/重做
- 优化快照存储策略

### 5. 导出功能
- 导出为 JSON、PPTX、PDF、图片
- 使用 pptxgenjs 生成 PPTX
- 使用 html-to-image 生成图片

## 注意事项

1. **性能敏感**: 画布操作频繁，注意性能优化
2. **类型安全**: 严格遵循 TypeScript 类型定义
3. **响应式陷阱**: 避免在循环中创建响应式数据
4. **内存管理**: 大文件和图片注意内存释放
5. **浏览器兼容**: 支持现代浏览器，不考虑 IE
