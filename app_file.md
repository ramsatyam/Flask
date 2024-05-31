Version 01
```
from flask import Flask, request

app = Flask(__name__)

stores = [{"name": "My Store", "items": [{"name": "Chair", "price": 15.99}]}]


@app.get("/store")
def get_stores():
    return {"stores": stores}


@app.post("/store")
def create_store():
    request_data = request.get_json()
    new_store = {"name": request_data["name"], "items": []}
    stores.append(new_store)
    return new_store, 201


@app.post("/store/<string:name>/item")
def create_item(name):
    request_data = request.get_json()
    for store in stores:
        if store["name"] == name:
            new_item = {"name": request_data["name"], "price": request_data["price"]}
            store["items"].append(new_item)
            return new_item, 201
    return {"message": "Store not found"}, 404


@app.get("/store/<string:name>")
def get_store(name):
    for store in stores:
        if store["name"] == name:
            return store
    return {"message": "Store not found"}, 404


@app.get("/store/<string:name>/item")
def get_item_in_store(name):
    for store in stores:
        if store["name"] == name:
            return {"items": store["items"]}
    return {"message": "Store not found"}, 404
```
Version 02
```
# db.py
stores = {}
items = {}
```
```
# app.py
import uuid
from flask import Flask, request
from db import stores, items
app = Flask(__name__)

@app.route('/')
def home():
    return "This is home function plus home route ."

@app.get('/store')
def get_store():
    return {'stores': list(stores.values())}

@app.get('/store/<string:store_id>')
def get_store01(store_id):
    try:
        return stores[store_id], 200
    except:
        return {'message': 'store not found'}, 404

@app.get('/item')
def get_items():
    return {'items': list(items.values())}

@app.get('/item/<string:item_id>')
def get_item(item_id):
    try:
        return items[item_id], 200
    except:
        return {"message": "Item id not found"}, 404

@app.post('/store')
def create_store():
    store_data = request.get_json()
    store_id = uuid.uuid4().hex
    store = {**store_data, "id": store_id}
    stores[store_id] =   store
    return store, 201

@app.post('/item')
def create_items():
    item_data = request.get_json()
    item_id = uuid.uuid4().hex
    item = {**item_data, "id": item_id}
    items[item_id] = item
    return item, 201
```
Here the return error messages are provided manually, so this doesn't come in the documentation.<br>
For this lets use `Flast Smorest`
```
import uuid
from flask import Flask, request
from db import stores, items
from flask_smorest import abort
app = Flask(__name__)

@app.route('/')
def home():
    return "This is home function plus home route ."

@app.get('/store')
def get_store():
    return {'stores': list(stores.values())}

@app.get('/store/<string:store_id>')
def get_store01(store_id):
    try:
        return stores[store_id], 200
    except:
        abort(404, message= "Store not found")

@app.get('/item')
def get_items():
    return {'items': list(items.values())}

@app.get('/item/<string:item_id>')
def get_item(item_id):
    try:
        return items[item_id], 200
    except KeyError:
        abort(404, message= "Item not found")

@app.post('/store')
def create_store():
    store_data = request.get_json()
    if "name" not in store_data:
        abort(
            400, message = "Bad Request. Ensure 'name' is included in the JSON payload."
        )
    for store in stores.values():
        print(store, "---",store_data)
        if store["name"] == store_data["name"]:
            abort(
                400, message = "Store already exists"
            )
    store_id = uuid.uuid4().hex
    store = {**store_data, "id": store_id}
    stores[store_id] = store
    return store, 201

@app.post('/item')
def create_items():
    item_data = request.get_json()
    if (
        "store_id" not in item_data
        or "price" not in item_data
        or "name" not in item_data
    ):
        abort(
            400, message = " Bad Request. Ensure 'Price', 'store_id' and 'name' are included in the JSON payload"
        )
    
    for item in items.values():
        if (
            item_data["name"] == item["name"]
            and item_data["store_id"] == item ["store_id"]
        ):
            abort(
                400, message =" Item already exists"
            )
    if item_data["store_id"] not in stores:
        abort(404, message= "Store id not found.")
    item_id = uuid.uuid4().hex
    item = {**item_data, "id": item_id}
    items[item_id] = item
    return item, 201

@app.delete('/store/<string:store_id>')
def delete_store(store_id):
    try:
        del stores[store_id]
        return {"message": "Store deleted"}
    except KeyError:
        abort(404, message = "Store id not found")

@app.delete('/item/<string:item_id>')
def delete_item(item_id):
    try:
        del items[item_id]
        return {"message": "Item deleted"}
    except KeyError:
        abort(404, message="item not found")

@app.put("/store/<string:store_id>")
def update_store(store_id):
    response_data = request.get_json()
    try:
        store = stores[store_id]
        store |= response_data
        return store
    except:
        abort(404 , message = "store id not found")

@app.put("/item/<string:item_id>")
def update_item(item_id):
    item_data = request.get_json()
    if "name" not in item_data or "price" not in item_data:
        abort( 400, message="Bad Request 'name' and 'price' must be included in the JSON payload")
    try:
        item = items[item_id]
        item |= item_data
        return item
    except:
        abort(404, message="Item not found")
```

# Smorest
Now, lets add blueprints and documentation using swaggerUI.<br>
Folder Structure before using smorest.
```
PROJECT01
│   .flaskenv
│   app.py
│   db.py
│   docker-compose.yml
│   Dockerfile
│   requirements.txt
```
After using smorest. The file structure is more like this

```
PROJECT01
│   .flaskenv
│   app.py
│   db.py
│   docker-compose.yml
│   Dockerfile
│   requirements.txt
│
├───resources
│   │   item.py
│   │   store.py
│   │
│   └───__pycache__
│           item.cpython-310.pyc
│           store.cpython-310.pyc
│
└───__pycache__
        app.cpython-310.pyc
        db.cpython-310.pyc
        main.cpython-310.pyc
```
Now app.py is
```
from flask import Flask
from flask_smorest import Api
from resources.item import blp as ItemBlueprint
from resources.store import blp as StoreBlueprint

app = Flask(__name__)

app.config["PROPAGATE_EXCEPTIONS"] = True
app.config["API_TITLE"] = "Stores REST API"
app.config["API_VERSION"] = "v1"
app.config["OPENAPI_VERSION"] = "3.0.3"
app.config["OPENAPI_URL_PREFIX"] = "/"
app.config["OPENAPI_SWAGGER_UI_PATH"] = "/swagger-ui"
app.config["OPENAPI_SWAGGER_UI_URL"] = "https://cdn.jsdelivr.net/npm/swagger-ui-dist/"

api = Api(app)

api.register_blueprint(ItemBlueprint)
api.register_blueprint(StoreBlueprint)
```
item.py
```
import uuid
from flask import Flask, request
from flask.views import MethodView
from db import items, stores
from flask_smorest import Blueprint, abort


blp = Blueprint("item", __name__, description= "Operations on items")

@blp.route("/item/<string:item_id>")
class item(MethodView):
    def get(self, item_id):
        try:
            return items[item_id], 200
        except KeyError:
            abort(404, message= "Item not found")

    def delete(self, item_id):
        try:
            del items[item_id]
            return {"message": "Item deleted"}
        except KeyError:
            abort(404, message="item not found")

    def put(self, item_id):
        item_data = request.get_json()
        if "name" not in item_data or "price" not in item_data:
            abort( 400, message="Bad Request 'name' and 'price' must be included in the JSON payload")
        try:
            item = items[item_id]
            item |= item_data
            return item
        except:
            abort(404, message="Item not found")

@blp.route("/item")
class item_list(MethodView):
    def get(self):
        return {'items': list(items.values())}

    def post(self):
        item_data = request.get_json()
        if (
            "store_id" not in item_data
            or "price" not in item_data
            or "name" not in item_data
        ):
            abort(
                400, message = " Bad Request. Ensure 'Price', 'store_id' and 'name' are included in the JSON payload"
            )
        
        for item in items.values():
            if (
                item_data["name"] == item["name"]
                and item_data["store_id"] == item ["store_id"]
            ):
                abort(
                    400, message =" Item already exists"
                )
        if item_data["store_id"] not in stores:
            abort(404, message= "Store id not found.")
        item_id = uuid.uuid4().hex
        item = {**item_data, "id": item_id}
        items[item_id] = item
        return item, 201
```
store.py
```
import uuid
from flask import Flask, request
from flask.views import MethodView
from db import stores
from flask_smorest import Blueprint, abort

blp = Blueprint("store", __name__, description="Operations on Stores")


@blp.route("/store/<string:store_id>")
class Store(MethodView):
    def get(self, store_id):
        try:
            return stores[store_id], 200
        except:
            abort(404, message= "Store not found")
    def delete(self, store_id):
        try:
            del stores[store_id]
            return {"message": "Store deleted"}
        except KeyError:
            abort(404, message = "Store id not found")

    def put(self, store_id):
        response_data = request.get_json()
        try:
            store = stores[store_id]
            store |= response_data
            return store
        except:
            abort(404 , message = "store id not found")

@blp.route("/store")
class StoreList(MethodView):
    def get(self):
        return {'stores': list(stores.values())}
    
    def post(self):
        store_data = request.get_json()
        if "name" not in store_data:
            abort(
                400, message = "Bad Request. Ensure 'name' is included in the JSON payload."
            )
        for store in stores.values():
            print(store, "---",store_data)
            if store["name"] == store_data["name"]:
                abort(
                    400, message = "Store already exists"
                )
        store_id = uuid.uuid4().hex
        store = {**store_data, "id": store_id}
        stores[store_id] = store
        return store, 201
```

# Marshmallow
Now lets introduce marshmallow for data validation. For this we first shall introduce a new file schemas.py and declare schemas required for validation.<br>
So the folder structure is like below
```
PROJECT01
│   .flaskenv
│   app.py
│   db.py
│   docker-compose.yml
│   Dockerfile
│   requirements.txt
│   schemas.py
│   
├───resources
│   │   item.py
│   │   store.py
│   │
│   └───__pycache__
│           item.cpython-310.pyc
│           store.cpython-310.pyc
│
└───__pycache__
        app.cpython-310.pyc
        db.cpython-310.pyc
        main.cpython-310.pyc
        schemas.cpython-310.pyc

```

schemas.py
```
from marshmallow import Schema, fields

class ItemSchema(Schema):
    id = fields.Str(dump_only=True)
    name = fields.Str(required=True)
    print = fields.Float(required=True)
    store_id = fields.Str(required=True)

class ItemUpdateSchema(Schema):
    name = fields.Str()
    price = fields.Float()

class StoreSchema(Schema):
    id = fields.Str(dump_only=True)
    name = fields.Str(required=True)
```
dump_only: data is provided when it is returned to the user.<br>
* item.py
```
import uuid
from flask import Flask, request
from flask.views import MethodView
from db import items, stores
from flask_smorest import Blueprint, abort

from schemas import ItemSchema, ItemUpdateSchema

blp = Blueprint("item", __name__, description= "Operations on items")

@blp.route("/item/<string:item_id>")
class item(MethodView):
    def get(self, item_id):
        try:
            return items[item_id], 200
        except KeyError:
            abort(404, message= "Item not found")

    def delete(self, item_id):
        try:
            del items[item_id]
            return {"message": "Item deleted"}
        except KeyError:
            abort(404, message="item not found")

    @blp.arguments(ItemUpdateSchema)
    @blp.response(200, ItemUpdateSchema)
    def put(self, item_data, item_id):
        # item_data = request.get_json()
        # if "name" not in item_data or "price" not in item_data:
        #     abort( 400, message="Bad Request 'name' and 'price' must be included in the JSON payload")
        try:
            item = items[item_id]
            item |= item_data
            return item
        except:
            abort(404, message="Item not found")

@blp.route("/item")
class item_list(MethodView):
    @blp.response(200, ItemSchema(many=True))
    def get(self):
        return items.values()


    @blp.arguments(ItemSchema)
    @blp.response(201, ItemSchema)
    def post(self, item_data):
        # item_data = request.get_json() #This part is done by marshmallow
        for item in items.values():     #marshmallow cannot check this.
            if (
                item_data["name"] == item["name"]
                and item_data["store_id"] == item ["store_id"]
            ):
                abort(
                    400, message =" Item already exists"
                )
        if item_data["store_id"] not in stores:
            abort(404, message= "Store id not found.")
        item_id = uuid.uuid4().hex
        item = {**item_data, "id": item_id}
        items[item_id] = item
        return item
```
* store.py 
```
import uuid
from flask import Flask, request
from flask.views import MethodView
from db import stores
from flask_smorest import Blueprint, abort

from schemas import StoreSchema

blp = Blueprint("store", __name__, description="Operations on Stores")


@blp.route("/store/<string:store_id>")
class Store(MethodView):
    def get(self, store_id):
        try:
            return stores[store_id], 200
        except:
            abort(404, message= "Store not found")
    def delete(self, store_id):
        try:
            del stores[store_id]
            return {"message": "Store deleted"}
        except KeyError:
            abort(404, message = "Store id not found")

    def put(self, store_id):
        response_data = request.get_json()
        try:
            store = stores[store_id]
            store |= response_data
            return store
        except:
            abort(404 , message = "store id not found")

@blp.route("/store")
class StoreList(MethodView):
    @blp.response(200, StoreSchema(many=True))
    def get(self):
        return stores.values()
    

    @blp.arguments(StoreSchema)
    @blp.response(201, StoreSchema)
    def post(self, store_data):
        # store_data = request.get_json()
        # if "name" not in store_data:
        #     abort(
        #         400, message = "Bad Request. Ensure 'name' is included in the JSON payload."
        #     )
        for store in stores.values():
            print(store, "---",store_data)
            if store["name"] == store_data["name"]:
                abort(
                    400, message = "Store already exists"
                )
        store_id = uuid.uuid4().hex
        store = {**store_data, "id": store_id}
        stores[store_id] = store
        return store
```
* app.py
```
from flask import Flask
from flask_smorest import Api
from resources.item import blp as ItemBlueprint
from resources.store import blp as StoreBlueprint

app = Flask(__name__)

app.config["PROPAGATE_EXCEPTIONS"] = True
app.config["API_TITLE"] = "Stores REST API"
app.config["API_VERSION"] = "v1"
app.config["OPENAPI_VERSION"] = "3.0.3"
app.config["OPENAPI_URL_PREFIX"] = "/"
app.config["OPENAPI_SWAGGER_UI_PATH"] = "/swagger-ui"
app.config["OPENAPI_SWAGGER_UI_URL"] = "https://cdn.jsdelivr.net/npm/swagger-ui-dist/"

api = Api(app)

api.register_blueprint(ItemBlueprint)
api.register_blueprint(StoreBlueprint)
```

* db.py
```
stores = {}
items = {}
```
# SQLAlchemy
SQLAlchemy is an ORM (Object Relational Mapping). Using ORM have advantages like
1. Multithreading support
2. Handles creating tables and columns for you
3. Easy in database migrations.

Lets add them in requirements.txt
```
flask
flask-smorest
python-dotenv
sqlalchemy
flask-sqlalchemy
```
lets install them using ` pip install -r requirements.txt `.<br>
Since we are running docker with volume mapping it will detect the changes in requirements.txt and will update the docker images automatically.<br>
#### SQLAlchemy model
