[原文](https://blog.nrwl.io/setup-a-monorepo-with-pnpm-workspaces-and-speed-it-up-with-nx-bc5d97258a7e)

## pnpm命令相关

- pnpm --filter \<package-name\> \<command\>

  根据名称进行过滤，让`command`仅在对应的`package`下执行：`pnpm --filter my-remix-app dev`

- pnpm add \<package-name\> --filter \<package-name> --workspace

  将`workspace`中的依赖添加到当前项目中（按`workspace`的协议），添加后在`package.json`看到的效果是：

  ```json
  "dependencies": {
  	"xxx": "workspace:*"
  }
  ```

- pnpm run -r \<command\>

  在所有package中执行`command`

- pnpm run --parallel -r \<command\>

  同上，区别在于是并行执行的

## Nx命令相关

- pnpm add nx -D -w

  添加nx，这里的`-w`表示将nx添加到项目根目录
  
- pnpm exec nx \<target\> \<project\>

  执行任务，比如：`pnpm exec nx build shared-ui`，在项目`shared-ui`下执行`build`命令

- 配置缓存以及构建依赖项，通过nx.json文件，指定需要缓存的操作（命令）

  ```json
  {
    "tasksRunnerOptions": {
      "default": {
        "runner": "nx/tasks-runners/default",
        "options": {
          // 指定需要进行缓存的操作
          "cacheableOperations": ["build", "test"]
        }
      }
    },
    "namedInputs": {
      // 出于复用考虑
      "noMarkdown": ["!{projectRoot}/**/*.md"]
    },
    "targetDefaults": {
      "build": {
        // 通过glob指定哪些类型的文件更新，不作为缓存的输入（默认是所有文件都作为输入）
        // "^"表示项目的依赖项中也按这个逻辑
        "inputs": ["noMarkdown", "^noMarkdown"],
        // 执行build时，该项目所依赖的其他项需要先进行build，"^"表示所有依赖的项目，比如间接依赖
        "dependsOn": ["^build"]
      },
      "test": {
        "inputs": ["noMarkdown", "^noMarkdown"]
      },
      "dev": {
        "dependsOn": ["^build"]
      }
    }
  }
  ```

  

- pnpm exec nx graph

  查看项目的依赖关系图