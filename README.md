# VTree

采用虚拟滚动树实现的处理大数据量的Vue树组件。

## 简介
有些情况下，可能需要渲染数据量大的树形结构，而传统递归生成树结构的方法会导致 `DOM` 节点暴增，渲染、更新都会变慢。该组件借鉴网上虚拟滚动树的实现方法，实现了和 `element-ui` API 基本一致的能够应对较大数据量的树组件，**建议在数据量不是很大、体验没有直线下滑的情况下，还是使用 `element-ui` 的树组件**。

## 原理

1. 虚拟滚动(就是为了减少 DOM 渲染，减少创建 DOM 开销)
2. 平级 DOM 结构，通过 indent 区分树的层级关系

## 内部 node 结构

```javascript
{
  	key,
    // 原始数据
    originNode,
    parent,
    children: [],
    // 当前节点层级，用于计算 indent
    level,
    expanded,
    visible,
    checked: false,
    // 半选状态
    indeterminate: false,
    filtered: false,
    hasFilteredChildren: false,
    selected: false,
    loading: false,
    loaded: false,
    isDisabled: false,
    isLeaf: false
}
```


## props

| prop 名字             | 数据类型                        | 描述                                       | 是否必传 |
| ------------------- | :-------------------------- | ---------------------------------------- | ---- |
| data                | Array                       | 和 element 一样，传入树数据                       | 是    |
| height              | Number \| String            | 用于指定组件根节点的高度，默认为 100%                    | 否    |
| nodeHeight          | Number                      | 用于指定树每个节点的高度，默认为 26                      | 否    |
| nodeKey             | String                      | 用于指定标识树的每一个节点，参考 element 的 nodeKey       | 否    |
| props               | Object                      | 用于标识每个树节点的状态及如何找到节点的子节点，参考 element 的 props，默认值为 { label: 'label', children: 'children' }**disabled 功能暂未实现** | 否    |
| load                | Function(data, resolve)     | 懒加载调用的方法，参考 element 的 load prop          | 否    |
| lazy                | Boolean                     | 指定是否是懒加载，默认为 false，只有为 true 的时候才是懒加载模式   | 否    |
| defaultExpandAll    | Boolean                     | 是否默认展开全部节点，默认为 false                     | 否    |
| expandOnClickNode   | Boolean                     | 是否在点击节点的时候展开节点，默认为 true                  | 否    |
| showCheckbox        | Boolean                     | 是否显示勾选框，默认为 false                        | 否    |
| checkStrictly       | Boolean                     | 是否严格遵守父子节点勾选不关联，默认为 false，参考 element 的checkStrictly | 否    |
| defaultCheckedKeys  | Array                       | 默认选中的节点的 keys 数组，参考 element 的 defaultCheckedKeys | 否    |
| defaultExpandedKeys | Array                       | 默认展开的节点的 keys 数组，参考 element 的 defaultExpandedKeys，**暂未实现 element 的 auto-expand-parent** | 否    |
| accordion           | Boolean                     | **暂未实现**                                 | 否    |
| indent              | Number                      | 层级的水平缩进，默认为 16，参考                        | 否    |
| filterNodeMethod    | Function(value, data, node) | 过滤节点时使用的算法，返回 true 则表示该节点可以显示，参考 element 的 filterNodeMethod | 否    |
| emptyText           | String                      | 没有数据时显示的文本，默认为暂无数据                       | 否    |
| expandedIcon        | String                      | 节点展开时的左侧 icon class                      | 否    |
| collapsedIcon       | String                      | 节点收起时的左侧 icon class                      | 否    |
| filterCheckStrictly | Boolean                     | 当过滤树的时候，勾选节点时是否勾选被过滤掉的节点，若为 true，则只会勾选被过滤出来的节点，隐藏的节点不会被勾选 | 否    |

## 方法

| 方法名                 | 描述                                   | 参数                                       |
| ------------------- | ------------------------------------ | ---------------------------------------- |
| setCheckedKeys      | 通过 keys 设置当前选中的节点，**需要设置 node-key**  | (keys, leafonly = false)，1. 要勾选的 keys 数组，2.是否只勾选叶子节点 |
| setCheckedNodes     | 通过 nodes 设置当前选中的节点，**需要设置 node-key** | (nodes)，1. 要勾选的 nodes                    |
| getCheckedKeys      | 获取当前被选中的节点的 keys，**需要设置 node-key**   | (leafonly = false)，1. 是否只获取叶子节点          |
| getCheckedNodes     | 获取当前选中的节点的 nodes，**需要设置 node-key**   | (leafonly = false)，1. 是否只获取叶子节点          |
| setChecked          | 根据 data 或者 key 设置节点的选中状态             | (keyOrData, checked, deep = false)，1. data、key，2. 选中状态，3. 是否更改子节点的选中状态 |
| getHalfCheckedNodes | 获取半选中状态的节点数据                         |                                          |
| getHalfCheckedKeys  | 获取半选中状态的节点的 keys                     |                                          |
| getCurrentKey       | 获取当前被选中节点的 key，若没有则返回 null           |                                          |
| getCurrentNode      | 获取当前被选中节点的数据，若没有则返回 null             |                                          |
| setCurrentKey       | 通过 key 设置当前被选中的节点                    | (key)，1. 要选中节点的 key                      |
| setCurrentNode      | 通过数据设置当前被选中的节点                       | (node)，1. 要选中节点的数据                       |
| filter              | 对树进行筛选操作                             | 接受一个参数作为 filterNodeMethod 的第一个参数         |


## 事件

| 事件名            | 描述          | 参数                                       |
| -------------- | ----------- | ---------------------------------------- |
| current-change | 当前选中节点变化时触发 | 1. 被点击的节点的数据，2. 被点击的节点的 node             |
| node-click     | 节点被点击时触发    | 1. 被点击的节点的数据，2. 被点击的节点的 node             |
| node-expand    | 节点被展开时触发    | 1. 被点击的节点的数据，2. 被点击的节点的 node             |
| node-collapse  | 节点被收起时触发    | 1. 被点击的节点的数据，2. 被点击的节点的 node             |
| check          | 复选框被点击时触发   | 1. 节点的数据，2. { checkedNodes, checkedKeys, halfCheckedNodes, halfCheckedKeys } |
| check-change   | 节点选中状态改变时触发 | 1. 节点的数据，2. 节点选中状态，3. 时候有子节点被选中          |

**由于使用虚拟滚动，导致无法监听每个数据的变化，所以 check-change 事件可能会触发多次**

**事件触发可能有偏差**


## 插槽

### content

该插槽用于自定义节点内容，插槽传入 data、node 两个属性

```html
<v-tree>
	<template v-slot="{ data, node }">
  		<span>{{ data.label }}</span>
  	</template>
</v-tree>
```

### empty

该插槽用于自定义没有数据时展示的内容

```html
<v-tree>
	<template v-slot:empty>
  		<span>没有数据，难受</span>
  	</template>
</v-tree>
```

### expanded-icon/collapsed-icon

该插槽用于自定义节点 展开/收起 时的 icon

```html
<v-tree>
	<template v-slot:expanded-icon>
  		<img src="xxxx" />
  	</template>
  
  	<template v-slot:collapsed-icon>
  		<img src="xxxx" />
  	</template>
</v-tree>
```
