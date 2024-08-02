# Vue 学习

# 一、简介

# 二、安装引入

# 三、工具

# 四、基础

```bash
yarn global add vite@latest
```


```bash
yarn create vite
```

```bash
cd web
yarn install
yarn dev
```

```bash
yarn install element-plus unplugin-vue-components unplugin-auto-import unplugin-icons @vitejs/plugin-vue-jsx
```

> vite.config.ts

```ts
import { defineConfig } from 'vite'
import { fileURLToPath, URL } from 'node:url'

import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'

// 使用unplugin-auto-import和unplugin-vue-components按需自动导入 vue、element plus、ts
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

// 自动导入Icon图标
import IconsResolver from 'unplugin-icons/resolver'
import Icons from 'unplugin-icons/vite'
import { FileSystemIconLoader } from 'unplugin-icons/loaders'

export default defineConfig({
   build: {
    emptyOutDir: true,
    chunkSizeWarningLimit: 600,
    cssCodeSplit: false,
    reportCompressedSize: true,
    terserOptions: {
        compress: {
          keep_infinity: true,
          // Used to delete console in production environment
          drop_console: true,
          drop_debugger: true,
        },
        output: {
          comments: true
        }
    },
    outDir: "../statics"
  },
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  },
  plugins: [
    vue(),
    vueJsx(),
    AutoImport({
      resolvers: [ElementPlusResolver()],
      imports: [
        "vue",
      ],
      // 如果使用 Typescript，需要设置 dts 为 true
      dts: true,
      // 如果使用了 eslint，需要设置 eslintrc 字段
      // 插件会在项目根目录生成类型文件 .eslintrc-auto-import.json ，确保该文件在eslint配置文件.eslintrc.cjs中被 extends
      eslintrc: {
        enabled: true
      }
    }),
    Components({
      resolvers: [
        ElementPlusResolver(),
        // 自动注册图标组件
        IconsResolver({
          // 修改Icon组件前缀，不设置则默认为i,禁用则设置为false
          prefix: 'icon',
          // 指定collection。ElementPlus图标集：ep
          enabledCollections: ['ep'],
          // 使用自定义的本地icon集合名
          customCollections: ['myIcons']
        })
      ]
    }),
    Icons({
      autoInstall: true,
      // 配置自定义的icon集合文件的路径
      customCollections: {
        'myIcons': FileSystemIconLoader('src/assets/icons'),
      }
    })
  ],
})

```
