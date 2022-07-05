# fastapi-postgresql

## Postgresql and FastAPI
An API 
- Setted python virtual environment to make independent environment for development.
- Developed with minimal low-level web server able to use all async frameworks.
- Import sqlalchemy, psycopg2-binary to use postgresql.

## Initialize project

This instruction is based on MacOS(UNIX) environment. If you are using Window, almost everything would be fine but you need to find additional materials.

venv (for Python 3) and virtualenv (for Python 2) allow you to manage separate package installations for different projects. They essentially allow you to create a “virtual” isolated Python installation and install packages into that virtual installation.

```
python3 -m venv env
source env/bin/activate
deactivate
```

In order to install basic python virtual environment, you should enter command ```python3 -m venv <folder name>```.
```source env/bin/activate``` command will activate virtual environment. If you don't activate this environment any longer, you can use ```deactivate``` command. We can see the envirnment is activated by information on the side.

<img width="450" alt="image" src="https://user-images.githubusercontent.com/39740066/177014141-7df32a93-b9f0-4687-b0cc-f801da0f134c.png">

Then install ```FastAPI``` with ```pip3 install fastapi``` command. And install uvicorn by using ```pip3 install uvicorn```.

## uvicorn server

Uvicorn is an ASGI web server implementation for Python. Until recently Python has lacked a minimal low-level server/application interface for async frameworks. The ASGI specification fills this gap, and means we're now able to start building a common set of tooling usable across all async frameworks.

By using ```httpie``` we can check ```GET``` request of uvicorn server.

```
uvicorn main:app --reload
```

Above command start uvicorn server on ```http://127.0.0.1:8000```, and you can check with ```http localhost:8000``` command.

<img width="450" alt="image" src="https://user-images.githubusercontent.com/39740066/177016948-8deb24ee-70ce-4780-816a-1225988c0f81.png">

## Create schema with pydantic

[Pydantic](https://pydantic-docs.helpmanual.io/) allows custom data types to be defined or you can extend validation with methods on a model decorated with the validator decorator. As well as BaseModel , pydantic provides a dataclass decorator which creates (almost) vanilla python dataclasses with input data parsing and validation. Pydantic models can be created from arbitrary class instances to support models that map to ORM objects.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
  id:int
  name:str
  description:str
  price:int
  on_offer:bool

@app.put('/item/{item_id}')
def update_item(item_id:int, item:Item):
  return {
    'name':item.name,
    'description':item.description,
    'price':item.price,
    'on_offer':item.on_offer
    }
```

You can check api PUT request by using [httpie](https://httpie.io/docs/cli/non-string-json-fields).

<img width="450" alt="image" src="https://user-images.githubusercontent.com/39740066/177057217-19b70f03-3566-493a-929d-9f5bc18c4051.png">

## Type chech with ```Optional```

Optional[...] is a shorthand notation for Union[..., None] , telling the type checker that either an object of the specific type is required, or None is required.

```python
@app.get('/greet')
def greet_optional_name(name:Optional[str]="user"):
  return {"message":f"Hello {name}"}
```

If you check ```/greet``` url, "Hello user" message will be shown like below.

<img width="450" alt="image" src="https://user-images.githubusercontent.com/39740066/177057807-426ddbe5-4172-466e-ab9c-7b340e9811c8.png">

We can also request JSON with username by using ```localhost:8000/greet?name=john```.

<img width="450" alt="image" src="https://user-images.githubusercontent.com/39740066/177057848-8ff9040c-0e47-4b3d-a2fd-03d853da12bf.png">

## Database setup

### 1. sqlalchemy

[SQLAlchemy](https://sqlalchemy.org/) is a library that facilitates the communication between Python programs and databases. Most of the times, this library is used as an Object Relational Mapper (ORM) tool that translates Python classes to tables on relational databases and automatically converts function calls to SQL statements.

```
pip3 install sqlalchemy
```

### 2. psycopg2-binary

[Psycopg](https://pypi.org/project/psycopg2-binary/) is the most popular PostgreSQL database adapter for the Python programming language. Its main features are the complete implementation of the Python DB API 2.0 specification and the thread safety (several threads can share the same connection).

```
pip3 install psycopg2-binary
```

### 3. Install postgresql

[PostgreSQL](https://www.postgresql.org/) is used as the primary data store or data warehouse for many web, mobile, geospatial, and analytics applications. Postgres offers a wider variety of data types than MySQL. If your application deals with any of the unique data types it has available, or unstructured data, PostgreSQL may be a better pick. If you're using only basic character and numeric data types, both databases will suit you.

Download installation file and follow instruction on the [webpage](https://www.postgresqltutorial.com/postgresql-getting-started/install-postgresql-macos/). 

<img width="450" alt="image" src="https://user-images.githubusercontent.com/39740066/177076183-a666502d-e577-4b9c-bae7-c2be5b275a79.png">

### 4. Add database

Create ```item_db``` database with ```pgAdmin```.

<img width="450" alt="image" src="https://user-images.githubusercontent.com/39740066/177076548-fd7d5425-6d24-49fb-bb48-6c43a23aaeaf.png">

After that, make a ```database.py``` like below.

```python
from sqlalchemy.orm import declarative_base, sessionmaker
from sqlalchemy import create_engine

engine = create_engine("postgresql://{POSTGRES_USER}:{POSTGRES_PASSWORD}@{POSTGRES_SERVER}:{POSTGRES_PORT}/{POSTGRES_DB}",
  echo = True
)

Base = declarative_base()

SessionLocal = sessionmaker(bind=engine)
```

### 5. Create Models

In order to communicate with FastApi and postgresql, ```model``` should be made which is connected with ```Item``` class inside of ```main.py```.

models.py
```python
from database import Base
from sqlalchemy import String, Boolean, Integer, Column, Text

class Item(Base):
  __tablename__ = 'items'
  id = Column(Integer, primary_key = True)
  name = Column(String(255), nullable = False, unique = True)
  description = Column(Text)
  price = Column(Integer, nullable = False)
  on_offer = Column(Boolean, default = False)
```

And you can check this model is working on the python3 environment.

<img width="450" alt="image" src="https://user-images.githubusercontent.com/39740066/177079442-c0a02474-a1de-4b5c-ae07-fca68d3d40d3.png">
<img width="450" alt="image" src="https://user-images.githubusercontent.com/39740066/177079477-8090f584-8556-4c1a-ab12-53569bdf5bdf.png">

### 6. Create database

Make ```create_db.py``` like below.

```python
from database import Base, engine
from models import Item

print("Creating database ....")

Base.metadata.create_all(engine)
```

And enter ```python3 create_db.py``` command, then you can see created database table.

<img width="450" alt="image" src="https://user-images.githubusercontent.com/39740066/177081902-e2d64d97-dbff-45bd-848c-e8242545758d.png">

You can also find out ```Columns``` inside of ```item_db``` on the ```pgAdmin```.

<img width="450" alt="image" src="https://user-images.githubusercontent.com/39740066/177226564-23d53cd4-fe6a-4932-b83c-09514c46aa46.png">

## Made a CRUD API code

Now, we can build up api setup with postgresql on the ```main.py```.

### 1. Get all items

main.py

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional, List
from database import SessionLocal
import models

db = SessionLocal()

@app.get('/items', response_model=List[Item], status_code=200)
def get_all_items():
  items=db.query(models.Item).all()

  return items
```

## References
- https://www.youtube.com/watch?v=2g1ZjA6zHRo&t=168s
- https://www.uvicorn.org/
- https://stackoverflow.com/questions/51710037/how-should-i-use-the-optional-type-hint
