## Build My Own React

- [original](https://pomb.us/build-your-own-react/)
- [translation](https://www.notion.so/byzhang/Build-your-own-React-43b3bec3e45c47caa222a8962c9888a0)

### Notes

#### v1:

- 渲染文本节点使用的是 `createTextNode` 而不是 `innerText`, 可以保证在后续处理元素时, 所有元素的形式都是一致的, 抹平差异, 统一处理. 

- 优化: `render` 的过程(见以下代码)会阻碍浏览器的主进程, 比如用户输入或者动画效果等, 因此引入 **concurrent mode** 的概念和 **fiber** 数据结构, 将渲染的过程拆分成多个小单元. 
```js
element.props.children.forEach(child => render(child, dom));
```

#### v2:

- `render`首次执行, workloop 开始,

- `performUnitOfWork_v1`

```js
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
​
  const elements = fiber.props.children
  let index = 0
	let prevSibling = null
	
​	 // prevSibling ?
	// And we add it to the fiber tree setting it either as a child or as a sibling, depending on whether it’s the first child or not.
	
	while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
​
    if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }
​
    prevSibling = newFiber
    index++
  }
​
  if (fiber.child) {
    return fiber.child
  }
	let nextFiber = fiber
	
	// find child -> sibling -> uncle
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}
```
`performUnitOfWork_v1` 有个问题, 以下操作会导致, 渲染流程结束之前, 浏览器主流程依然被阻滞. 借用 `wipRoot` fiber 来解决这个问题. [performUnitOfWork_v2](./src/index_v2.jsx#L132)

```js
if (fiber.parent) {
	fiber.parent.dom.appendChild(fiber.dom)
}
```

#### [v3](./src/index.jsx)



### TODO

- [ ] add types,  remove @types/react