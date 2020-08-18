# Авторское

```python
import random

from flask import Flask, jsonify, request

app = Flask('user_agent_service')

@app.route('/client/info', methods=['GET'])
def client_info():
    # берем User-Agent из заголовков
    user_agent = request.headers['User-Agent']
    data = {
        'user_agent': user_agent,
    }
    # прекращаем словарик в json ответ
    return jsonify(data)
```