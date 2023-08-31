# Flask总结

# 1、工程目录

```bash

```

# 2、快速搭建

```bash
pip3 install flask
```

>  main.py

```python
from flask import Flask,request
import json

app = Flask('test')
f = open('test.log','w')

@app.route('/test', methods=['POST'])
def test():
    print(json.dumps(request.get_json(), sort_keys=True, indent=4, separators=(',', ':'),ensure_ascii=False), file=f)
    f.close()
    return ""
  
if __name__ == '__main__':
    app.run(debug=True, port=5005, host='0.0.0.0')
```

# 3、USGI

wsgi

```bash
pip3 install gunicorn
```

