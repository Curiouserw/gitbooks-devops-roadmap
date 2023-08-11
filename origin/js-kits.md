# JavaScript常用工具函数

# 1、把字节数字转换成易于阅读格式

```javascript
function getBytesSize(size) {
    if (!size)  return "";
    var num = 1024.00; //byte
    if (size < num)
        return size + "B";
    if (size < Math.pow(num, 2))
        return (size / num).toFixed(2) + "KB"; //kb
    if (size < Math.pow(num, 3))
        return (size / Math.pow(num, 2)).toFixed(2) + "MB"; //M
    if (size < Math.pow(num, 4))
        return (size / Math.pow(num, 3)).toFixed(2) + "G"; //G
    return (size / Math.pow(num, 4)).toFixed(2) + "T"; //T
}
```

# 2、将秒转换为时分秒

```bash
function formatDuring(s){
    var days = parseInt(s / ( 60 * 60 * 24));
    var hours = parseInt((s % ( 60 * 60 * 24)) / ( 60 * 60));
    var minutes = parseInt((s % ( 60 * 60)) / 60);
    var seconds = (s % 60) ;
    return days + " 天 " + hours + " 小时 " + minutes + " 分钟 ";
}
```

# 3、设置点击复制并弹出模态框提示复制内容

```html
<!DOCTYPE html>
<html>

<head>
    <link href="https://cdn.staticfile.org/twitter-bootstrap/5.1.3/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.staticfile.org/twitter-bootstrap/5.1.3/js/bootstrap.min.js"></script>
    <style type="text/css">
      .modal {
          display: none;
          position: fixed;
          z-index: 1;
          left: 0;
          top: 0;
          width: 100%;
          height: 100%;
          overflow: auto;
          color: white;
      }

      .modal-content {
          background-color: #0d6efd;
          margin: 15% auto;
          padding: 20px;
          border: 1px solid #888;
          width: 300px;
          text-align: center;
      }
    </style>
    <script type="text/javascript">
        function copyordernum(value) {
            var clipboardInput = document.createElement("input");
            clipboardInput.style.position = "absolute";
            clipboardInput.style.left = "-9999px";
            clipboardInput.style.top = "-9999px";
            clipboardInput.value = value;
            document.body.appendChild(clipboardInput);
            clipboardInput.select();
            document.execCommand("copy");
            document.body.removeChild(clipboardInput);
            var modalMessage = document.getElementById("modalMessage");
            modalMessage.textContent = "已复制到剪贴板: " + value;
            var modal = document.getElementById("myModal");
            modal.style.display = "block";
            setTimeout(function () {
                modal.style.display = "none";
            }, 900);
        }
    </script>
</head>
<body>
    <div id="his" class="container tab-pane active">
        <section id="hissection" class="container py-2">
            <table class="table table-hover table-bordered" id="histable">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>A</th>
                        <th>B</th>
                        <th>C</th>
                        <th>D</th>
                    </tr>
                </thead>
                <tbody id="histablebody">
                    <tr id="11111232131231211">
                        <td>1</td>
                        <td class="hide-string" data-value="11111232131231211" onclick="copyordernum('11111232131231211')">
                            1111***1211</td>
                        <td>111</td>
                        <td>2021-06-03 10:01:15</td>
                        <td>111</td>
                    </tr>
                </tbody>
            </table>
        </section>
    </div>
    <div id="myModal" class="modal">
        <div class="modal-content">
            <p id="modalMessage"></p>
        </div>
    </div>
</body>
</html>
```

# 4、CSS 伪元素实现隐藏过长字符串，焦点悬停式显示完整字符串

方案：使用 BS5 的 CSS伪元素。

- JS在生成表格元素时，将标签的 class 设置为`class="hide-string"`；将原字符串赋予标签的 data-vaule属性；标签值则赋予隐藏字符。

- JS 替换字符串前 4 位和后 4 位中间的字符为*：`str.replace(/(^\d{4})(.*)(\d{4}$)/, "$1***$3");`

```html
<!DOCTYPE html>
<html>

<head>
    <link href="https://cdn.staticfile.org/twitter-bootstrap/5.1.3/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.staticfile.org/twitter-bootstrap/5.1.3/js/bootstrap.min.js"></script>
    <style type="text/css">
      .hide-string {
          position: relative;
      }
      .hide-string::after {
          content: attr(data-value);
          position: absolute;
          top: 0;
          left: 0;
          visibility: hidden;
          opacity: 0;
          background-color: #cccccc;
          padding: 5px;
          border: 1px solid #cccccc;
      }
      .hide-string:hover::after {
          visibility: visible;
          opacity: 1;
      }
    </style>
</head>
<body>
    <div id="his" class="container tab-pane active">
        <section id="hissection" class="container py-2">
            <table class="table table-hover table-bordered" id="histable">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>A</th>
                        <th>B</th>
                        <th>C</th>
                        <th>D</th>
                    </tr>
                </thead>
                <tbody id="histablebody">
                    <tr id="11111232131231211">
                        <td>1</td>
                        <td class="hide-string" data-value="11111232131231211" >
                            1111***1211</td>
                        <td>111</td>
                        <td>2021-06-03 10:01:15</td>
                        <td>111</td>
                    </tr>
                </tbody>
            </table>
        </section>
    </div>
</body>
</html>
```