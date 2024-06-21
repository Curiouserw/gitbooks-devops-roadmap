# 一、Element Plus

文档：https://element-plus.org/zh-CN/guide/design.html

## 1、导入引用

### ①全量导入

`main.ts`

```bash
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import App from './App.vue'

const app = createApp(App)
  .use(ElementPlus)
  .mount('#app')
```

### ②按需导入

- **手动按需导入**

  ```ts
  // 按需导入 element-plus 组件，配置文件为 babel.config.js
  import { ElTable, ElTableColumn } from 'element-plus';
  // 引入 Element Plus 默认样式文件
  import 'element-plus/theme-chalk/index.css'
  ```

- **插件自动按需导入**

  安装`unplugin-vue-components` 和 `unplugin-auto-import`这两款插件

  > ```bash
  > npm install -D unplugin-vue-components unplugin-auto-import
  > ```
  
  `vite.config.ts`
  
  ```ts
  import { defineConfig } from 'vite'
  
  import AutoImport from 'unplugin-auto-import/vite'
  import Components from 'unplugin-vue-components/vite'
  import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'
  
  export default defineConfig({
    // ...
    plugins: [
      // ...
      Components({
        resolvers: [ElementPlusResolver()],
      }),
    ],
  }) 
  ```
  
# 二、Echarts

文档：https://echarts.apache.org/handbook/zh/get-started/

## 1、导入引用

### ①全量导入

```bash
import * as echarts from 'echarts'
```

### ②按需导入

```ts
// 引入 echarts 核心模块，核心模块提供了 echarts 使用必须的接口。
import * as echarts from 'echarts/core';

// 引入要使用的图表类型，图表后缀都为 Chart  
import { 
  PieChart, 
  BarChart 
} from 'echarts/charts';

// 引入提示框，标题，直角坐标系，数据集，内置数据转换器组件，组件后缀都为 Component
import {
  TitleComponent,
  TooltipComponent,
  GridComponent,
  DatasetComponent,
  TransformComponent
} from 'echarts/components';

// 引入 Canvas 渲染器，注意引入 CanvasRenderer 或者 SVGRenderer 是必须的一步
import { CanvasRenderer } from 'echarts/renderers';
// 注册必须的组件
echarts.use([
  PieChart, 
  BarChart, 
  TooltipComponent, 
  TitleComponent, 
  CanvasRenderer, 
  GridComponent
]);
```

使用跟之前一样，初始化图表，设置配置项。
