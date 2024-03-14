# npm最佳实践

## 一、命令

- `npx npm-packlist`

  用于发布npm包之前查看将被包含在所发布的版本包中的内容

- `npm publish --dry-run`

  查看publish命令实际将会做什么

- `npm publish --access=public`

  用于发布scope包时修改包的可访问性，因为scope包默认是私有的，也可以修改package.json中的private字段为false，来将其公开
  
- `npm docs [package-name]`

  查看指定npm包的文档
  
- `npm repo [package-name]`

  查看指定npm包的git仓库

- `npm outdated`

  检查当前项目中所有包的版本情况，包括当前版本号、要求的版本号以及最新的版本号

- `npm v [package-name] versions`

  查看指定npm包的所有历史版本

- `npm audit [--fix]`

  查看当前项目所使用的包是否存在漏洞（需要向默认的registry发送请求并提交当前的依赖配置）并得到一份建议，如果指定了`--fix`参数，则会自动应用返回的建议（升级包的版本）
  
- `npm version [major|minor|patch|]`

  修改版本号，major-主版本号；minor-次版本号；patch-修订版本号
  
- `npm cache clean  --force`

  清空本地缓存
  
- `npm config set registry xxx`

  设置镜像源为xxx
  
- `npm config list`

  查看配置信息

- `npm update [package-name]`

  更新npm包（可以指定包名）

  从npm v2.6.1 开始，npm update只会更新顶层的模块，而不更新依赖的依赖模块，而之前的版本是递归更新的。如果想要这种效果，可以使用以下命令

  `npm --depth 9999 update`

- `npm info [package-name]`

  查看远程npm上指定包的版本信息

- `npm root [-g]`

  查看当前包（或全局包）的安装路径

- `npm home [package-name]`

  查看指定包名的首页介绍，此操作会打开浏览器

- `npm config set save-prefix='~'`

  默认安装包时，在package.json中的版本号以"^"开头，表示当再次执行`npm install`时，保持该包大版本号不变，只升级次版本号，如果想修改这个功能，比如修改前缀为"~"（再次安装时保持该包小版本号也不变，只升级补丁版本号）

- `npm config set save-exact true`

  与上一条类似，此处表示锁定版本号

- `npm shrinkwrap`

  **彻底的锁定依赖的版本，让应用在任何机器上都安装同样的版本。**

  执行这个命令之后，就会在项目的根目录产生一个npm-shrinkwrap.json配置文件，这里面包含了通过node_modules 计算出的模块的依赖树及版本。只要目录下有 npm-shrinkwrap.json 则运行 npm install 时就会优先使用 npm-shrinkwrap.json 中的配置进行安装，没有则使用 package.json 进行安装

- `npm outdated`

  查看过时的依赖项，可以配合`npm view <package-name>`来查看npm包的最新版本

- `npm ci`

  当执行该命令时，它会先删除本地的node_modules文件，因此它不需要去校验已下载文件版本与控制版本的关系，也不用校验是否存在最新版本的库，所以下载的速度相比npm install会更快。之后它会按照 package-lock.json 文件来安装确切版本的依赖项。并且不会将这个版本写入package.json或者package-lock.json文件，使用该命令时要注意：

  - 项目必需有 package-lock.json 或 npm-shrinkwrap.json 文件，如果没有，该命令将不起作用；
  - npm ci 是 npm v6 中引入了的新命令，所以使用该命令时需要确保npm版本要>=5.7；
  - npm ci 不能用来安装单个依赖，只能用来安装整个项目的依赖；
  - npm ci 会安装 dependencies 和 devDependencies；
  - 整个安装过程不会更新 package.json 或 package-lock.json 文件，整个安装过程是锁死的；
  - 当package-lock.json中的依赖和package.json中不一致时，npm ci 会退出但不会修改package-lock.json文件。

- `npm dedupe`或`npm ddp`

  删除重复包并在多个依赖包之间共享公共依赖项来简化整体的结构。**它会产生一个扁平的、去重的树**

- `npm audit`

  扫描项目来查找所有依赖项中存在的漏洞，并且可以通过`npm audit fix`来自动安装所有易受攻击包的补丁版本



## 二、构建CommonJS模块和ECMAScript模块

这里以typescript为例

1. 创建tsconfig.base.json配置，保存通用的编译选项

   ```json
   {
     "compilerOptions": {
       "strict": true,
       "esModuleInterop": true,
       "forceConsistentCasingInFileNames": true,
       "skipLibCheck": true,
       "checkJs": true,
       "allowJs": true,
       "declaration": true,
       "declarationMap": true,
       "allowSyntheticDefaultImports": true
     },
     "files": ["../src/index.ts"]
   }
   ```

2. 创建cjs的配置项：tsconfig.cjs.json

   ```json
   {
     "extends": "./tsconfig.base.json",
     "compilerOptions": {
       "lib": ["ES6", "DOM"],
       "target": "ES6",
       "module": "CommonJS",
       "moduleResolution": "Node",
       "outDir": "../lib/cjs",
       "declarationDir": "../lib/cjs/types"
     }
   }
   ```

   - lib 属性向TypeScript指出它应该参考哪些类型。
   - target 属性向TypeScript指出要编译的项目代码的JavaScript版本。
   - module 属性向 TypeScript 指出在编译的项目代码时应该使用哪种JavaScript模块格式。
   - moduleResolution 属性帮助 TypeScript 弄清 "import"语句应该如何被提及。
   - outDir 和 declarationDir 属性向TypeScript指出了将编译的代码和定义其中使用的类型的结果放在哪里。

3. 创建esm的配置选项：tsconfig.esm.json

   ```json
   {
     "extends": "./tsconfig.base.json",
     "compilerOptions": {
       "lib": ["ES2022", "DOM"],
       "target": "ES2022",
       "module": "ESNext",
       "moduleResolution": "NodeNext",
       "outDir": "../lib/esm",
       "declarationDir": "../lib/esm/types"
     }
   }
   ```

4. 修改package.json中的字段

   - 增加一个files字段，指向lib文件夹

     ```json
     "files": [
        "lib/**/*"
     ],
     ```

   - 更新exports字段：

     ```json
     "exports": {
       ".": {
         "import": {
           "types": "./lib/esm/types/index.d.ts",
           "default": "./lib/esm/index.mjs"
         },
         "require": {
           "types": "./lib/cjs/types/index.d.ts",
           "default": "./lib/cjs/index.js"
         }
       }
     },
     ```


## 三、语义版本管理

这里借助工具：`semantic-release`

```
npm i -D semantic-release
npx semantic-release-cli setup
```



## 参考来源：

- [创建现代npm包的最佳实践](https://www.toutiao.com/article/7152047393553318411/?app=news_article&timestamp=1666567208&use_new_style=1&req_id=202210240720080101420170210FABCED9&group_id=7152047393553318411&wxshare_count=1&tt_from=weixin&utm_source=weixin&utm_medium=toutiao_android&utm_campaign=client_share&share_token=9abd3e3e-5a05-4737-82ec-67cf72116cb2&source=m_redirect&wid=1666569852110)
- [Best practices for creating a modern npm package](https://snyk.io/blog/best-practices-create-modern-npm-package/)
- [npm包常用命令介绍](https://mp.weixin.qq.com/s/c2JNUWBsUpgPC_kwgshgEg)



