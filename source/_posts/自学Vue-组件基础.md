---
title: 组件基础
date: 2020-04-03 09:36:19
tags: [vue, js]
categories: 前端学习
toc: true
---

<!-- @format -->

### 组件基础基本使用

- 组件要有一个根元素把子元素包住
- 组件可以多次使用，而且组件里面数据都是当前组件独享的moment，不关别的组件的事，即使它们的名字是一样的
- 在使用模板字符串的时候，可以使用单引号、双引号、反引号（ ``），为了模板字符串格式好看，推荐使用反引号

#### 全局注册组件<!-- more -->

- 全局注册的组件可以用在任意的 Vue 根实例中（ new Vue ( { }) ）

- 使用 *Vue.component( "组件名", " { 模板字符串等内容 } ")*

  ```html
  <div id="app">
      <my-com></my-com>
  </div>
  <script>
  	Vue.component("myCom", {
          data(){
              return {
                  msg: "加油加油。"
              }
          }
          template: "<h1>我是一个模板字符串，学习很快乐！</h1>"
      })
      var vm = new Vue({
          el: "#app"
      })
  </script>
  ```

  - 在全局注册的时候使用的是 *驼峰法* 或者加 *-*  ，按时在使用的时候只能是加 *-* ，局部注册也一样

  - data 和实例 vue 不一样，这里的 data 是一个方法，返回一个对象，数据在里面

    

#### 局部注册组件

- 只能在当前注册了这个组件的 Vue 中使用

  ```html
  <div id="app">
      <my-com></my-com>
  </div>
  <script>
  	var login = {
          tempalte:`<div>
      			<h3>努力</h3>
  			<h3>奋斗</h3>
      		</div>`
      }
      var vm = new Vue({
          el: "#app",
          data: {},
          componentss: {
              "my-com": login
          }
      })
  </script>
  ```

  

### 父组件向子组件传值

> 使用 props

+ 在子组件定义一个 _props_ 可以用字符串数组的形式列出，也可以用对象的形式列出

+ 在需要绑定的组件上加上 _props_ 中写好的值，可以静态绑定，即是没有 _v-bind_ 你给它什么值就是什么值, 动态绑定 _v-bind_ 简写为,它的值是父组件 data 中的值

```html
<my-h1 title="msg"></my-h1>
<my-h1 :title="msg"></my-h1>
```

```javascript
Vue.component("myH1", {
    props: ["title", "title1"],
    template: `
		<div>
		  <h1>{{title}}</h1>
		  <h1>{{title1}}</h1>
		</div>
`
});
var vm = new Vue({
    el: "#app",
    data: {
        msg: "hello prop",
}
});
```
### 子组件向父组件传值

+ 子组件使用 *$emit* 触发事件

+ *$emit* 的第一个参数为自定义对象的名称，第二个参数是要传递的数据

+ 父组件使用 *v-on* 监听子组件的事件

  ```html
  <div id="app">
      <div :style="{fontSize: fontSize + 'px'}"> {{msg}} </div>
      <my-com @enlage-font='handle($event)'></my-com>
  </div>
  <script>
  	Vue.component("myCom", {
          template: `
  		<button @click="$emit('enlage-font', 10)">你点一下试试</button>
  `
      })
      var vm = new Vue({
          el: "#app",
          data: {
              msg: "加油加油“,
              fontSize: 10
          },
          methods: {
              handle: function(val){
                  this.fontSize += val;
              }
          }
      })
  </script>
  ```

  

  + 和使用 props 传值很像
  + 假如我有定义一个方法，需要修改页面的值，但是页面的值来自父组件。这是就需要使用事件抛出一个值

  在子组件中,比如我想删除列表中的一项数据（这个就好比是子组件中的*props*）,可以是表达式，也可以抛出一个对象（{id: id, type="del"}）

```javascript
del: function(id) {
    this.$emit("cart-del", id);
}
```

然后，父组件中(这个就好比是data中的数据)

```javascript
delCart: function(id){
    var index = this.goodList.findIndex(item => {
    return item.id == id;
});
    this.goodList.splice(index, 1);
}
```

最后就像*props*一样，在标签汇中绑定。（*props*用*v-bind*，这里用*v-on*）

```html
<cart-list @cartDel="del-cart($event)"></cart-list>
```

-----------------



### 组件插槽

- 自定义组件，在使用的时候不能在其中插入东西，所以要使用 *组件插槽* 
- 感觉就相当于一个占位符
- 当组件开始渲染的时候，*slot*  会被替换成 *芜湖* 

#### 匿名插槽

```html
<test>芜湖</test>
<script>
	Vue.component('test', {
        data(){
            return { name: "张三" }
        }
        template: `<div>
    		<strong>起飞~</strong>
			<slot><slot>
    </div>`
    })
</script>
```

- 插槽内不仅可以放文本，还可以放 HTML、其他自定义组件
- 如果没有 slot 标签，自定义的内容将不会显示

#### 插槽传值以及值得作用域

```html
<test url="/aoligei">
	{{ name }}
    {{ url }}
</test>
```

- 插槽和模板中的其他兄弟一样，他们能访问到相同作用域的值

- 上面的值中， *name* 是可以取得到值得，但是 *url* 不行，并且显示 undefined。因为 url 已经不是哥儿们了，url的值是传给 test 组件的

  > 父级模板里的所有内容都是在父级作用域中编译的；子模板里的所有内容都是在子作用域中编译的。

#### 后备内容

```html
<!-- 在body中-->
<test></test>
<!-- 在模板中-->
<button>
    <slot>Submit</slot>
</button>
```

- 如果就这样，显示的是 Submit，如果在 *test* 里面加上 save ，那 submit 就变成 save

#### 具名插槽

- 当使用了好多个插槽的时候，可以给插槽起个名字

  ```html
  <!-- 在模板中 -->
  <div>
      <header>
      	<slot name="header"></slot>
      </header>
      <session>
      	<slot></slot>
      </session>
      <footer>
      	<slot name="fotter"></slot>
      </footer>
  </div>
  ```

  - 不没 *name* 的 slot 被人取了个花名叫 "default"

- 使用的时候，要在标签中加一个 *template* 元素，并且在里面使用 **v-slot** 绑定，简写 **#**

  ```html
  <!-- 在body中(继续用上面的test组件做例子) -->
  <tset>
  	<template v-slot:header>
      	<h1>加油啊</h1>
    	</template>
    	<p>好好学习天天向上</p>
    	<template v-slot:footer>
      	<p>每天都是好心情</p>
    	</template>
  </tset>
  ```

  - 这样 slot 就为它命中注定的那个 template 占了位置

#### 作用域插槽

- 父组件对子组件加工处理

- 即可复用子组件的 slot，也可以使得 slot 的内容不一样

  ```html
  <!-- 在模板中 ○ -->
  <span>
  	<slot> {{user.lastName}} </slot>
  </span>
  ```

  - 然后，我不想使用 lastName 了，我想换个东西

    ```html
    <!-- 在body中 ※ -->
    <test> {{user.firstName}} </test>
    ```

    - *这样是不行的*，因为在 test 中，他们已经不是 slot 的好兄弟了

  - 为了让 *user* 在父级的插槽内容中可用，我们可以将 *user* 作为 *<slot>* 元素的一个 attribute 绑定上去：

    ```html
    <!-- 在模板中 ○ -->
    <span>
    	<slot v-bind:user="user"> {{user.lastName}} </slot>
    </span>
    ```

    - 这个绑定在 *slot* 上的属性叫作 **插槽prop**

  - 然后就可以使用带值的 v-slot 来定义 插槽prop 的名字了：

    ```html
    <!-- 在body中 ※ -->
    <test>
    	<template v-slot:default="slotProps">
        	{{ slotProps.user.first }}
        </template>
    </test>
    ```

    - *slotProps* 名字随便取
    - 因为 slot 没有名字，所以 这里使用了 *default* 作为 slot 的属性
    - 如果是没名字，可以直接使用不带参数，也就是直接 `v-slot="slotProps"` 

### 一些笔记

1. $event  
    通过在方法后面加个 *$event* 可以访问原生事件对象。比如：

```html
<input @blur="warn('hahaha', $event)">
```

  在js中定义方法warn

 ```javascript
 warn: function(msg, event){
     console.log(msg);
     console.log(event.target.value);
    //  获取到input的值
 }
 ```

2. 在字符串模板中，包含模板的是 *`* 斜点，而不是单引号或者双引号

3. findIndex和some是遍历数组的。

> 语法：array.findIndex(function (currentValue, index, arr), thisValue)  

> *currentValue* 是必选参数

```javascript
var index = this.goodList.findIndex(item => {
    return item.id == id;
}
```
