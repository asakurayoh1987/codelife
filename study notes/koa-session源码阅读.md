# koa-session源码阅读记录
## 源码逻辑
这里有两篇讲解得不错的文章，就不再重复写了
[koa-session学习笔记](https://segmentfault.com/a/1190000013039187)
[从koa-session中间件源码学习cookie与session](https://segmentfault.com/a/1190000012412299)
## 关于获取sessionId
基于我们的业务场景，session的信息是存储在服务端（redis），cookie中仅保存session的一个标识——sessionId，虽然koa-session未提供该字段，但通过阅读源码，可以发现，当使用外部存储来保存session时，对应的有一个名为externalKey的字段（位于lib/context.js，为ContextSession类的实例属性，通过genid这个方法创建，存储在cookie中）来保存session的标识，这正是我们所需要的sessionId，接下来就是考虑如何获取这个字段。
### 方法一 修改源码
* 改动点：
源码在koa的context上暴露了session与sessionOptions的代码如下：
```javascript
Object.defineProperties(context, {
    [CONTEXT_SESSION]: {
      get() {
        if (this[_CONTEXT_SESSION]) return this[_CONTEXT_SESSION];
        this[_CONTEXT_SESSION] = new ContextSession(this, opts);
        return this[_CONTEXT_SESSION];
      },
    },
    session: {
      get() {
        return this[CONTEXT_SESSION].get();
      },
      set(val) {
        this[CONTEXT_SESSION].set(val);
      },
      configurable: true,
    },
    sessionOptions: {
      get() {
        return this[CONTEXT_SESSION].opts;
      },
    },
  });
```
externalKey是声明在ContextSession的实例中，而这个实例是通过Symbol的方式作为一个私有属性添加到koa的context对象上的，在外部是无法直接获取的，所以如果在这里直接添加一个sessionId的声明，如下：
```javascript
Object.defineProperties(context, {
    [CONTEXT_SESSION]: {
      get() {
        if (this[_CONTEXT_SESSION]) return this[_CONTEXT_SESSION];
        this[_CONTEXT_SESSION] = new ContextSession(this, opts);
        return this[_CONTEXT_SESSION];
      },
    },
    sessionId: {
      get() {
        return this[CONTEXT_SESSION].externalKey;
      },
    },
    ...
  });
```
在业务端使用时，只需ctx.sessionId即可获取
* 缺点：因为修改了源码，为了保持别人从git上下拉之后能正常进行依赖安装，必须将包的修改点进行上传，无非三种：原作者修改并更新npm包；作为自建包上传到npm；上传到一个团队可以下载的地方，比如git仓库
### 方法二 绕过访问限制
虽然ContextSession实例的访问被私有化，从外部不可访问，但作者也提供了一个session对象供访问，其获取方式如下：
```javascript
session: {
  get() {
    return this[CONTEXT_SESSION].get();
  },
},
```
阅读lib/context.js中，其get方法返回的是Session的实例（lib/session.js），再看create方法创建Session实例的代码：
```javascript
create(val, externalKey) {
  debug('create session with val: %j externalKey: %s', val, externalKey);
  if (this.store) this.externalKey = externalKey || this.opts.genid();
  // 这里的this指向当前的ContextSession实例
  this.session = new Session(this, val);
}
```
然后在Session的构造函数中有如下代码
```javascript
constructor(sessionContext, obj) {
  this._sessCtx = sessionContext;
  this._ctx = sessionContext.ctx;
  ...
}
```
可以发现Session实例的_sessCtx属性保存了对ContextSession实例的引用，所以在请求会话中，通过`ctx.session._sessCtx.externalKey`就绕过了对ContextSession实例访问的限制，拿到其下的externalKey属性。
PS：可以考虑封装一个处理session的中间件，在此中间件中处理koa-session的引入、配置以及sessionId的处理：
```javascript
const session = require('koa-session');
// 仅示例作用
const store = {
  store: {},
  get(key) {
    return this.store[key] || {};
  },
  set(key, sess) {
    this.store[key] = sess;
  },
  destroy(key) {
    delete this.store[key];
  },
};
module.exports = (app) => {
  Object.defineProperty(app.context, 'sessionId', {
    get() {
      /* eslint-disable no-underscore-dangle */
      return this.session._sessCtx.externalKey;
    },
  });
  app.use(
    session(
      {
        httpOnly: true,
        signed: false,
        renew: true,
        store,
      },
      app,
    ),
  );
};
```
缺点：虽然绕过访问限制，但不知道后续作者会不会修改源码，毕竟以'_'开头的属性属于私有属性是一个不成文的约定，不排队在后续的版本中会被去掉，如果使用此种方式，建议锁定koa-session的版本。