# 一、JavaScript

DOM是一个树形结构。操作一个DOM节点实际上就是这么几个操作：

- 更新：更新该DOM节点的内容，相当于更新了该DOM节点表示的HTML的内容；
- 遍历：遍历该DOM节点下的子节点，以便进行进一步操作；
- 添加：在该DOM节点下新增一个子节点，相当于动态增加了一个HTML节点；
- 删除：将该节点从HTML中删除，相当于删掉了该DOM节点的内容以及它包含的所有子节点。



DOM节点是指Element，但是DOM节点实际上是Node，在HTML中，Node包括Element、Comment、CDATA_SECTION等很多种，以及根节点Document类型，但是，绝大多数时候我们只关心Element，也就是实际控制页面结构的Node，其他类型的Node忽略即可。根节点Document已经自动绑定为全局变量document。



document对象表示当前页面。由于HTML在浏览器中以DOM形式表示为树形结构，document对象就是整个DOM树的根节点。



最常用的方法是

- document.getElementById()
- document.getElementsByTagName()
- CSS选择器document.getElementsByClassName()。

document.getElementsByTagName()和document.getElementsByClassName()总是返回一组DOM节点。要精确地选择DOM，可以先定位父节点，再从父节点开始选择，以缩小范围。





ES6标准新增了一种新的函数：Arrow Function（箭头函数）。`x => x * x` 箭头函数相当于：`function (x) { return x * x; }`



如果仔细观察一个Form的提交，你就会发现，一旦用户点击“Submit”按钮，表单开始提交，浏览器就会刷新页面，然后在新页面里告诉你操作是成功了还是失败了。如果不幸由于网络太慢或者其他原因，就会得到一个404页面。

这就是Web的运作原理：一次HTTP请求对应一个页面。

如果要让用户留在当前页面中，同时发出新的HTTP请求，就必须用JavaScript发送这个新请求，接收到数据后，再用JavaScript更新页面，这样一来，用户就感觉自己仍然停留在当前页面，但是数据却可以不断地更新。

## 1、this 关键词
在JavaScript中，this 是一个特殊的关键字，它在不同的上下文中具有不同的值。this 的值取决于函数是如何被调用的。

- **全局上下文中**： 当在全局上下文中使用 this 时，它指向全局对象，在浏览器环境中通常是 window 对象。

  ```js
  console.log(this); // 指向全局对象（在浏览器中通常是 window）
  ```

- **函数内部**：
  
  - **作为普通函数调用**
  
    如果函数是作为普通函数调用的，this 将指向全局对象（在浏览器中通常是 window）
  
    ```js
    function myFunction() {
        console.log(this);
    }
    myFunction(); // 指向全局对象
    ```
  
  - **作为对象的方法调用**
  
    如果函数是作为对象的方法调用的，this 将指向调用该方法的对象。

    ```js
    var obj = {
        myMethod: function() {
            console.log(this);
        }
    };
    obj.myMethod(); // 指向 obj
    ```

  - **使用 call 或 apply 方法**
  
    可以显式地设置函数的上下文。

    ```js
    function myFunction() {
        console.log(this);
    }
    var myObject = { name: "Object" };
    myFunction.call(myObject); // 指向 myObject
    ```
  
- **事件处理函数中** 

    当在事件处理函数中使用 this 时，它通常指向触发事件的元素。

    ```js
    document.getElementById("myButton").addEventListener("click", function() {
        console.log(this); // 指向触发点击事件的按钮元素
    });
    ```

- **箭头函数中**

    箭头函数的 this 始终指向定义函数时的上下文，而不是调用时的上下文。

    ```js
    var myObject = {
        myMethod: function() {
            var myArrowFunction = () => {
                console.log(this);
            };
            myArrowFunction();
        }
    };
  myObject.myMethod(); // 指向 myObject
  ```

## 2、jQuery 中的this与$(this)

在 jQuery 中，this 关键字与原生 JavaScript 中的一些情况相似。行为也取决于上下文，但主要是与事件处理函数和遍历集合时的上下文有关。

- 事件处理函数中： 当你使用 jQuery 绑定事件处理函数时，this 通常指向触发事件的 DOM 元素。

  ```js
  $("button").click(function() {
      console.log(this); // 指向触发点击事件的按钮元素
  });
  ```

- 遍历集合中： 当你使用 jQuery 的遍历方法（如 each）时，this 通常指向当前正在迭代的元素。

  ```bash
  $("li").each(function() {
      console.log(this); // 指向当前正在迭代的 <li> 元素
  });
  ```

**而 `$(this)` 是 jQuery 提供的用于将当前 DOM 元素包装为 jQuery 对象的语法。在事件处理函数中，`this` 是事件触发元素的原生 DOM 引用，而 `$(this)` 是通过 jQuery 包装后的对象，可以方便地使用 jQuery 方法。**

```js
$("button").click(function() {
    // 使用 $(this) 来引用当前 DOM 元素的 jQuery 对象
    var $button = $(this);
    $button.text("Button Clicked!");
});
```

# 二、jQuery

## 1、简介

### ①能做哪些事情

- 消除浏览器差异：你不需要自己写冗长的代码来针对不同的浏览器来绑定事件，编写AJAX等代码；
- 简洁的操作DOM的方法：写$('#test')肯定比document.getElementById('test')来得简洁；
- 轻松实现动画、修改CSS等各种操作。

$是著名的jQuery符号。实际上，jQuery把所有功能全部封装在一个全局变量jQuery中，而$也是一个合法的变量名，它是变量jQuery的别名：

$本质上就是一个函数，但是函数也是对象，于是$除了可以直接调用外，也可以有很多其他属性。

注意，你看到的$函数名可能不是`jQuery(selector, context)`，因为很多JavaScript压缩工具可以对函数名和参数改名，所以压缩过的jQuery源码$函数可能变成`a(b, c)`

### ②jQuery和DOM对象互相转化

```bash
var div = $('#abc'); *// jQuery*对象
var divDom = div.get(0); *//* 假设存在*div*，获取第*1*个*DOM*元素
var another = $(divDom); *//* 重新把*DOM*包装为*jQuery*对象
```

通常情况下你不需要获取DOM对象，直接使用jQuery对象更加方便。如果你拿到了一个DOM对象，那可以简单地调用$(aDomObject)把它变成jQuery对象，这样就可以方便地使用jQuery的API了。

### ③引入

```html
<! -- CDN链接引入 -->
<script src="https://code.jquery.com/jquery-3.6.4.min.js"></script>
<! -- 本地引入 -->
<script src="path/to/jquery-3.6.4.min.js"></script>
```

## 2、选择器

jQuery的选择器就是帮助我们快速定位到一个或多个DOM节点。

如果查询的节点不存在，jQuery返回的对象是 `[]` ，不会返回`undefined`或者`null`，这样的好处是不必在下一行判断 `if (div === undefined)`。

- **按ID查找**：`$('#abc')`

- **按tag查找**：`$('table')`

- **按class查找**：`$('.button')`

- **组合查找**

  - tag组合ID：`$('table#test_table1')`查找页面两个 Tables 中的一个

- **多项选择器**

  多项选择器就是把多个选择器用`,`组合起来一块选。选出来的元素是按照它们在HTML中出现的顺序排列的，而且不会有重复元素。

  - `$('table#projects_mr_table,input')`查找页面两个 Tables 中的一个和页面中的 input 元素

- **层级选择器（Descendant Selector）**：`$('ancestor descendant')`

  如果两个DOM元素具有层级关系，就可以用`$('ancestor descendant')`来选择，层级之间用空格隔开。
  
  层级选择器相比单个的选择器好处在于，它缩小了选择范围，因为首先要定位父节点，才能选择相应的子节点，这样避免了页面其他不相关的元素。
  
  - `$('table#test_table1 tr td') `查找页面两个 Table 中的一个的数据行所有单元格对象
  
- **子选择器（Child Selector）**

  子选择器`$('parent>child')`类似层级选择器，但是限定了层级关系必须是父子关系，就是`<child>`节点必须是`<parent>`节点的直属子节点
  
  - `$('table#test_table1>tbody>tr')`查找页面两个 Tables 中的一个的所有行
  
- **过滤器（Filter）**

  过滤器一般不单独使用，它通常附加在选择器上，帮助我们更精确地定位元素。
  
  - ```js
    $('table#test_table1>tbody>tr td:first-child') 查找页面两个 Tables 中的一个的所有数据行的第一个单元格
    $('table#test_table1>tbody>tr td:last-child') 查找页面两个 Tables 中的一个的所有数据行的最后一个单元格
    $('table#test_table1>tbody>tr td:nth-child(3)') 查找页面两个 Tables 中的一个的所有数据行的第三个单元格
    $('table#test_table1>tbody>tr td:nth-child(even)') 查找页面两个Tables中的一个的所有数据行中序号为偶数的单元格
    $('table#test_table1>tbody>tr td:nth-child(odd)') 查找页面两个Tables中的一个的所有数据行中序号为奇数的单元格
    ```
  
- **特殊的选择器**
  
    针对表单元素，jQuery还有一组特殊的选择器：
    
    - `:input`：可以选择`<input>`，`<textarea>`，`<select>`和`<button>`；
    - `:file`：可以选择`<input type="file">`，和`input[type=file]`一样；
    - `:checkbox`：可以选择复选框，和`input[type=checkbox]`一样；
    - `:radio`：可以选择单选框，和`input[type=radio]`一样；
    - `:focus`：可以选择当前输入焦点的元素，例如把光标放到一个`<input>`上，用`$('input:focus')`就可以选出；
    - `:checked`：选择当前勾上的单选框和复选框，用这个选择器可以立刻获得用户选择的项目，如`$('input[type=radio]:checked')`；
  - `:enabled`：可以选择可以正常输入的`<input>`、`<select>` 等，也就是没有灰掉的输入；
  - `:disabled`：和`:enabled`正好相反，选择那些不能输入的。
  
- **其他选择器**

    - 选出可见的或隐藏的元素：

      `$('div:visible');` // 所有可见的div
      `$('div:hidden');` // 所有隐藏的div

## 3、DOM操作函数

### ①添加节点元素

#### append

> 添加到目标元素内部的最后

```js
// 使用 append 会直接添加到目标元素内部的最后，而 after 则会将内容插入到目标元素的同级位置。
// 如果目标元素有兄弟元素，after 会将内容插入到目标元素的后面，而 append 会将内容添加到目标元素内部的末尾。
$("#DomID").append($('<input type="button" class="button" style="text-align: center;margin-right: 4px">').attr('value', 'test');
```

#### prepend

> 插入到目标元素内的起始位置，即作为其子元素中的第一个子元素。

```js
$("#DomID").prepend($('<input type="button" class="button" style="text-align: center;margin-right: 4px">').attr('value', 'test');
```

#### before

> 在指定节点前添加目标元素 

```js
$("#DomID").before($('<input type="button" class="button" style="text-align: center;margin-right: 4px">').attr('value', 'test');
```

#### after

> 在指定节点后添加目标元素 

```js
$("#DomID").after($('<input type="button" class="button" style="text-align: center;margin-right: 4px">').attr('value', 'test');
```

### ②清空节点/多重操作

```js
$('#DomID').empty().append($('<input type="button" class="button" style="text-align: center;margin-right: 4px">').attr('value', 'test');
```

## 4、效果和动画函数

### ① 显示或隐藏元素

`show()`：显示元素

`hide()`：隐藏元素

### ② 淡入或淡出元素

`fadeIn()`： 淡入元素

 `fadeOut()`：淡出元素

### ③ 上滑或下滑元素

- `slideUp()`：上滑元素
- `slideDown()`：下滑元素

### ④ 创建自定义动画

- `animate(properties, duration, easing, complete)`

## 5、事件处理函数

`click(handler)`: 在元素被点击时执行函数。

`change(handler)`: 在表单元素的值发生变化时执行函数。

`mouseover(handler)`, `mouseout(handler)`: 当鼠标移入或移出元素时执行函数。

`keydown(handler)`, `keyup(handler)`: 当键盘按下或释放时执行函数。

## 6、Ajax函数

### ① 通用Ajax请求

`$.ajax(options)`：是 jQuery 提供的一个通用的 Ajax 请求方法，它具有更大的灵活性和可配置性

```js
var id = parseInt(Math.random() * 100000000);
var postData = { "id": id, key1: "value1", key2: "value2" };

$.ajax({
    url: "targetURL",             											// 请求的目标 URL
    type: "POST",                  											// 请求的类型，例如 "GET" 或 "POST"
    data: JSON.stringify(postData),  										// 要发送到服务器的数据，可以是对象、字符串或数组
    dataType: "json",              											// 预期从服务器接收的数据类型，例如 "json"
    contentType: "application/json",										// 发送数据到服务器时使用的内容类型
    headers: {                     											// 设置请求头
      "Authorization": "Bearer token"
    },
    beforeSend: function(xhr) {     										// 发送请求之前执行的回调函数
      // 可以在这里进行一些预处理操作，例如设置请求头
    },
    success: function(response) {   										// 请求成功时执行的回调函数
      console.log("服务器响应:", response);
    },
    error: function(jqXHR, textStatus, errorThrown) {  	// 请求失败时执行的回调函数
      console.error("请求失败:", textStatus, errorThrown);
    },
    complete: function(xhr, status) {  									// 请求完成时执行的回调函数（无论成功或失败）
      console.log("请求完成，状态:", status);
    },
    timeout: 5000,                   										// 超时时间，单位毫秒
    async: true,                     										// 是否使用异步请求，默认为 true
    cache: false                     										// 是否启用缓存，默认为 true
});
```

### ② GET、POST请求

`$.get(url, data, success, dataType)`

`$.post(url, data, success, dataType)`

- **url (必需)：** 发送请求的目标 URL。
- **data (可选)：** 要发送到服务器的数据。可以是字符串、对象或数组。
- **success (可选)：** 请求成功时执行的回调函数。该函数的参数是服务器返回的数据。
- **dataType (可选)：** 预期从服务器接收的数据类型。常见的值包括 "xml"、"json"、"html" 和 "text"。

## 7、工具函数

### ① 遍历数组或对象

`$.each(数组, 回调函数)`

```js
// 遍历 Ajax 返回的响应JSON数据，操作其中的值
$.each(response.data, function (index, data) {
    var row = $('<tr>').attr('id', index);
    row.append($('<td style="width: 43%;">').text(data.name));
    tbody.append(row);
});
```

### ②  过滤数组

`$.grep(数组, 回调函数)`

```js
// 例如过滤数组中所有的偶数
var numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
var evenNumbers = $.grep(numbers, function (element) {
    return element % 2 === 0;
});
```

### ③ 合并对象

`$.extend(target, object1, object2, ...)`: 将一个或多个对象的内容合并到目标对象中

# 三、Vue3

## 1、基础文件详解

- `App.vue` 是 Vue.js 应用的根组件。它包含了整个应用的结构和行为。

  - `<template>`：定义组件的模板，它描述了组件的结构和布局。
  - `<script>`：定义组件的逻辑，它包含了组件的方法、数据和生命周期钩子。
  - `<style>`：定义组件的样式，它包含了组件的样式规则。

- `main.js` 是 Vue.js 应用的入口文件。它负责创建 Vue 实例并挂载到 DOM 元素上。

  ```js
  import { createApp } from 'vue'
  import App from './App.vue'
  
  createApp(App).mount('#app')
  ```

  - `import { createApp } from 'vue'`：导入 Vue.js 库中的 `createApp` 函数。

  - `import App from './App.vue'`：导入根组件 `App.vue`。

  - `createApp(App).mount('#app')`：使用 `createApp()` 函数创建 Vue 实例，并将其挂载到 `#app` 元素上。

- `index.html` 是 HTML 入口文件。它定义了页面的结构和内容。

  ```js
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Vue.js App</title>
  </head>
  <body>
    <div id="app"></div>
    <script src="./main.js"></script>
  </body>
  </html>
  ```

  - `<!DOCTYPE html>`：声明 HTML 文档类型。

  - `<html>`：HTML 根元素。

  - `<head>`：HTML 头部元素，包含元数据和脚本。

  - `<title>`：页面标题。

  - `<body>`：HTML 主体元素，包含页面的内容。

  - `<div id="app"></div>`：一个空 `<div>` 元素，用于挂载 Vue 实例。

  - `<script src="./main.js"></script>`：导入 `main.js` 脚本文件。

- `package.json` 是 Node.js 项目的配置文件。它包含了项目的基本信息、依赖项和其他元数据。

  ```json
  {
    "name": "vue-app",
    "version": "1.0.0",
    "description": "A Vue.js application",
    "main": "main.js",
    "scripts": {
      "dev": "vite",
      "build": "vite build"
    },
    "dependencies": {
      "vue": "^3.2.36"
    }
  }
  ```

  - `"name"`：项目的名称。
  - `"version"`：项目的版本。
  - `"description"`：项目的描述。
  - `"main"`：项目的入口文件。
  - `"scripts"`：项目中可以运行的脚本命令。
  - `"dependencies"`：项目所需的依赖项。


- `vite.config.js` 是 Vite 配置文件。它用于配置 Vite 开发服务器和其他构建选项。

  ```javascript
  module.exports = {
    plugins: [],
    server: {
      port: 3000
    }
  }
  ```


  - `"plugins"`：Vite 插件列表。
  - `"server"`：Vite 开发服务器配置。
