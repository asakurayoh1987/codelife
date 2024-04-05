# Ring-Pro Pnpm Workspace

## 一、pnpm-workspace.yaml

用于定义整个monorepo下子项目的关联关系，例如：

```yaml
packages:
  - 'business/**'
  - 'apps/*'
  - 'packages/**'
```

## 二、配置pnpm

通过`.npmrc`可以定义一个pnpm的配置，例如

```
public-hoist-pattern[]=*lint*
public-hoist-pattern[]=*prettier*
public-hoist-pattern[]=pretty-quick
public-hoist-pattern[]=axios
public-hoist-pattern[]=vconsole
public-hoist-pattern[]=vite-plugin-checker
public-hoist-pattern[]=*vue-macros*
public-hoist-pattern[]=unplugin-vue-define-options
public-hoist-pattern[]=unplugin-auto-import*

public-hoist-pattern[]=lodash-es
public-hoist-pattern[]=@types/lodash-es

registry=https://depend.iflytek.com/artifactory/api/npm/npm-repo/

strict-peer-dependencies=false
enable-pre-post-scripts=true
ignore-workspace-root-check=true
```

- public-hoist-pattern[]=\*lint\*

  用于开启公共依赖的提升，如果命中则将该依赖包提升至顶层目录

- registry=xxx

  用于设置npm仓库的地址，这样安装npm依赖包里不走npm官方仓库

- strict-peer-dependencies=false

- enable-pre-post-scripts=true

  默认是不开启pre与post钩子，当这项设置为true时，则允许像npm那样支持pre与post钩子

- Ignore-workspace-root-check=true

### 执行pnpm install时执行了什么

ChatGPT的回答：

> 在一个使用 pnpm 管理的 workspace 项目中，当你在项目的根目录下执行 `pnpm install` 时，执行流程如下：
>
> 1. **读取配置文件**：
>    - pnpm 会读取项目根目录中的 `pnpm-workspace.yaml` 文件和 `package.json` 文件，以确定哪些项目是工作区的一部分，以及工作区的根目录。
> 2. **解析依赖关系**：
>    - pnpm 会解析所有工作区项目的依赖关系，包括 `dependencies`、`devDependencies`、`peerDependencies` 和 `optionalDependencies`。
> 3. **创建虚拟存储库**：
>    - pnpm 会在工作区的根目录下创建一个名为 `node_modules` 的虚拟存储库，使用符号链接（symlinks）将工作区中的每个包连接到适当的位置。
>    - 虚拟存储库通过符号链接来共享依赖，从而避免了重复安装相同的依赖。
> 4. **安装根目录依赖**：
>    - pnpm 会首先安装根目录下的 `package.json` 文件中指定的依赖项。
> 5. **安装工作区项目依赖**：
>    - 然后，pnpm 会遍历 `pnpm-workspace.yaml` 文件中列出的每个工作区项目，分别安装它们的依赖项。
>    - 如果多个项目依赖相同的包，pnpm 会尽可能地共享这些依赖，减少重复安装。
> 6. **创建符号链接**：
>    - pnpm 会在每个项目的 `node_modules` 目录中创建符号链接，指向共享的依赖。
>    - 符号链接确保每个项目都能正确地访问其依赖项，同时避免了重复安装。
> 7. **生成 `pnpm-lock.yaml` 文件**：
>    - pnpm 会生成或更新 `pnpm-lock.yaml` 文件，该文件记录了当前安装的所有依赖项的精确版本，以及它们之间的依赖关系。
>    - `pnpm-lock.yaml` 文件确保项目在不同环境中安装时使用相同的依赖版本。
> 8. **处理 `hoisting`（提升）**：
>    - pnpm 会根据需要将一些常用的依赖项提升到更高的目录层次，以减少项目之间的重复依赖。
>    - 提升策略可以通过配置文件进行调整，以满足特定项目的需求。
> 9. **清理和优化**：
>    - pnpm 会清理未使用的包，确保 `node_modules` 目录保持干净。
>    - pnpm 还会优化安装过程，提高后续安装的速度。
>
> 整个流程旨在高效管理工作区中的多个项目，最大限度地减少磁盘空间占用，并确保依赖项的一致性。pnpm 的符号链接和共享依赖的机制，使其在处理 monorepo 项目时具有显著的优势。在 pnpm 中，如果启用了 `enable-pre-post-scripts=true` 选项，并且在根目录的 `package.json` 以及子项目的 `package.json` 中都定义了 pre 和 post 钩子，它们的执行顺序如下：
>
> 1. **根目录的 `preinstall` 钩子**：
>    - 如果根目录的 `package.json` 中有 `preinstall` 脚本，pnpm 会首先执行这个脚本。
> 2. **子项目的 `preinstall` 钩子**：
>    - 接下来，pnpm 会依次遍历子项目的 `package.json` 文件，并执行每个子项目中的 `preinstall` 脚本。
> 3. **安装依赖**：
>    - pnpm 会安装根目录以及子项目的依赖。
> 4. **子项目的 `install` 钩子**：
>    - 安装依赖后，pnpm 会依次遍历子项目的 `package.json` 文件，并执行每个子项目中的 `install` 脚本。
> 5. **根目录的 `install` 钩子**：
>    - 接下来，pnpm 会执行根目录的 `install` 脚本。
> 6. **子项目的 `postinstall` 钩子**：
>    - 然后，pnpm 会依次遍历子项目的 `package.json` 文件，并执行每个子项目中的 `postinstall` 脚本。
> 7. **根目录的 `postinstall` 钩子**：
>    - 最后，pnpm 会执行根目录的 `postinstall` 脚本。
>
> 总结一下，完整的执行顺序如下：
>
> 1. 根目录的 `preinstall`
> 2. 每个子项目的 `preinstall`（依次执行）
> 3. 安装依赖
> 4. 每个子项目的 `install`（依次执行）
> 5. 根目录的 `install`
> 6. 每个子项目的 `postinstall`（依次执行）
> 7. 根目录的 `postinstall`
> 8. 执行`prepare`
>
> 这种顺序确保了在安装依赖之前和之后，分别执行定义好的脚本，从而满足项目的定制化需求。

如果想查看pnpm的相关日志，可以使用类似`DEBUG=pnpm:* pnpm install`的命令

## 三、项目依赖npm包

### 1. [unbuild](https://www.npmjs.com/package/unbuild)

### 📦 Optimized bundler

Robust [rollup](https://rollupjs.org/) based bundler that supports typescript and generates commonjs and module formats + type declarations.

### 2. [tsup](https://www.npmjs.com/package/tsup)

Bundle your TypeScript library with no config, powered by [esbuild](https://github.com/evanw/esbuild).

### 3. [tsx](https://www.npmjs.com/package/tsx)

TypeScript Execute (tsx): The easiest way to run TypeScript in Node.js

