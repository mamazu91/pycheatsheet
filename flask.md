## Добавление маршрута

### Через декоратор
```python
@app.route('/api/v1', methods=['GET'])
def some_func():
    return None
```
### Через метод  
```python
def some_func():
    return None
    
app.add_url_rule('/api/v1', view_func=some_func, methods=['GET'])
```

## Доступ к данным и заголовкам

### Заголовки
```python
from flask import request

def some_func():
    if request.headers.get('token') == 'something':
        return jsonify({'response': 'ok'})
```

### Данные
