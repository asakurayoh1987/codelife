# VitePress官方文档

## 一、起步

### 1.1 Node.js版本要求

**Node.js版本要求18及以上**

如果是通过`nvm`安装了多个node.js版本，并且默认版本不是满足要求的node.js版本，可以在当前项目目录下添加`.nvmrc`配置，指定当前项目所使用的node.js版本

![image-20231227094009179](../media/image-20231227094009179.png)

```bah
echo 'v21' > .nvmrc
```

**VitePress只支持ESM**

即不允许通过`require()`来引入，所以要么在`package.json`中设置`"types": "module"`，要么将文件名后缀修改为".mjs"或".mts"

### 1.2 使用方式

- 现有项目中集成：`pnpm add -D vitepress`
- 使用引导创建新的vitepress项目：`pnpm vitepress init`

### 1.3 目录结构

1. `.vitepress`目录作为vitepress的保留目录，里面保存了vitepress相关的**配置文件（`config.mts`）、开发服务器的缓存（cache目录）、构建产物（dist目录）以及主题自定义配置相关的逻辑**

2. **project root**：

3. `.vitepress`外层的`.md`文件都被认为是源文件，最终会编译成`.html`，其中`index.md`会编译成`index.html`并直接通过使用网站的根目录（"/"）来访问

   **PS：VitePress支持路径覆写、动态生产页面等能力**

## 二、路由处理

### 2.1 基于文件目录的路由机制

### 2.2 根目录及源文件目录

1. 根目录，即`project root`，vitepress据此来查找`.vitepress`目录（前文提到了它的作用）

2. 当通过命令行使用`vitepress dev`或`vitepress build`时，会以当前目录作为根目录，也可以通过如`vitepress dev docs`来指定以`docs`目录作为根目录来启动项目

3. 根目录会影响最终生成的页面的访问路径，因为vitpress使用的基于文件目录的路由机制，如下目录结构

   ```
   .
   ├─ docs                    # project root
   │  ├─ .vitepress           # config dir
   │  ├─ getting-started.md
   │  └─ index.md
   └─ ...
   ```

   `docs`是根目录 ，所以`docs/index.md`最终