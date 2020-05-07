# vue-property-decorator

在Vue中使用Typescript,通过vue-property-decorator装饰器库来简化书写。
 vue-property-decorator依赖vue-class-component

- @Component （from vue-class-component）
- @Emit
- @Prop
- @PropSync
- @Watch
- @Inject
- @Provide
- @Model
- @Ref
- Mixins（from vue-class-component）
- filters
- directives

#### @Component  声明组件



```jsx
import { Vue,Component } from 'vue-property-decorator'
import {componentA} from '@/components'
@Component({
    components: {componentA} // 其他组件声明
})
// @Component
export default class '组件名' extends Vue {
    private valueA: string = '....'  // data里的变量
    private valueB: number[] = [1,2,3]
    private get valueC () {  // 计算属性
        return 1
    }
    public render() {
        return (<div>...</div>)
    }
}
```

#### @Emit 事件触发

接受一个参数 event?: string, 如果没有的话会自动将 camelCase 转为 dash-case 作为事件名.

会将函数的返回值作为回调函数的第二个参数, 如果是 Promise 对象,则回调函数会等 Promise resolve 掉之后触发.

如果$emit 还有别的参数, 比如点击事件的 event , 会在返回值之后, 也就是第三个参数.

子组件:



```vue
<template>
  <div>
    <button @click="emitChange">Emit!!</button>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Emit } from "vue-property-decorator";

@Component
export default class EmitComponent extends Vue {
  private count = 0
  @Emit("button-click") private emitChange() {
    this.count += 1
    return this.count
  }
}
</script>
```

父组件, 父组件的对应元素上绑定事件即可:



```vue
<template>
  <EmitComponent v-on:button-click="listenChange" />
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator"

@Component({ components: { EmitComponent } })
export default class App extends Vue {
  private listenChange(value: number, event: any) {
    console.log(value, e)
  }
}
</script>
```

```kotlin
  // 子组件
  @Emit() // 参数为父组件绑定事件名，不写默认跟随的函数名称[这种情况下，camelCase名称将转换为kebab-case]
  private emitChange() {
     return 'something' // 如果想返回值，可以这边return 
  }

  @Emit('reset') 
  private emitChange1() {
     return 'something' // 如果想返回值，可以这边return 
  }

  // 父组件
  <child onReset={(res: any) => this.parentHandleChange(res)}>
```

#### @Watch 值变化监听



```csharp
private value: string = 'tt'
@Watch('value')
private valueWatch(newval: string, old: string) {}

@Watch('person', {immediate: true, deep: true})
private valueWatch(newval: Person, old: Person) {}
```

❣ watch方法需要声明在变量和jsx后

#### Prop 父组件传递数据

参数可以传

- Constructor 例如String, Number, Boolean
- Constructor[], 构造函数的队列, 类型在这队列中即可
- PropOptions
- - type 类型不对会报错 Invalid prop: type check failed for prop "xxx". Expected Function, got String with value "xxx".
  - default 如果父组件没有传的话为该值, 注意只能用这一种形式来表示默认值, 不能@Prop() name = 1来表示默认值 1, 虽然看起来一样, 但是会在 console 里报错, 不允许修改 props 中的值
  - required 没有会报错 [Vue warn]: Missing required prop: "xxx"
  - validator 为一个函数, 参数为传入的值, 比如(value) => value > 100

```tsx
  // child.tsx
  @Prop()
   public msg:string;
  @Prop(Number) prop1!: number;
  @Prop({
    type: String,
    default: 'default value',  // 一般为String, Number
    // 对象|数组，则默认值从函数返回
    // default: ()=>{ return [1,2] }
    required: false,
    validator: (value) => {
        return [1,2].indexOf(value) !== -1
    }
  }) prop2!: string;
  @Prop([string, Boolean]) prop3: string | boolean;
```

🎈 ！必定有值 ？可传参数

#### @PropSync

**@PropSync()** 与 Prop 的区别是子组件可以对 props 进行更改, 并同步给父组件,

子组件:



```vue
<template>
  <div>
    <p>{{count}}</p>
    <button @click="innerCount += 1">increment</button>
  </div>
</template>
<script lang="ts">
import { Component, Vue, PropSync } from "vue-property-decorator"

@Component
export default class PropSyncComponent extends Vue {
  // 注意@PropSync 里的参数不能与定义的实例属性同名, 因为还是那个原理, props 是只读的
  @PropSync('count') private innerCount!: number
}
</script>
```

父组件: 注意父组件里绑定 props 时需要加修饰符 .sync



```vue
<template>
  <PropSyncComponent :count.sync="count" />
</template> 

<script lang="ts">
import { Component, Vue, PropSync } from "vue-property-decorator"

@Component({ components: PropSyncComponent })

export default class PropSyncComponent extends Vue {
  // 注意@PropSync 里的参数不能与定义的实例属性同名, 因为还是那个原理, props 是只读的.
  @PropSync('count') private innerCount!: number
}
</script>
```

也可结合 input 元素的 v-model 绑定数据, 实时更新. 由读者自行实现.



```
@PropSync(propName: string, options: (propOptions | Constructor[] | Constructor) = {})
```



```tsx
  import { Vue, Component, Prop } from 'vue-property-decorator'

  @Component
  export default class YourComponent extends Vue {
    @PropSync('name', { type: String }) syncedName!: string
  }
```

等同于



```jsx
export default {
  props: {
    name: {
      type: String
    }
  },
  computed: {
    syncedName: {
      get() {
        return this.name
      },
      set(value) {
        this.$emit('update:name', value)
      }
    }
  }
}

//父組件.js
// 我们在使用@PropSync装饰器的时候，计算属性set方法抛出的事件名让我们无法直接以onEventName的形式书写
on={['update:name']: this.handle}
```
### @Watch


> **@Watch**监听属性发生更改时被触发.

可接受配置参数 options

- immediate?: boolean 是否在侦听开始之后立即调用该函数
- deep?: boolean 是否深度监听.



```vue
<template>
  <div>
    <button @click="person.name.firstName = '修'">change deeper</button>
    <button @click="person.name = '修'">change deep</button>
  </div>
</template>
<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator'

@Component
export default class PropSyncComponent extends Vue {  
  private person = { name: { firstName: '修' } }  
  @Watch('person', { deep: true })
    private firstNameChange(person: object, oldPerson: object) {   
    console.log(person, oldPerson)
  }
} 
 </script>
```

#### mixins



```tsx
  // mixins.ts
  export default class MyMixins extends Vue{
    value: string ='hi'
  }
  // 引入
  // 法一  vue-class-component提供
  import {mixins} from 'vue-class-component'
  import myMixins from 'mixin.ts'
@Componnet
  export default class myCom extends mixins(myMixins){
      public create () {
        console.log(this.value)
      }
  }

  // 法2 @Component中混入
  // mixins.ts
  declare module 'vue/types/vue' {
      interface Vue {
        value: string
    }
  }
  @Component
  export default class myMixins extends Vue {
    value: string = 'hello'
  }
 // 引入
  @Component({
    mixins: [myMixins]
  })
export default class myCom extends mixins(myMixins){
     public create () {
        console.log(this.value)
      }
  }
```

#### @Model

`vue`组件提供`model:{prop?:string, event?:string}`让我们可以定制`prop`和`event`,默认情况下，一个组件上的`v-model`会把value用作prop,把`input`用作 `event`,但是一些组件比如单选框和复选框可能想使用value prop来达到不同的目的，可以使用model选项回避冲突



```tsx
import { Vue, Component, Model} from 'vue-property-decorator';

@Component
export class myCheck extends Vue{
   @Model ('change', {type: Boolean})  checked!: boolean;
}
```

`@Model()`接收两个参数, 第一个是`event`值, 第二个是`prop`的类型说明, 与@Prop类似; 【🎈这里的类型要用JS的. 后面在接着是prop和在TS下的类型说明.】



```csharp
Vue.component('my-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    // this allows using the `value` prop for a different purpose
    value: String,
    // use `checked` as the prop which take the place of `value`
    checked: {
      type: Number,
      default: 0
    }
  },
  // ...
})
<my-checkbox v-model="foo" value="some value"></my-checkbox>
```

等同于



```csharp
<my-checkbox
  :checked="foo"
  @change="val => { foo = val }"
  value="some value">
</my-checkbox>
```

即foo双向绑定的是组件的checked, 触发双向绑定数值的事件是change

#### @Provide提供/@Inject注入

父组件不便于向子组件传递数据，就把数据通过provide传递下去，然后子组件通过Inject来获取



```kotlin
 // js写法
  data() {return {foo: 'foo'}}
  provide() {
    return { foo, this.foo}
  }
  inject: {
    
  }

  // ts
  import {Vue,Component,Inject,Provide} from 'vue-property-decorator'
@Component
export defalut class MyComponent extends Vue{ 
  @Inject()
  foo!: string
  @Inject('bar')
  bar!: string
  @Inject({ from:'optional', default:'default'})
  optional!: string;

  @Provide()
  foo = 'foo'
    
  @Provide('bar')
  baz = 'bar'
}
```

#### @Ref

**@Ref** 跟 react 中的一样, ref 是用于引用实际的 DOM 元素或者子组件.应尽可能避免直接使用, 但如果不得不用 ref 比 document 拿要方便很多, 参数传一个字符串refKey?:string, 注意这里如果省略传输参数, 那么会自动将属性名作为参数, 注意与@Emit的区别, @Emit在不传参数的情况下会转为 dash-case, 而 @Ref不会转, 为原属性名


(refKey?: string) decorator



```dart
import { Vue, Component, Ref } from 'vue-property-decorator'
 
import AnotherComponent from '@/path/to/another-component.vue'
 
@Component
export default class YourComponent extends Vue {
  @Ref() readonly anotherComponent!: AnotherComponent
  @Ref('aButton') readonly button!: HTMLButtonElement
}
```

等同于



```kotlin
export default {
  computed() {
    anotherComponent: {
      cache: false,
      get() {
        return this.$refs.anotherComponent as AnotherComponent
      }
    },
    button: {
      cache: false,
      get() {
        return this.$refs.aButton as HTMLButtonElement
      }
    }
  }
}
```



#### filters

filter 表示对数据的筛选, 跟 linux 中的管道符十分相似, 数据通过 filter 进行处理变成新的数据.注意, 在这里配置时一定要使用 filters, 不要忘了 s, 否则不会报错但是也没作用。



```vue
<template>
  <div>{{msg | filterA}}</div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

@Component({
  filters: {
    addWorld: (value: string) => `${value} world`,
  },
})
export default class App extends Vue {
  private msg = 'Hello' // filter 之后显示 hello world
}
</script>
```

#### directives

具体的介绍可以看 Vue 的[官方介绍](https://links.jianshu.com/go?to=https%3A%2F%2Fcn.vuejs.org%2Fv2%2Fguide%2Fcustom-directive.html) 简单来说就是 DOM 节点在一些特定钩子触发时添加一些额外的功能。

钩子函数有：

- bind 指令绑定到元素时调用, 可以进行一次性设置
- inserted 绑定元素被插入到父节点时调用
- update VNode 更新时调用
- componentUpdated VNode 及子 VNode 全部更新后调用
- unbind 元素解绑时调用

如果bind 和 update 时调用一样可以进行简写, 不用指定某个钩子

钩子函数参数:

- el 对应绑定的 DOM
- binding
- - name 指令名 v-demo="1+1" 为 "demo"
  - value 绑定的值 v-demo="1+1" 为 2
  - oldValue 在 update componentUpdated 可用
  - expression 字符串形式的表达式 v-demo="1+1" 为 "1+1"
  - arg 指令的参数 v-demo:foo 为 'foo', 注意要在 modifier 前使用 arg, 不然会将 arg 作为 modifier 的一部分, 如v-demo.a:foo arg 为 undefined, modifier 为{'a:foo': true}
  - modifiers 包含修饰符的对象 比如 v-demo.a 这个值为 {a:true}
- vnode vue 的虚拟节点, 可参考[源码](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Fblob%2Fdev%2Fsrc%2Fcore%2Fvdom%2Fvnode.js)查看可用属性
- oldVnode 上一个虚拟节点(update 和 componentUpdated 可用)

看个简单的实例:



```vue
<template>
  <span v-demo:foo.a="1+1">test</span>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

@Component({
  directives: {
    demo: {
      bind(el, binding, vnode) {
        console.log(`bindingName: ${binding.name}, value: ${binding.value}, args: ${binding.arg}, expression: ${binding.expression}`); // bindingName: demo, value: 2, args: foo, expression: 1+1
        console.log('modifier:', binding.modifiers); // {a:true}, 无法转为 primitive, 所以单独打印
      },
    },
    demoSimplify(el, binding, vnode) {
      // do stuff
    },
  },
})
export default class App extends Vue {}
</script>
```

[https://www.npmjs.com/package/vue-property-decorator](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fvue-property-decorator)