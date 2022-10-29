# npm最佳实践

## 一、命令

- npx npm-packlist

  用于发布npm包之前查看将被包含在所发布的版本包中的内容

- npm publish --dry-run

  查看publish命令实际将会做什么

- npm publish --access=public

  用于发布scope包时修改包的可访问性，因为scope包默认是私有的，也可以修改package.json中的private字段为false，来将其公开

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

