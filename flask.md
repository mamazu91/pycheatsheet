### run.py
```python
import views
from app import app

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```
Импортирование views обеспечивает доступность и регистрацию маршрутов. Без этого импорта маршрутов не будет. Причем при запуске приложения через команду flask run маршрутов (по какой-то причине) тоже не будет.

### app.py
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_marshmallow import Marshmallow

app = Flask(__name__)
app.config.update(
    SQLALCHEMY_DATABASE_URI='postgresql://postgres:postgres@db:5432/netology_flask',
    JSON_SORT_KEYS=False
)

db = SQLAlchemy(app)
ma = Marshmallow(app)
```

