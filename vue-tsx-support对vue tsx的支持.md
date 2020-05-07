[wonderful-panda/vue-tsx-support](https://link.zhihu.com/?target=https%3A//github.com/wonderful-panda/vue-tsx-support)

vue-tsx-support 主要是用于支持再Vue的渲染函数中使用TSX，Vue本生是支持render JSX渲染的，在不涉及自定义Prop、事件时不使用该库也可以。但如果component中有自定义prop，event，TS的类型检查就会报错。大致逻辑是在原有的Vue类上有包装了一层，将属性、事件、作用域插槽以泛型方式传入接口定义，再将接口定义应用到TSX中。