# Lerna 常用命令



## 一、初始化

### 1. 固定模式

```bash
lerna init
```

所有的包共用一个版本线，版本号保存在`lerna.json`中的`version`字段，子包中任一修改，在使用`lerna publish`时，都会更新所有子包的版本号至最新，Babel目前就是使用这种模式，如果你希望将所有子包的版本号都绑定在一起，则可以使用这种方式，但这种方式的一个问题是，即任一包的主版本号变化，会使用所有包都更新为一个新的主版本号，这在版本语义上可能不友好

### 2. 独立模式

```bash
lerna init --independent
```

可简写为`lerna init -i`，此模式下子包有独立的版本线，每次进行`lerna publish`时，会得到了一个提示，来指定当前包是补丁、次版本号变更还是主版本号变更，可以通过将`lerna.json`中的`version`设为字符串`independent`为将其由固定模式变更为独立模式

### 3. lerna.json

```json
{
	// 版本号
  "version": "1.1.3",
  // 执行命令的客户端，默认为npm，可以修改为"yarn"
  "npmClient": "npm",
  // 特定命令的配置项
  "command": {
  	// lerna publish时使用些配置
    "publish": {
      // 指定一组glob，用于指定在执行lerna changed/publish时过滤的文件，比如指定README.md的变动不应导致版本号的变更
      "ignoreChanges": ["ignored-file", "*.md"],
      // git提交时的自定义消息
      "message": "chore(release): publish",
      // npm包发布的位置，用来代替默认的`npmjs.org`
      "registry": "https://npm.pkg.github.com"
    },
    "bootstrap": {
      // 指定执行`lerna bootstrap`时过滤掉的子包
      "ignore": "component-*",
      // 在执行`lerna bootstrap`时传递给如`npm install`命令的参数
      "npmClientArgs": ["--no-package-lock"],
    }
  },
  // 子包的位置，这里是相对于`lerna.json`文件的位置
  "packages": ["packages/*"]
}
```

## 二、公用devDependencies

大多数`devDependencies`可以提到lerna仓库的根级，通过使用`lerna link covert`来自动提升这些`devDependencies`，并使用`file:`标识符指定的相对路径，但需要注意一点，`devDependencies`提供的二进制可执行文件（通常通过npm script来调用）还是需要直接安装到每个包中

## 二、创建子模块

```bash
# lerna create <name> [loc]
lerna create 
```

## 三、添加npm包

```bash
# lerna add <package>[@version] [--dev] [--exact]
```
添加本地包或远程包到当前的lerna项目下的每一个子模块中，一次只能添加一个包，默认情况下是作为dependency依赖，如果指定`--dev`则表示作为devDependencies添加，如果需要指定准确版本号，比如`1.0.1`而非`^1.0.1`的形式，则可以使用`--exact`标识

其它标识：
`--registry <url>`：从指定的仓库安装包
`--no-bootstrap`：跳过`lerna bootstrap`操作，默认安装依赖包之后会触发一次[bootstrap](#四、bootstrap)的操作
`--scope`：指定添加到某个子模块中，如：
```bash
lerna add module-1 --scope=module-2
```

## 四、bootstrap

