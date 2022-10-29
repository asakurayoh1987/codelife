# Node.js源码阅读

## 前提

* node分支为12.x
* 系统：WSL——Ubuntu 20.04（lsb_release -a查看系统版本）
* IDE：vscode
* g++编译环境准备，参考[这里](https://code.visualstudio.com/docs/cpp/config-wsl)

## Node源码编译

* check代码：`git clone -b v12.x https://github.com/nodejs/node.git`
* 在node代码的目录下执行`./configure --debug`
* 上一步执行完成后执行`make BUILDTYPE=Debug -j4`，其中`-j4`表示开启4个线程，可根据cpu数调整

## VSCode配置

* 插件安装：[C/C++ for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
* 打开node源码项目，执行Run-->Add Configuration新建配置文件，按如下方式修改

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "debug node",
      "type": "cppdbg",
      "request": "launch",
      // 指定刚才构建出来的debug包
      "program": "${workspaceRoot}/out/Debug/node",
      "args": [
        "--inspect-brk",
        // 测试用的js文件，内容可以直接是console.log('xxx')，也可以引用具体的内置模块看看代码是如何执行的
        "/home/ycyu/tmp/node/app.js"
      ],
      "stopAtEntry": false,
      "cwd": "${workspaceFolder}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "gdb",
      "setupCommands": [
        {
          "description": "Enable pretty-printing for gdb",
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        }
      ],
      "miDebuggerPath": "/usr/bin/gdb"
    }
  ]
}
```

* 后续则可进行在代码中打断点调试c++部分的代码，js部分的代码可以在浏览器中调试

## 源码阅读

### 关于宏

```c++
#define NODE_MODULE_CONTEXT_AWARE_X(modname, regfunc, priv, flags)    \
  extern "C" {                                                        \
    static node::node_module _module =                                \
    {                                                                 \
      NODE_MODULE_VERSION,                                            \
      flags,                                                          \
      NULL,                                                           \
      __FILE__,                                                       \
      NULL,                                                           \
      (node::addon_context_register_func) (regfunc),                  \
      NODE_STRINGIFY(modname),                                        \
      priv,                                                           \
      NULL                                                            \
    };                                                                \
    NODE_C_CTOR(_register_ ## modname) {                              \
      node_module_register(&_module);                                 \
    }                                                                 \
  }
  
#define NODE_MODULE_CONTEXT_AWARE_BUILTIN(modname, regfunc)           \
  NODE_MODULE_CONTEXT_AWARE_X(modname, regfunc, NULL, NM_F_BUILTIN)   \
```

> extern是C/C++语言中表明函数和全局变量作用范围（可见性）的关键字，该关键字告诉编译器，其声明的函数和变量可以在本模块或其它模块中使用。通常，在模块的头文件中对本模块提供给其它模块引用的函数和全局变量以关键字extern声明。例如，如果模块B欲引用该模块A中定义的全局变量和函数时只需包含模块A的头文件即可。这样，模块B中调用模块A中的函数时，在编译阶段，模块B虽然找不到该函数，但是并不会报错；它会在连接阶段中从模块A编译生成的目标代码中找到此函数。与extern对应的关键字是 static，被它修饰的全局变量和函数只能在本模块中使用。因此，一个函数或变量只可能被本模块使用时，其不可能被extern “C”修饰。
> 在C语言的该函数在编译器编译后在库中的名字为_foo，而C++中该函数被编译后在库中的名字为_foo_int_int（为实现函数重载所做的改变）。如果C++中需要使用C编译后的库函数，则会提示找不到函数，因为符号名不匹配。C++中使用extern “C”解决该问题，说明要引用的函数是由C编译的，应该按照C的命名方式去查找符号。 

* 可以通过`gcc -E`来查看宏展示后的效果

### 启动过程

* node_main.cc中Start方法
* node.cc中的InitializeOncePerProcess
* InitializeOncePerProcess下InitializeNodeWithArgs方法
* InitializeNodeWithArgs下binding::RegisterBuiltinModules();对于标准模块，进行宏展开后，以模块fs为例，会调用`_register_fs();`，对应在node_file.cc中，会有如下的宏：

```c++
NODE_MODULE_CONTEXT_AWARE_INTERNAL(fs, node::fs::Initialize)
```
这个宏定义在node_binding.h中，如下：

```c++
// 原代码中的换行用的'\'为了阅读方便已经去除
#define NODE_MODULE_CONTEXT_AWARE_CPP(modname, regfunc, priv, flags)
  static node::node_module _module = {                                   
      NODE_MODULE_VERSION,                                               
      flags,                                                             
      nullptr,                                                           
      __FILE__,                                                         
      nullptr,                                                           
      (node::addon_context_register_func)(regfunc),                     
      NODE_STRINGIFY(modname),                                           
      priv,                                                             
      nullptr};                                                         
  void _register_##modname() { node_module_register(&_module); }
#define NODE_MODULE_CONTEXT_AWARE_INTERNAL(modname, regfunc)             
  NODE_MODULE_CONTEXT_AWARE_CPP(modname, regfunc, nullptr, NM_F_INTERNAL)
```

```c
// node_module的结构说明如下，对应到上述的声明
struct node_module {
  int nm_version; // NODE_MODULE_VERSION
  unsigned int nm_flags; // NM_F_INTERNAL
  void* nm_dso_handle; // nullptr
  const char* nm_filename; // __FILE__
  node::addon_register_func nm_register_func; // nullptr
  node::addon_context_register_func nm_context_register_func; //(node::addon_context_register_func)(node::fs::Initialize)
  const char* nm_modname; // NODE_STRINGIFY(fs), 
  void* nm_priv; // nullptr
  struct node_module* nm_link; //nullptr
};
```

展开后的调用顺序为`_register_fs();`-->`node_module_registe()`会走到node_binding.cc下的`node_module_register`方法，此方法中会将当前的模块拼接到内建模块的最后：

```c
if (mp->nm_flags & NM_F_INTERNAL) {
	// 拼接的逻辑在这里，可以看到这里只做了模块的拼接，并没有调用初始化函数，初始化是在node_binding.cc中的GetInternalBinding中处理的
    mp->nm_link = modlist_internal;
    modlist_internal = mp;
  } else if (!node_is_initialized) {
    // "Linked" modules are included as part of the node project.
    // Like builtins they are registered *before* node::Init runs.
    mp->nm_flags = NM_F_LINKED;
    mp->nm_link = modlist_linked;
    modlist_linked = mp;
  } else {
    thread_local_modpending = mp;
  }
```

