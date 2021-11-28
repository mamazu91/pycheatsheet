### Оглавление
1. [run.py](#run.py)
2. [app.py](#2)
3. [views.py](#3)

### run.py <a name="1"></a>
```python
import views
from app import app

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```
Импортирование views обеспечивает доступность и регистрацию маршрутов. Без этого импорта маршрутов не будет. Причем при запуске приложения через команду flask run маршрутов (по какой-то причине) тоже не будет.

### views.py <a name="3"></a>
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
Метод **dump** в advertisement_schema.dump(advertisement) сериализует указанный объект в нативную структуру данных в соответствии с выбранной схемой. По-умолчанию это словарь. Метод **dumps** делает то же самое, только возвращает строку. Для множества объектов подойдет **jsonify**: all_advertisements_schema.jsonify(all_advertisements), который возвращает объект типа Response (в словарь его уже превращать не нужно).

Метод **get_or_404** в advertisement = Advertisements.query.get_or_404(advertisement_id, description=f'Advertisement with id {advertisement_id} not found') делает запрос в базу и возвращает или полученную запись (если она существует) или объект типа response (?) с кодом 404 (если она не существует). Без этого метода пришлось бы передавать advertisement на None и т.д., так что очень удобно. Параметр **description** позволяет добавить кастомный текст для кейса с несуществующей записью. Выглядит это так:

![image](https://user-images.githubusercontent.com/48824617/143735578-461f90af-7eaf-451b-bb01-50cee398d590.png)

Метод **load** в advertisement_schema.load(input_data) десериализует нативную структуру данных в объект в соответствии с выбранной схемой, т.е. валидирует данные.

При наличии аутентификации указанного пользователя можно забрать из **request.authorization.username**.

### models.py <a name="4"></a>
```python
from app import db, ma
from datetime import datetime
from marshmallow import fields


class Publishers(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50), unique=True)

    @classmethod
    def select_or_create(cls, username):
        publisher = Publishers.query.filter_by(username=username).first()

        if publisher:
            return publisher

        new_publisher = Publishers(username=username)
        db.session.add(new_publisher)
        db.session.commit()

        return new_publisher


class Advertisements(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False, unique=True)
    description = db.Column(db.String(200), nullable=False)
    created_on = db.Column(db.DateTime(), nullable=False, default=datetime.utcnow)
    publisher_id = db.Column(db.Integer, db.ForeignKey('publishers.id'), nullable=False)
    publisher = db.relationship("Publishers", backref="advertisements")

    @classmethod
    def delete(cls, advertisement):
        db.session.delete(advertisement)
        db.session.commit()

        return advertisement


class PublisherSchema(ma.SQLAlchemySchema):
    class Meta:
        fields = ('id', 'username')
        ordered = True


class AdvertisementSchema(ma.SQLAlchemySchema):
    class Meta:
        fields = ('id', 'title', 'description', 'created_on', 'publisher')
        ordered = True

    publisher = ma.Nested(PublisherSchema)
    title = fields.String(required=True)
    description = fields.String(required=True)


advertisement_schema = AdvertisementSchema()
all_advertisements_schema = AdvertisementSchema(many=True)

db.create_all()
```
Функция **select_or_create** сделана по подобию Джанговской функции update_or_create, для удобства.

**Publisher** в publisher = db.relationship("Publishers", backref="advertisements") подобен related_name в Django: т.е. он позволяет выбрать всех авторов указанного объявления. Параметр **backref** позволяет задать related_name для обратного процесса: т.е. в данном случае через это имя можно все объявления у указанного автора.

Параметр **ordered = True** позволяет избежать сортировки конечного JSON'а по именам ключей. Необходимый порядок ключей нужно указать через **fields**.

Метод **Nested** в ma.Nested(PublisherSchema) добавляет ключ с автором в JSON объявлений при их получении. Выглядит это так:
![image](https://user-images.githubusercontent.com/48824617/143738507-7c8ed3fa-8a7b-4f84-9f5c-de773b421422.png)

Параметр **required=True** в title = fields.String(required=True) используется для возвращения кода 422, если в теле POST запроса не будет указано поле с этим параметров. Выглядит это так:

![image](https://user-images.githubusercontent.com/48824617/143740451-c69732e8-ca64-40d0-8d70-99b81541207e.png)

Параметр **many=True** в AdvertisementSchema(many=True) позволяет использовать эту схему для работы со множеством объектов (т.е. для запросов вида GET /advertisements, а не GET /advertisements/1, например).

### app.py <a name="2"></a>
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
