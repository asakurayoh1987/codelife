# Javascript基础



## 一、this的使用场景

1. 对象的.操作符，此时this指向.操作符前
2. new 构造函数，此时this指向new操作符创建的新对象
3. 原型对象中的this，指向具体调用时的实例，即谁调用就指谁
4. 普通函数调用、匿名函数调用、回调函数中的this，指向window，但在严格模式下返回undefined
5. DOM事件处理函数里的this指向正在触发事件的.前DOM元素对象
6. vue中的this指向当前vue对象（Composition API和setup中已经没有this）
7. 箭头函数中的this指向当前函数之外最近的作用域中的this
8. 使用call\apply来替换this
   1. 临时替换：call、apply，后者的参数是以数组形式传递
   2. 创建副本，永久绑定this：bind，同时还可以绑定部分形参

## 一、创建对象的方式

1. 字面量

   ```javascript
   var obj = {...};
   ```
   
2. new Object表达式

   ```javascript
   // 第一步new操作符创建对象
   var obj = new Object();
   // 第二步添加属性或方法
   obj.xxx = ...
   ```

   揭示了javascript中所有对象底层都是**关联数组**，五大证据

   - 存储结构：都是名值对儿的组合
   - 访问成员时，既可以用["成员名"]，也可以用.成员名
   - 强行给不存在的位置赋值不会报错，而是会自动添加该属性
   - 强行访问不存在的位置的值不会报错
   - 都可以用for...in的方式遍历

3. 用构造函数

   前面的两种方式一次只能创建一个对象，使用构造函数可以创建相同结构内容不同的对象
   
   第一步：创建构造函数
   
   第二步：new 构造函数名()来创建对象实例
   
   构造函数的原理：
   
   - new做了4件事：
     - 创建新对象
     - 设置新对象的__proto__属性，指向构造函数的原型对象，即构造函数.prototype
     - 调用构造函数，将构造函数中的this指向刚才创建的对象
     - 返回新对象的地址，保存到变量中
   - 向原型对象中添加共有属性
     - 强行赋值：构造函数.prototype.xxx=xxx
   
4. 工厂函数方式——判断不了类型，都是object

5. 原型对象方式：先创建完全相同的对象，再给子对象添加个性化属性

6. 混合模式：先创建完全相同的对象，再给子对象添加个性化属性

7. 动态混合：在构造函数内部添加prototype上的方法或属性，因为只应添加一次，所以需要配置if

8. 寄生构造函数：构造函数里调用其他的构造函数

9. ES6 Class：新瓶装旧酒，换汤不换药

10. 稳妥构造函数：闭包，不用this，不用new

    ```javascript
    function Person(name, age) {
      var p = {};
      p.getName = function(){return name};
      p.setName = function(value){name = value};
      return p;
    }
    ```

    

    