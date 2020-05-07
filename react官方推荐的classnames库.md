# react官方推荐的classnames库

原文地址：https://www.jianshu.com/p/23accf80a850

# 一、为什么使用classnames这个库

在react开发中，我们有的时候需要使用js来动态判断是否为组件添加class（类名），这里我们使用到了classnames

# 二、学习网址

> https://www.npmjs.com/package/classnames
>  https://github.com/JedWatson/classnames

# 三、安装与引入

安装



```undefined
npm install classnames --save
```

引入



```jsx
import classnames  from ‘classnames’;
```

# 四、使用示例



```kotlin
<Button className={classnames({
    //这里可以根据各属性动态添加，如果属性值为true则为其添加该类名，如果值为false，则不添加。这样达到了动态添加class的目的
      base: true,
      inProgress: this.props.store.submissionInProgress,
      error: this.props.store.errorOccurred,
      disabled: this.props.form.valid,
    })}>
<Button/>
```