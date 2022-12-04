# Webpack5核心原理与实战

## 一、知识图谱

![25c05841892d41bfae468d496a968442_tplv-k3u1fbpfcp-zoom-in-crop-mark_3024_0_0_0](../media/25c05841892d41bfae468d496a968442_tplv-k3u1fbpfcp-zoom-in-crop-mark_3024_0_0_0.webp)

## 二、Webpack配置底层结构逻辑

### webpack编译流程

![img](../media/784ef02a144a4b3ebfbcbe13b06ac7bb~tplv-k3u1fbpfcp-zoom-in-crop-mark.awebp)

- 输入：从文件系统读入代码文件；
- 模块处理：调用Loader转译Module内容，并将其转换为AST，从中分析模块依赖关系，进一步**递归**调用模块处理的过程，直到所有依赖文件处理完毕
- 后处理：所有模块递归处理完毕后开始执行后处理，包括模块合并、注入运行时、产物优化等，最终输出Chunk集合
- 输出：将Chunk写入到外部文件系统

根据上述流程，可将Webpack的配置项归为两大类

- 流程类：用于打包流程的某个或某几个环节，直接影响编译打包效果的配置项

  - 输入（找到入口文件）输出（产物输出位置）：
    - entry：定义项目入口文件，Webpack会从这些入口文件开始递归找出所有依赖项
    - context：项目执行上下文路径，用于与entry配合使用
    - output：构建产物的输出路径、名称等
  - 模块处理（解析模块，包括转译、依赖分析）：
    - resolve：配置模块路径解析规则，帮助Webpack更精确、高效地找到指定模块
    - module：用于配置模块的加载规则，比如使用指定Loader进行加载
    - externals：用于声明外部资源，让Webpack在打包时忽略这部分资源，跳过这些资源的解析、打包操作
  - 后处理（合并模块、注入运行时依赖、优化产物结构）：
    - optimization：用于控制如何优化产物包体积，内置Dead Code Elimination、Scope Hoisting、代码混淆、代码压缩等功能
    - target：用于配置编译产物的目标运行环境，支持web、node、electron等值，不同值最终产物会有所差异
    - mode：编译模式短语，支持development、production等值，可以理解为一种声明环境的短语，并提供了一些默认的配置项

- 工具类：打包主流程之外，提供更多工程化的配置项

  - 开发效率类：
    - watch：监听文件变化，持续构建
    - devtool：配置Sourcemap生成规则
    - devServer：配置与HMR强相关的开发服务器功能
  - 性能优化类：
    - cache：Webpack5之后，用于控制如何缓存编译过程信息与编译结果
    - performance：用于配置产物大小超过阈值时，如何通知开发者
  - 日志类：
    - stats：精确地控制编译过程的日志内容，在做比较细致的性能调试时非常有用
    - infrastructureLoggin：用于控制日志输出方式，如可以将日志输出到磁盘文件
  - 其它...

  ![img](../media/02ccea1194e045689143011ef62ff553~tplv-k3u1fbpfcp-zoom-in-crop-mark.awebp)