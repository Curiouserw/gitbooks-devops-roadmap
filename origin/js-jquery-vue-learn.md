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





ES6标准新增了一种新的函数：Arrow Function（箭头函数）。

​	x => x * x

​	上面的箭头函数相当于：

​	**function** (x) {

  		**return** x * x;

​	}



如果仔细观察一个Form的提交，你就会发现，一旦用户点击“Submit”按钮，表单开始提交，浏览器就会刷新页面，然后在新页面里告诉你操作是成功了还是失败了。如果不幸由于网络太慢或者其他原因，就会得到一个404页面。

这就是Web的运作原理：一次HTTP请求对应一个页面。

如果要让用户留在当前页面中，同时发出新的HTTP请求，就必须用JavaScript发送这个新请求，接收到数据后，再用JavaScript更新页面，这样一来，用户就感觉自己仍然停留在当前页面，但是数据却可以不断地更新。

# 二、jQuery

## jQuery能做哪些事情

- 消除浏览器差异：你不需要自己写冗长的代码来针对不同的浏览器来绑定事件，编写AJAX等代码；
- 简洁的操作DOM的方法：写$('#test')肯定比document.getElementById('test')来得简洁；
- 轻松实现动画、修改CSS等各种操作。



$是著名的jQuery符号。实际上，jQuery把所有功能全部封装在一个全局变量jQuery中，而$也是一个合法的变量名，它是变量jQuery的别名：



$本质上就是一个函数，但是函数也是对象，于是$除了可以直接调用外，也可以有很多其他属性。

注意，你看到的$函数名可能不是jQuery(selector, context)，因为很多JavaScript压缩工具可以对函数名和参数改名，所以压缩过的jQuery源码$函数可能变成a(b, c)

## jQuery的选择器

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
  
  - ```
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


## jQuery对象和DOM对象互相转化

```bash
var div = $('#abc'); *// jQuery*对象
var divDom = div.get(0); *//* 假设存在*div*，获取第*1*个*DOM*元素
var another = $(divDom); *//* 重新把*DOM*包装为*jQuery*对象
```

​	通常情况下你不需要获取DOM对象，直接使用jQuery对象更加方便。如果你拿到了一个DOM对象，那可以简单地调用$(aDomObject)把它变成jQuery对象，这样就可以方便地使用jQuery的API了。

## 操作DOM

