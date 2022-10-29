# vscode调试npm scripts

这里以调试jest的测试脚本为例

## 添加npm scripts

```json
{
	"scripts":{
		"debug":"node --inspect-brk=9102 node_modules/.bin/jest"
	}
}
```

## 添加vscode调试配置

* 通过vscode添加node.js默认调试配置（略）
* 修改`.vscode/launch.json`，即上一步添加的文件
* 如下修改

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug npm scripts",
      // 指定使用npm来运行
      "runtimeExecutable": "npm",
      // 运行参数，数组中第一个元素固定为'run-script'，第二个元素为要调试的脚本名称，后续参数就如在命令行运行npm时一样，指定传给此脚本的参数
      "runtimeArgs": [
        "run-script",
        "debug",
        "--",
        "-t",
        "1.1"
      ],
      "cwd": "${workspaceFolder}",
      // 端口要写npm scripts中的--inspect-brk指定的端口一致
      "port": 9102,
      // 在代码的第一行断点
      "stopOnEntry": true,
      "outputCapture": "std",
    }
  ]
}

```

[参考文档](https://code.visualstudio.com/docs/nodejs/nodejs-debugging#_launch-configuration-support-for-npm-and-other-tools)