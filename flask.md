## Добавление маршрута

### Через декоратор
```python
@app.route('/api/v1', methods=['GET'])
def some_func():
    pass
```
### Через метод  
```python
def some_func():
    pass
    
app.add_url_rule('/api/v1', view_func=some_func, methods=['GET'])
```

## Доступ к заголовкам и данным в запросе

### Заголовки
```python
from flask import request, jsonify

def some_func():
    if request.headers.get('token') == 'something':
        return jsonify({'response': 'correct token'})
```

### Данные
```python
from flask import request, jsonify

def some_func():
    r = request.json
    user = r.get('user')

    if user == 'some_user':
        return jsonify({'response': 'correct user'})

    return jsonify({'response': 'wrong user'})
```

## Валидация данных

### Популярные библиотеки
- jsonschema
- marshmallow
- pydantic

### jsonschema
