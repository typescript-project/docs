# [如使用Typescript撸Vue（Vue2 + TS +TSX+CSS module）](https://www.cnblogs.com/vaynewang/p/10794128.html)



原文地址：https://www.cnblogs.com/vaynewang/p/10794128.html

Vue对TS的支持一致不太好，连Vue作者尤大也自嘲真香压错了宝。期待Vue3.0会用TS重构且会有较大改进。不过目前有一些第三方的库可以曲线优化对TS的支持。主要就介绍下过下面两个库来写Vue。

总体体验尚可，类型检查，智能提示该有的都有，顺滑中带着一丝蹩脚。
如果要支持组件Props的类型检查及智能提示，则必须放弃template通过render写TSX, 总有种写React的感觉。

## 介绍

[kaorun343/vue-property-decorator](https://link.zhihu.com/?target=https%3A//github.com/kaorun343/vue-property-decorator)

vue-property-decorator 定义了很多的装饰器，如 @prop,@component，@watch 等。已经相当齐全了，不多介绍了，而且此库已经集成进了 vue-cli 3.0中，通过cli创建的项目也集成demo页面。

[wonderful-panda/vue-tsx-support](https://link.zhihu.com/?target=https%3A//github.com/wonderful-panda/vue-tsx-support)

vue-tsx-support 主要是用于支持再Vue的渲染函数中使用TSX，Vue本生是支持render JSX渲染的，在不涉及自定义Prop、事件时不使用该库也可以。但如果component中有自定义prop，event，TS的类型检查就会报错。大致逻辑是在原有的Vue类上有包装了一层，将属性、事件、作用域插槽以泛型方式传入接口定义，再将接口定义应用到TSX中。

CSS Module

Vue 默认是 scoped 方式引入css ，防止样式污染 ，通过vue模板使用也很方便。实际CSS 选择器使用 scoped 这种方式效率低于 CSS Module，使用TSX渲染时样式也只能通过CSS Module这样方式引用。这里再介绍个库 classNames ，通过这个库可以方便的组合样式名。

## 创建项目

使用vue-cli 3.0 创建一个项目 ， 必选 typescript Babel ，其他根据需要选。创建完成后已经引入了Vue 及 TS 相关包了，也包括上面提到的 vue-property-decorator。包含了一个实例代码，npm install，npm run serve 已经可以跑起来了。

## 导入和配置

\1. 安装 vue-tsx-support 包

```
`npm ``install` `vue-tsx-support --save`
```

　　

\2. 导入TS声明，有两种方式

编辑tsconfig.js

```
`  ``...``"include"``: [``    ``"node_modules/vue-tsx-support/enable-check.d.ts"``, ``    ``"src/**/*.ts"``,``    ``"src/**/*.tsx"``,``    ``"src/**/*.vue"``,``    ``"tests/**/*.ts"``,``    ``"tests/**/*.tsx"``  ``]``// 注意：将exclude内的 "node_modules" 删掉，不然永远也无法被引用到了``  ``...`
```

　　

或者在main.js中 import

```
import "vue-tsx-support/enable-check";
```

 

\3. 删除根目录下的 shims-tsx.d.ts ，否则会报重复定义的错误。

\4. 根目录新建 vue.config.js

```
`module.exports = {``  ``css: {``    ``modules: ``true` `// 开启CSS module``  ``},``  ``configureWebpack: {``    ``resolve: {``      ``extensions: [``".js"``, ``".vue"``, ``".json"``, ``".ts"``, ``".tsx"``] ``// 加入ts 和 tsx``    ``},``  ``},``  ``devServer: {``    ``port: 8800 ``// webpack-dev-server port``  ``}``};`
```

　　

## 创建视图组件

来创建个按钮组件：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 import { Component, Prop } from "vue-property-decorator";
 2 import * as tsx from "vue-tsx-support";
 3 
 4 export enum ButtonType {
 5   default = "default",
 6   primary = "primary"
 7 }
 8 
 9 export enum ButtonSize {
10   large = "large",
11   small = "small"
12 }
13 export interface IButtonProps {
14   type?: ButtonType;
15   size?: ButtonSize;
16   num: number;
17 }
18 
19 @Component
20 export default class Button extends tsx.Component<IButtonProps> {
21   @Prop() public type!: ButtonType;
22   @Prop() public size!: ButtonSize;
23   @Prop({ default: 0 }) public num!: number;
24 
25   protected render() {
26     return (
27       <div>
28         <p>id:{this.num}</p>
29         {this.type && <p>type:{this.type}</p>}
30         {this.size && <p>size:{this.size}</p>}
31       </div>
32     );
33   }
34 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

再创建Container 用TSX引用组件Button：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
import { Component, Prop } from "vue-property-decorator";
import { Component as tsc } from "vue-tsx-support";
import Button, { ButtonType, ButtonSize } from "./button";

interface IContainerProps {
  name?: string;
}

@Component
export default class Container extends tsc<IContainerProps> {
  @Prop() public name!: string;

  protected render() {
    return (
      <div>
        <p>container Name:{this.name}</p>
        <p>{this.$slots.default}</p>
        <p>
          button:
          <Button num={9} type={ButtonType.primary} size={ButtonSize.large} />
        </p>
      </div>
    );
  }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

此时即使在Container 的 Render 方法同样会对 Props 进行类型检查 ，而VS Code也有智能提示和自动引入，这体验棒极了。

## CSS Module 导入样式

注意：使用CSS module 前 需要在vue.config.js 中配置 css.modules = true

注意：如要添加全局样式可在 App.vue 中 @import 方式引用公用样式，这样就不会被CSS Module 加上后缀了。

然后加入TS定义

```
`declare module ``"*.scss"` `{``  ``const content: any;``  ``export` `default` `content;``}`
```

　　

可以配合classnames库，更方便的操作CSS类

[classnameswww.npmjs.com](https://link.zhihu.com/?target=https%3A//www.npmjs.com/package/classnames)

## 示例：

[vaynewang/SampleCode/vue-tsx-sample/](https://link.zhihu.com/?target=https%3A//github.com/vaynewang/SampleCode/tree/master/vue-tsx-sample)