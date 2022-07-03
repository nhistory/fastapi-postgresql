# fastapi-postgresql

## Postgresql and FastAPI
An API 
- Setted python virtual environment to make independent environment for development.
- Developed with minimal low-level web server able to use all async frameworks.

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


## References
- https://www.youtube.com/watch?v=2g1ZjA6zHRo&t=168s
- https://www.uvicorn.org/
- https://stackoverflow.com/questions/51710037/how-should-i-use-the-optional-type-hint
