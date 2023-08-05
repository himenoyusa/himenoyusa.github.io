---
title: "pnpm monorepo 初体验"
url: pnpm-monorepo.html
id: pnpm-monorepo
index_img: /img/post/95186677_p3.jpg
categories:
  - 开发日志
date: 2023-08-04 21:00:00
tags:
---

![](/img/post/95186677_p3.jpg)

<!-- TOC -->

- [前情提要](#%E5%89%8D%E6%83%85%E6%8F%90%E8%A6%81)
- [技术选型](#%E6%8A%80%E6%9C%AF%E9%80%89%E5%9E%8B)
- [单个项目改造](#%E5%8D%95%E4%B8%AA%E9%A1%B9%E7%9B%AE%E6%94%B9%E9%80%A0)
- [pnpm monorepo 项目搭建](#pnpm-monorepo-%E9%A1%B9%E7%9B%AE%E6%90%AD%E5%BB%BA)
- [总结](#%E6%80%BB%E7%BB%93)
- [完](#%E5%AE%8C)

<!-- /TOC -->

### 前情提要

半年前，为了解决万恶的百度地图内存泄露的问题，组里的前端被我拆分成了两个独立的项目，地图作为单独的前端服务部署，主项目通过 iFrame 的方式引入地图服务，才算真正解决了地图内存泄露的问题

半年后，项目终于快要交付了，2.0 版本迭代也提上了日程，于是我想到，能不能把这两个项目再合起来，类似微服务一样来管理，也就是说终于要开始接触 monorepo 这玩意了

### 技术选型

先是技术选型，monorepo 目前好像也就 yarn 的 lerna、巨硬的 rush、pnpm 三种相对流行的解决方案

- rush  巨硬家的很多东西给人的感觉就是，玩具，玩腻了就丢，留下一堆坑给开发者，不考虑

- lerna  lerna 相对老一点，好像也没什么缺点

- pnpm  pnpm 给我的感觉就是，js 在走 java 的路了，node_modules 变成中央仓库了，从概念上来看好像不错，拿来玩玩

### 单个项目改造

要想从 yarn 切换到 pnpm，首先肯定要看看删除 yarn.lock 换成 pnpm install 之后，项目能不能正常跑起来

> 得益于项目的依赖版本还不算太老，pnpm install 完之后，run dev 成功没跑起来（

问题不大，报错一个一个解决，主要涉及到以下几点

1. **webpack-cli 报错** 这个还好，很容易找到问题，webpack-cli 版本低了要升级一下，我是从 `^4.8.0` 升级到了 `^4.10.0`

1. **部分幽灵依赖报错** 老生长谈了，把 lodash、zrender 等几个没安装就拿来用的依赖加上就好了

1. **webpack acron 别名配置** 忘记这个别名配置是用来解决什么问题的了，反正也是个幽灵依赖，加上就行了，注意版本号

### pnpm monorepo 项目搭建

pnpm 初始化一个 monorepo 项目很简单，不啰嗦，但是把两个独立的项目放在 monorepo 根目录下，作为子项目之后，问题就多了

1. **tsconfig 配置** ts 和 eslint 的配置本来就是老大难了，monorepo 想要把一些公共配置提升到根目录就更难了。主要是注意路径别名的配置、baseUrl 的设置等。看看下面的示例吧 

```json
// 根目录 tsconfig，
{
  "compilerOptions": {
    "jsx": "react",
    "esModuleInterop": true,
    "sourceMap": true,
    "resolveJsonModule": true,
    "target": "es5",
    "lib": ["es5", "es2015", "es2016", "dom", "es2015.promise"],
    "module": "ESNext",
    "moduleResolution": "node",
    "allowJs": true,
    "strict": true,
    "noImplicitAny": false,
    "downlevelIteration": true,
    "forceConsistentCasingInFileNames": true,
    "rootDir": "./",
    "baseUrl": "./",
    "outDir": "./dist",
  },

  "include": ["packages/*/src"]
}

/**
  * 子目录 tsconfig，先 extends，然后要用 baseUrl 指明当前文件所处的相对路径
  * 再配置路径别名，不然路径别名就是相对根目录的了
  * 注意子项目的配置一定要有 include 指明当前子项目路径，不然打包时 ts 会把根目录作为项目路径
  */
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "baseUrl": "./",
    "outDir": "./dist",
    "paths": {
      "@src/*": [
        "./src/*"
      ]
    }
  },

  "include": ["./src", "./postcss.config.js", "tailwind.config.js"]
}

```

2. **eslint 配置** eslint 配置简直就是永远的痛啊，太难了。这里主要是注意，alias map 的路径是当前 IDE 自行决定的，所以 `vscode` 和 `webstorm` 有时候**表现会不一致**！为了解决这个问题头发都掉了好多，其实解决方法也简单，就是在 `.eslintrc.js` 里用 `path.resolve(__dirname, './src')` 来指明别名路径，如下

```js
// 子项目 eslint 配置
const path = require('path');

module.exports = {
  // 继承规则
  extends: '../../.eslintrc.js',
  overrides: [
    {
      // 注意 overrides 需要配置 files 属性
      files: ['*.tsx', '*.ts', '*.jsx', '*.js'],
      settings: {
        // 自动检查 react 版本
        react: {
          version: 'detect',
        },
        // 配置路径别名
        'import/resolver': {
          alias: {
            map: [['@src', path.resolve(__dirname, './src')]],
            extensions: ['.tsx', '.ts', '.jsx', '.js'],
          },
        },
      },
    },
  ],
};

```

3. **tailwind class 失效** 原因暂时不明，推测是因为启动脚本在根目录，tailwind 在 js 入口组件里引入 `import 'tailwindcss/tailwindcss.css'` 的时候，目录指向不对，所以引入不了。**解决办法：**`新建一个 css，然后在入口组件里引入这个 css 就行`，内容就是引入 tailwind 的基本 css，如下

```css
@tailwind base;

@tailwind components;

@tailwind utilities;
```

4. **prettier 配置** 这个简单，一般也只需要在根目录配置一下 `.prettierrc` 就行了

5. **.npmrc 管理依赖提升问题** 这个配置没做，但是以后可能会用得上，可以解决一些依赖问题

### 总结

项目配置基本上都是一劳永逸的事情，所以一些东西也很容易忘记，而且这些前端工具一天一个样，只能说难搞，希望以后能有个统一简单的工程化方案吧

### 完
