# Excalidraw画布

# 一、简介

Github: https://github.com/excalidraw/excalidraw

官方文档：https://docs.excalidraw.com/docs

# 二、私有化部署

excalidraw collaboration server：https://github.com/excalidraw/excalidraw-room

https://github.com/alswl/excalidraw-storage-backend

excalidraw http storage backend：https://github.com/alswl/excalidraw-storage-backend



改字体浏览器插件：https://chromewebstore.google.com/detail/excalidraw-custom-font/afbeaojffbjckicjpkecknoocdpmgoah



# 三、代码改造

## 1："工具栏"添加自定义功能

- 新增 icon：定义icon 的 svg：web/src/components/icons.tsx
- 新增标题定义：web/src/locales/zh-CN.json，web/src/locales/en.json
- 新增快捷键定义：web/src/keys.ts
- "工具栏"新增定义功能：web/src/components/Actions.tsx
- 注册新增功能的函数：web/src/components/App.tsx

## 2：“更多工具”添加自定义功能

- 新增 icon：定义icon 的 svg：web/src/components/icons.tsx
- 定义下拉菜单功能按钮功能（触发按钮弹窗的功能）及 icon：src/components/Actions.tsx
- 注册函数：web/src/components/App.tsx
- 新增弹窗函数类型定义
- 定义弹窗布局：web/src/components/DrawMarkdown.tsx
- 定义弹窗功能函数：web/src/components/DrawMarkdownToExcalidrawXmind.tsx
- 定义弹窗布局样式：web/src/components/DrawMarkdown.scss

## 3：“监听按键”自定义功能

- 键盘事件监听器函数：web/src/components/App.tsx 3062 onKeyDown()
- 键盘事件触发的功能函数：web/src/components/App.tsx 3006 handleXMindNodeShortcuts(){}
- 函数实现：web/src/components/DrawMarkdownToExcalidrawXmind.tsx

# 四、部署改造

## 1、纯Web

## 2、后端：协作+Libraries



参考：

- https://github.com/excalidraw/excalidraw
- https://github.com/excalidraw/excalidraw/discussions/3879
- https://blog.alswl.com/2022/10/self-hosted-excalidraw/
- https://github.com/alswl/excalidraw-storage-backend
- https://github.com/alswl/excalidraw