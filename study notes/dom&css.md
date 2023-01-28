# DOM & CSS 相关知识

## 关于nodeType

参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Node/nodeType#%E8%8A%82%E7%82%B9%E7%B1%BB%E5%9E%8B%E5%B8%B8%E9%87%8F)

觉的div、p的nodeType为1（Node.ELEMENT_NODE），文件结点为nodeType为3（Node.TEXT_NODE）

## 关于z-index

![img](../media/608782-20160923104742809-2054066790.png)

按照 [W3官方](https://www.w3.org/TR/CSS2/visuren.html#propdef-z-index) 的说法，**准确的 7 层为：**

1. the background and borders of the element forming the stacking context.
1. the child stacking contexts with negative stack levels (most negative first).
1. the in-flow, non-inline-level, non-positioned descendants.
1. the non-positioned floats.
1. the in-flow, inline-level, non-positioned descendants, including inline tables and inline blocks.
1. the child stacking contexts with stack level 0 and the positioned descendants with stack level 0.
1. the child stacking contexts with positive stack levels (least positive first).

**翻译一下：**

1. 形成堆叠上下文环境的元素的背景与边框
1. 拥有负 `z-index` 的子堆叠上下文元素 （负的越高越堆叠层级越低）
1. 正常流式布局，非 `inline-block`，无 `position` 定位（static除外）的子元素
1. 无 `position` 定位（static除外）的 float 浮动元素
1. 正常流式布局， `inline-block`元素，无 `position` 定位（static除外）的子元素（包括 display:table 和 display:inline ）
1. 拥有 `z-index:0` 的子堆叠上下文元素
1. 拥有正 `z-index:` 的子堆叠上下文元素（正的越低越堆叠层级越低）

**如何触发一个元素形成 `堆叠上下文` ？**方法如下，摘自 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context)：

- 根元素 (HTML),
- z-index 值不为 "auto"的 绝对/相对定位，
- 一个 z-index 值不为 "auto"的 flex 项目 (flex item)，即：父元素 display: flex|inline-flex，
- opacity 属性值小于 1 的元素（参考 the specification for opacity），
- transform 属性值不为 "none"的元素，
- mix-blend-mode 属性值不为 "normal"的元素，
- filter值不为“none”的元素，
- perspective值不为“none”的元素，
- isolation 属性被设置为 "isolate"的元素，
- position: fixed
- 在 will-change 中指定了任意 CSS 属性，即便你没有直接指定这些属性的值
- -webkit-overflow-scrolling 属性被设置 "touch"的元素

## 关于`:not`伪类

- `:not` 伪类不像其它伪类，它不会增加选择器的优先级。它的优先级即为它参数选择器的优先级。
- 这个选择器只会应用在一个元素上， 你不能用它在排除所有祖先元素。 举例来说， body :not(table) a 将依旧会应用在table内部的`<a>` 上, 因为 `<tr>`将会被:not() 这部分选择器匹配。（摘自[MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:not)）