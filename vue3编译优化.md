# 编译优化

## PatchFlag & block

  例如模板：

  ```html
    <div>
      <div>foo</div>
      <div>{{ bar }}</div>
    </div>
  ```

  编译成传统的虚拟`dom`：

  ```js
  const vnode = {
    tag: 'div',
    children: [
      { tag: 'p', children: 'foo'},
      { tag: 'p', children: ctx.bar },
    ]
  }
  
  ```

  编译优化后，编译器会将提取到他的关键信息附着在vnode上：

  ```JS
  // 只要存在 `patchFlag` 我们称为动态节点。
  const PatchFlags = {
    TEXT: 1, // 表示动态文本内容
    CLASS: 2, // 表示动态的 class
    STYLE: 3, // 表示动态的 style
    PROPS: 4 // 表示动态的 props
    // 省略...
  }

  const vnode = {
    tag: 'div',
    children: [
      { tag: 'p', children: 'foo'},
      { tag: 'p', children: ctx.bar, patchFlag: PatchFlags.TEXT }, // 这是一个动态节点
    ]
  }
  ```

  有了这项信息，我们可以将动态子节点提出来，存储到该虚拟节点的 `dynamicChildren`:

  ```js
  // ...
  const vnode = {
    tag: 'div',
    children: [
      { tag: 'p', children: 'foo'},
      { tag: 'p', children: ctx.bar, patchFlag: PatchFlags.TEXT }, // 这是一个动态节点
    ],
    // children 中的动态节点被提取到这里
    dynamicChildren: [
      { tag: 'p', children: ctx.bar, patchFlag: PatchFlags.TEXT }, // 这是一个动态节点

    ]
  }
  ```

  我们将带有 `dynamicChildren` 的虚拟节点称为 `block` （块）

  有了 block 概念后，渲染器更新操作将会以 block 为维度，也就是说，当渲染器 在 更新一个block时，会忽略虚拟节点的 children，而是
  直接找到 dynamicChildren 数组，并且只更新 数组中的动态节点，从而跳过静态内容，只更新动态内容。同时，由于动态节点中存在 补丁标志
  (PatchFlags), 所以只需要靶向更新。


  什么节点会称为 block 节点？

  - 所有模板的根节点
  - 带有 v-if、v-for、v-else-if



## 静态提升

### 静态节点提升

  例如：

  ```js
    <div>
      <div>foo</div>
      <div>{{ bar }}</div>
    </div>
  ```

  对应的渲染函数：

  ```js
  function render() {
    return (
      openBlock(), createBlock('div', null, [
        createVNode('p', null, 'foo'),
        createVNode('p', null, ctx.bar, 1 /*TEXT*/),
      ])
    )
  }

  // 将纯静态的 虚拟节点提升 

  const hosit1 =  createVNode('p', null, 'foo')
  function render() {
    return (
      openBlock(), createBlock('div', null, [
       hosit1, // 传递引用，这样每次render时候不会重新生成
        createVNode('p', null, ctx.bar, 1 /*TEXT*/),
      ])
    )
  }

  ```

  静态提升是以树为单位

### 静态节点提升

  例如：

  ```html
    <div>
      <div foo="bar" a=b>{{ text }}</div>
    </div>
  ```

  可以将静态属性提升

  ```JS
  const hositProps = { foo: 'bar', a: 'b' }
  function render() {
    return (
      openBlock(), createBlock('div', null, [
        createVNode('div', hositProps, ctx.text, 1 /*TEXT*/),
      ])
    )
  }
  ```

### 预字符串化

  预字符串化 基于动态提升的一种策略。

  例如：

  ```html
  <div>
    <p></p>
    <p></p>
    <p></p>
    <!-- ... 20个p标签 -->
  </div>
  ```

  我们将静态节点序列化为字符串，并生成一个 `static` 类型的 `vnode`:

  ```JS
  const hoistStatic = createStaticVNode('<p></p><p></p><p></p><p></p>...20个...<p></p>')
  
  render() {
    return (openBlock(), createBlock('div', null, [hoistStatic]))
  }
  ```

  优势在于：
  - 大块的静态内容可以通过 innerHTML 进行设置， 性能上有一定优势
  - 减少内容占用
  - 减少虚拟节点 虚拟节点创建产生的性能开销

### 缓存内联事件处理函数

例如：

```html
<Comp @chang="a + b"/>
```

编译器会将其编译为：

```JS
function render() {
  return h(Comp, {
    onChange: () => (ctx.a + ctx.b) // 每次重新渲染，都会为组件创建一个全新的 props对象。同时props中的onChange属性值也会是一个
    // 全新的值
  })
}

// 我们增加缓存进行优化
function render() {
  return h(Comp, {
    onChange: cache[0] || (cache[0] = ($event) => (ctx.a + ctx.b))
    // 全新的值
  })
}

```

### v-once


例如：

```vue
<template>
  <div>
    <div v-once>
      {{ NAME }}
    </div>
  </div>
</template>
```

将被编译为：

```js
import { setBlockTracking as _setBlockTracking, toDisplayString as _toDisplayString, createTextVNode as _createTextVNode, createElementVNode as _createElementVNode, openBlock as _openBlock, createElementBlock as _createElementBlock } from "vue";

const NAME = "bar";

function _sfc_render(_ctx, _cache, $props, $setup, $data, $options) {
  return _openBlock(), _createElementBlock("div", null, [
    _cache[0] || (_setBlockTracking(-1), _cache[0] = _createElementVNode("div", null, [
      _createTextVNode(_toDisplayString($setup.NAME))
    ]), _setBlockTracking(1), _cache[0])
  ]);
}
```

适用于 需用使用静态变量，但是 只需要渲染一次的场景。
