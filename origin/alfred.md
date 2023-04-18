# Alfred

# 一、Workflow简介

# 二、Workflow配置

## 1、组件

- 触发组件Trigger
  - 快捷键Hotkey：设置键盘组合键进行快速触发
  - 关键词Keyword：设置Alfred关键词，在Alfred调用框中输入关键字进行快速触发
- 输入组件Input
  - 脚本Script filter
  - List filter

## 2、变量引用

- **引用上一个输出：** `{query}`
- **引用Workflow设置的变量：**`{var:变量名}`
- **脚本中引用：** `og.getenv("变量名")`

# 三、Workflow开发

```python
pip3 install --target=. Alfred-Workflow
# 或者
pip3 download --no-deps --no-binary :all: Alfred-Workflow
```

```bash
Your Workflow/
    info.plist
    icon.png
    workflow/
        __init__.py
        background.py
        notify.py
        Notify.tgz
        update.py
        version
        web.py
        workflow.py
    yourscript.py
    ....
```

```python
import sys
from workflow import Workflow, ICON_WEB, web

API_KEY = 'your-pinboard-api-key'

def main(wf):
    url = 'https://api.pinboard.in/v1/posts/recent'
    params = dict(auth_token=API_KEY, count=20, format='json')
    r = web.get(url, params)
    r.raise_for_status()
    for post in r.json()['posts']:
        wf.add_item(post['description'], post['href'], arg=post['href'],
                    uid=post['hash'], valid=True, icon=ICON_WEB)
    wf.send_feedback()


if __name__ == u"__main__":
    wf = Workflow()
    sys.exit(wf.run(main))
```

## 9、实现CheckBox复选框功能

`Script Filter`组件通过添加使用`Call External组件`进行循环调用，再加上`condition条件组件`可实现CheckBox复选框功能

参考：https://www.alfredforum.com/topic/17529-checkbox-logic-workflow/#comment-90456

## 其他信息

- 缓存目录：`~/Library/Caches/com.runningwithcrayons.Alfred/Workflow Data/<bundle id>`

# 参考

- [Workflow官网文档](http://www.deanishe.net/alfred-workflow/)
- [Workflow API文档](http://www.deanishe.net/alfred-workflow/api/index.html)
- https://pypi.org/project/Alfred-Workflow/
- http://www.saitjr.com/others/alfred-script-filter-json-format.html
- https://www.alfredapp.com/help/workflows/inputs/script-filter/json/
- https://www.alfredapp.com/help/workflows/inputs/script-filter/
- https://www.alfredforum.com/topic/17529-checkbox-logic-workflow/#comment-90456
- http://www.deanishe.net/alfred-workflow/api/index.html#workflow-variables
- http://www.deanishe.net/alfred-workflow/guide/variables.html#variables-run-script