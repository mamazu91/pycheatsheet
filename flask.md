1. [run.py](#1)
2. [app.py](#2)
3. [views.py](#3)


# run.py
```python
import views
from app import app

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```
Импортирование views обеспечивает доступность и регистрацию маршрутов. Без этого импорта маршрутов не будет. Причем при запуске приложения через команду flask run маршрутов (по какой-то причине) тоже не будет.

# app.py
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
Создание инстанса marshmallow должно происходить после создания инстанса sqlalchemy.

# views.py
```python
from flask.views import MethodView
from models import Advertisements, advertisement_schema, all_advertisements_schema, Publishers
from flask import request
from app import app, db
from datetime import datetime
from marshmallow import ValidationError


class AllAdvertisementsView(MethodView):
    def get(self):
        all_advertisements = Advertisements.query.all()

        return all_advertisements_schema.jsonify(all_advertisements)

    def post(self):
        input_data = request.get_json()

        if not input_data:
            return {"response": "No input data provided"}, 400

        try:
            validated_data = advertisement_schema.load(input_data)
        except ValidationError as err:
            return err.messages, 422

        if not request.authorization:
            publisher = 'anonymous'
        else:
            publisher = request.authorization.username

        advertisement = Advertisements.query.filter_by(title=validated_data.get('title')).first()

        if advertisement:
            return {'response': f'Advertisement with title {advertisement.title} already exists'}, 422

        new_advertisement = Advertisements(
            title=validated_data.get('title'),
            description=validated_data.get('description'),
            created_on=datetime.now(),
            publisher=Publishers.select_or_create(publisher)
        )

        db.session.add(new_advertisement)
        db.session.commit()

        return advertisement_schema.dump(new_advertisement)


class AdvertisementView(MethodView):

    def get(self, advertisement_id):
        advertisement = Advertisements.query.get_or_404(
            advertisement_id,
            description=f'Advertisement with id {advertisement_id} not found'
        )

        return advertisement_schema.dump(advertisement)

    def patch(self, advertisement_id):
        input_data = request.get_json()

        if not input_data:
            return {"response": "No input data provided"}, 400

        try:
            validated_data = advertisement_schema.load(input_data)
        except ValidationError as err:
            return err.messages, 422

        if not request.authorization:
            publisher = 'anonymous'
        else:
            publisher = request.authorization.username

        advertisement = Advertisements.query.get_or_404(
            advertisement_id,
            description=f'Advertisement with id {advertisement_id} not found'
        )

        advertisement.title = validated_data.get('title'),
        advertisement.description = validated_data.get('description'),
        advertisement.created_on = datetime.now(),
        advertisement.publisher = Publishers.select_or_create(publisher)

        db.session.commit()

        return advertisement_schema.dump(advertisement)

    def delete(self, advertisement_id):
        advertisement = Advertisements.query.get_or_404(
            advertisement_id,
            description=f'Advertisement with id {advertisement_id} not found'
        )

        advertisement_to_delete_json = advertisement_schema.dump(advertisement)

        db.session.delete(advertisement)
        db.session.commit()

        return advertisement_to_delete_json


app.add_url_rule('/advertisements', view_func=AllAdvertisementsView.as_view('all_advertisements'),
                 methods=['GET', 'POST'])
app.add_url_rule('/advertisements/<int:advertisement_id>', view_func=AdvertisementView.as_view('advertisement'),
                 methods=['GET', 'PATCH', 'DELETE'])

```
