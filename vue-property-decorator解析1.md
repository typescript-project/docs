### 什么是Typescript

TypeScript 是一种由微软开发的自由和开源的编程语言，它是 JavaScript 的一个超集，扩展了 JavaScript 的语法。作者是安德斯大爷， Delphi、C# 之父（你大爷永远是你大爷）。把弱类型语言改成了强类型语言，拥有了静态类型安全检查， IDE 智能提示和追踪，代码重构简单、可读性强等特点。
 现在VUE 也支持了 TypeScript ，面对安德斯大爷放出的这些大招，果断用之。

### 安装使用

使用 vue-cli 创建项目的时候 选择Typescript就行了，
 注意下几个配置文件





![img](https:////upload-images.jianshu.io/upload_images/8145282-ab52eb2486b06a14.png?imageMogr2/auto-orient/strip|imageView2/2/w/214/format/webp)

image.png



tsconfig.json



```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "strict": true,
    "jsx": "preserve",
    "importHelpers": true,
    "moduleResolution": "node",
    "experimentalDecorators": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "sourceMap": true,
    "baseUrl": ".",
    "types": [
      "webpack-env"
    ],
    "paths": {
      "@/*": [
        "src/*"
      ]
    },
    "lib": [
      "esnext",
      "dom",
      "dom.iterable",
      "scripthost"
    ]
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.tsx",
    "src/**/*.vue",
    "tests/**/*.ts",
    "tests/**/*.tsx"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

tslint.json



```json
{
  "defaultSeverity": "warning",
  "extends": [
    "tslint:recommended"
  ],
  "linterOptions": {
    "exclude": [
      "node_modules/**"
    ]
  },
  "rules": {
    "quotemark": [true, "single"],
    "indent": [true, "spaces", 2],
    "interface-name": false,
    "ordered-imports": false,
    "object-literal-sort-keys": false,
    "no-consecutive-blank-lines": false,
    "no-console": false, //允许使用console
    "member-access": [true, "no-public"], //禁止指定公共可访问性，因为这是默认值
    // "noImplicitAny": false, //允许参数而不声明其类型
    "one-variable-per-declaration": false, //允许在同一声明语句中使用多个变量定义
    "no-unused-expression": [true, "allow-fast-null-checks"], //允许使用逻辑运算符执行快速空检查并执行副作用的方法或函数调用（ 例如e && e.preventDefault（））
    "curly": [true, "ignore-same-line"],
    "arrow-parens": [true, "ban-single-arg-parens"],
    "semicolon": [true, "never"],//是否提示不必要的分号
    "trailing-comma": [
      true,
      {
        "multiline": {
          "objects": "ignore",
          "arrays": "ignore",
          "functions": "ignore",
          "typeLiterals": "ignore"
        },
        "esSpecCompliant": true
      }
    ]
  }
}
```

### 重要的是怎么在项目中使用Typescrit写法



1:安装npm install --save vue-property-decorator
 此类库提供了7个装饰器

- @Component （from vue-class-component）
- @Emit
- @Inject
- @Model
- @Prop
- @Provide
- @Watch
- @Component
   实现生成像原生 JavaScript class 那样的声明组件。

下面分别给出实例解释其用法：

- @Component
   组件声明
   原生写法



```jsx
import UploadImage from '@/components/UploadImage'

export default {
  name: 'user',
  components: { UploadImage },
  data() {
    return {
      name:"张三",
      sex: '男'
    }
  },
  methods: {
    funcA(params) {},
    funcB() {}
  }
}
```

使用Ts中写法



```tsx
import UploadImage from '@/components/UploadImage'
import { Component, Vue, Provide } from 'vue-property-decorator'

@Component(
    name:"user",
    components:{
    	UploadImage // 其他组件声明
    }
)

export default class user extends Vue{
    private name:string="张三"
    private sex:string="男"

    private funcA(params:any){}
    private funcB(){}
}
```

其中使用 @Component 声明了 user组件 ，同时引用 子组件 UploadImage，写在 Components 参数中。

- @Prop
   属性声明 在自定义组建中使用，属性的类型需要定义，不然会报错，可以参见 《vue-tsx-support对vue tsx的支持.md》
   原生写法



```tsx
export default{
    name:"upload",
    props:{
          value:{
               type:String,
               default:''
          }
    }
}
```

在ts中写法



```dart
@Component()
export default class upload extends Vue{
       @Prop()
       private value:string='';
}
```

- computed
   计算属性
   这个很类似于c#中的 属性概念，属性值本身可以通过计算得出。

原生写法



```kotlin
computed: {
    imageUrl() {
      return 'http://xxxx.xxxx.com/' + this.value;//value是定义的一个字段
    }
  },
```

在ts中写法



```csharp
get imageUrl(){
      return 'http://xxxx.xxxx.com/' + this.value;//value是定义的一个字段
}

template 中一样使用{{imageUrl}}
```

- @watch
   用来监测Vue实例上的数据变动
   如果对应一个对象，键是观察表达式，值是对应回调，值也可以是方法名，或者是对象，包含选项。



```kotlin
  export default {
    name: 'index',
    data() {
      return {
        demo: {
          name: ''
        },
        value: ''
      };
    },
    computed: {
      newName() {
        return this.demo.name;
      }
    },
    watch: {
      newName(val) {
        this.value = val;
      }
    }
  };
```

ts写法



```kotlin
export default class index extends Vue{
        demo:any={name:''};
        value:string='';

        get newName(){  
              return this.demo.name;
        }

        @watch('wnewName')
        wnewName(val){
                  this.value=val;
        }
}
```

- emit
   我们知道，父组件是使用 props 传递数据给子组件，但如果子组件要把数据传递回去，应该怎样做？那就是自定义事件！

每个 Vue 实例都实现了[事件接口(Events interface)](https://links.jianshu.com/go?to=http%3A%2F%2Fcn.vuejs.org%2Fv2%2Fapi%2F)，即：



```bash
-   使用 $on(eventName) 监听事件
-   使用 $emit(eventName)触发事件
```



```jsx
Vue.component('counter', {
  template: `
  <button v-on:click="increment">{{ counter }}</button>`,
  data() {
    return {
      counter: 0
    }
  },
  methods: {
    increment: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
});

new Vue({
  el: '#example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})

<div id="example">
  <p>{{ total }}</p>
  <counter v-on:increment="incrementTotal"></counter>
</div>
```

子组件自定义了个事件，然后把这个事件发射出去，父组件使用这个事件