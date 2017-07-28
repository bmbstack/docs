title: React 一个用户构建用户界面的库
---

## 组件、组件实例、元素

通常一个组件在页面中会产生多个组件实例，每个组件实例都有各自的属性props和状态state，当一个组件中包含了其他组件，也就是组件嵌套场景，通常的处理办法会像backbone一样，在父组件中创建这个子组件，
并持有这个组件的引用，也就是会在适当的时候去创建、更新、销毁组件，那么当代码规模随着组件复杂度和组件状态复杂度变大时，这个组件的可维护性就会变差，进而增大组件间的耦合度，无法实现解耦。

**那么这就是一个问题**：
> 由于组件嵌套增大了组件间的耦合度，导致组件的可维护性变差

怎么解决这个问题的呢？React引入元素的概念，元素的引入就是为了解决上面的问题。

**元素是一个具有描述性的对象而非真正的实例**, 元素比DOM元素更轻量，因为它只是普通对象。
- 只是一个普通对象
- 可以使用对象来描述，也可以使用JSX来描述

## React中的元素树（利用组件来封装元素树）
React利用组件来封装元素树，背后会构建一个虚拟DOM也就是virtual DOM。

当React看到一个元素类型是**函数**或者**类**的时候，会去查询那个组件会渲染出什么元素，并且安排好对应的属性，直到知道页面中每一个组件对应的最根本的DOM标签。

> 对于一个React组件，属性是输入，元素树是输出

元素并不是一个真正的实例，而是一种用来告诉 React 你希望哪些东西显示在页面中的方式。你不能调用元素里的任何方法，因为它一个不可变的描述对象，包含两个属性：type(string | ReactClass) 和 props(Object)。

##### 1. 当元素的 type 是一个字符串，就代表 DOM 节点中对应的标签名，props 对应标签上的属性，这就是 React 将要渲染的内容。

```JavaScript
{
  type: 'button',
  props: {
      className: 'button button-blue',
      children: {
            type: 'b',
            props: {
                    children: 'OK!'
            }
      }
  }
}
```
这段元素的代码表示的是下面这样的HTML：
```html
<button class='button button-blue'>
    <b>
        OK!
    </b>
</button>
```
按照惯例，当我们需要构建元素树的时候，就会在父级元素的 props.children 中将子元素详列出来。

##### 2. 然而，元素的 type 也可以是一个表示 React 组件的函数或者类
```JavaScript
{
  type: Button,
  props: {
      color: 'blue',
      children: 'OK!'
  }
}
```
这就是 React 的核心思想了。

一个描述组件的元素也属于元素的范畴，就像一个描述 DOM 节点的元素一样，它们可以互相嵌套混搭使用。
在一棵 DOM 树中，可以混合使用匹配 DOM 节点或者 React 组件的元素。
```JavaScript
const DeleteAccount = () => ({
  type: 'div',
  props: {
  children: [{
        type: 'p',
        props: {
            children: 'Are you sure?'
        }
            
    }, {
        type: DangerButton,
        props: {
            children: 'Yep'
        }
    }, {
        type: Button,
        props: {
            color: 'blue',
            children: 'Cancel'
        }
    }]
  });
})
```
如果喜欢 jsx，还可以写成下面形式：

```JavaScript
const DeleteAccount = () => (
  <div>
    <p>Are you sure?</p>
    <DangerButton>Yep</DangerButton>
    <Button color='blue'>Cancel</Button>
  </div>
);

```
