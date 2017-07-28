title: React 一个构建用户界面的库
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
> **这种混搭模式能保持组件之间相互解耦，因为可以通过组合唯一地表达 `is-a` 和 `has-a` 两种关系**

Button 是一个有指定属性的 DOM `button` 元素
DangerButton 是一个有指定属性的 `Button` 元素
DeleteAccount 在一个 <div> 内包含了 `Button` 和 `DangerButton` 元素

当 React 看见一个元素的类型是函数或者类的时候，它知道去问那个组件会渲染出什么元素，并安排好对应的属性。

当它看见这样的元素时：

```JavaScript
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```
React 将会询问 Button 渲染出什么元素，然后 Button 就会返回这个元素：

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
React 会不断重复这个过程，直到知道页面中每一个组件最根本的 DOM 标签元素为止。

**组件可以是类或者函数**

可以写成函数的形式，也可以是继承于 React.Component 的类。下面三种声明一个组件的方式大部分是等价的：
```JavaScript
// 1) As a function of props
const Button = ({ children, color  }) => ({
  type: 'button',
  props: {
      className: 'button button-' + color,
      children: {
        type: 'b',
        props: {
            children: children
        }
      }
  }
});

// 2) Using the React.createClass() factory
const Button = React.createClass({
    render() {
        const { children, color  } = this.props;
        return {
          type: 'button',
          props: {
                className: 'button button-' + color,
                children: {
                    type: 'b',
                    props: {
                        children: children
                    }
                }
          }
        };
   }
});

// 3) As an ES6 class descending from React.Component
class Button extends React.Component {
    render() {
        const { children, color  } = this.props;
        return {
          type: 'button',
          props: {
            className: 'button button-' + color,
            children: {
                type: 'b',
                props: {
                    children: children
                }
            }
          }
        };
   }
}

```
当一个组件被声明为一个类时，它比函数式组件稍微强大一些，因为可以保存一些本地状态以及在生命周期函数内执行自定义逻辑等。函数式组件没那么强大，但胜在简单，它就像一个只有 render() 方法的组件类。除非你需要那些类才能提供的特性，否则用函数式组件就好了。

React 会首先询问**父组件**它会返回什么形式的元素树，并配齐需要的属性。它会逐步把你的元素树分解成更小的颗粒。这个过程被 React 称为调度，这个过程始于你调用 ReactDOM.render() 或者 setState() 。在调度结束时，React 就能获得整棵 DOM 树，然后 react-don 或者 react-native 这样的渲染器将在需要更新的时候执行最少的 DOM 操作。
这个逐步提炼的过程正是 React APP 易于优化的原因，如果组件树的某些部分规模太大，React 访问效率不高，那么你可以告诉 React 如果属性没有变化就略过这部分组件树的提炼操作。当属性是不可变的数据时，可以很快判断它是否发生变化。因此，React 和不可变数据是一对天生的好基友，对于优化性能有事半功倍的效果。

> 元素就是一个用语描述出现在页面中的 DOM 节点或者 React 组件的纯对象。元素可以在自己的属性中包含其它元素。创建一个元素的成本很低，一旦元素被创建之后，就不再发生变化。
> React 组件可以用好几种方式声明，可以是一个包含 render() 方法的类，也可以是一个简单的函数，不管怎么样，它都是以属性作为输入，返回元素树作为输出。
> 当一个组件被注入一些属性值时，属性值来源于它的父级元素，所以人们常说，属性在 React 中是单向流动的：从父级到子元素。
> 所谓的实例，就是你在组件类中用 this 引用的那个对象，对于保存本地状态以及介入生命周期函数是有用的。
> 函数式组件没有实例，类组件才有，但你从来不需要手动创建，React 会帮你搞定。



