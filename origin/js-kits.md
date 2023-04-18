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

·