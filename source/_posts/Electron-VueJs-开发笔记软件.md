---
title: Electron+Vue 开发笔记软件
comments: true
mathjax: false
date: 2021-04-28 22:53:58
tags:
    - Electron
    - Vue
    - Index
categories:
    - [Projects, Softwares]
---

- <a href="{% post_path Vue-学习记录 %}">Vue 学习记录</a>

<!-- more -->

# 搭建基本项目

## 相关依赖

- node、npm（cnpm、yarn）
- vue-cli：
  ```bash
  sudo cnpm i -g @vue/cli
  ```
- 

## vue 基本项目

如在 electron-vue 目录下创建项目：
```bash
vue create electron-vue
```

这时就可以用 `yarn serve` 命令运行项目了。

{% note warning %}
注意路由（router）不要启用 histroy 模式，否则最后打包时会出现问题。
{% endnote %}

## 加入 electron

安装 [vue-cli-plugin-electron-builder](https://nklayman.github.io/vue-cli-plugin-electron-builder/)：

```bash
vue add electron-builder
```

然后即可通过 yran（推荐）或 npm 启动测试 server：

```bash
yarn election:serve
npm run electron:serve
```

要构建应用：

```bash
yarn electron:build
npm run electron:build
```

{% note info %}
首次进行这两个步骤需要访问境外资源，可能会由于网络原因而失败（解决方法不便明说）。
{% endnote %}

## 配置 Electron Builder

文档：
- https://nklayman.github.io/vue-cli-plugin-electron-builder/guide/configuration.html#configuring-electron-builder
- https://www.electron.build/

```json "vue.config.js"
module.exports = {
  pluginOptions: {
    electronBuilder: {
      builderOptions: {
        // options placed here will be merged with default configuration and passed to electron-builder
        // Windows 平台打包配置
        // https://www.electron.build/configuration/nsis
        win: {
          target: 'nsis'        // 打包方式
        },
        nsis: {
          oneClick: false,      // 禁用一键安装
          allowToChangeInstallationDirectory: true,
        }
      }
    }
  }
}
```

