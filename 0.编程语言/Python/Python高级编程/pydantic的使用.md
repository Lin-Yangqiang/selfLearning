# Pydantic

Pydantic 是一个用于数据验证、模型定义和序列化操作的库。

## 安装

要使用 Pydantic 库，可以通过 pip 命令安装：

```
pip install pydantic
```

注：接下来的教程都是基于 pydantic v2版本，因此请自行确认下Pydantic的版本是否为2.x：

```
pip show pydantic
```

```
Name: pydantic
Version: 2.9.2
Summary: Data validation using Python type hints
Home-page:
Author:
Author-email: Samuel Colvin <s@muelcolvin.com>, Eric Jolibois <em.jolibois@gmail.com>, Hasan Ramezani <hasan.r67@gmail.com>, Adrian Garcia Badaracco <1755071+adriangb@users.nore
ply.github.com>, Terrence Dorsey <terry@pydantic.dev>, David Montague <david@pydantic.dev>, Serge Matveenko <lig@countzero.co>, Marcelo Trylesinski <marcelotryle@gmail.com>, Sydney Runkle <sydneymarierunkle@gmail.com>, David Hewitt <mail@davidhewitt.io>, Alex Hall <alex.mojaki@gmail.com>
License:
Location: d:\software\miniconda\envs\flask\lib\site-packages
Requires: annotated-types, pydantic-core, typing-extensions
Required-by: fastapi, gradio
```

## 基本功能

### 1. 定义数据模型

Pydantic 库可以帮助我们定义数据模型，并对数据进行验证。

需求：检查传入的数据是否符合我们定义的数据模型的规范。

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    username: str
    email: str


user_data = {"id": 1, "username": "John Doe", "email": "john@example.com"}


user = User(**user_data)
print(user)
```

输出：

```
id=1 username='John Doe' email='john@example.com'
```

> 上述代码定义了一个 `User` 的数据模型，并且传入了一个 `user_data` 数据，Pydantic 帮我们验证 `user_data` 这个数据是否符合 `User` 数据模型。

### 2. 数据序列化

Pydantic 库可以将数据序列化为 JSON 等格式。

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    username: str
    email: str

user = User(id=1, username="john_doe", email="john.doe@example.com")
json_data = user.json()
print(json_data)
```

输出：

```json
{"id": 1, "username": "john_doe", "email": "john.doe@example.com"}
```

## 高级功能

### 1. 自定义校验器

pydantic库允许开发者定义自定义校验器，以实现特定的数据验证逻辑。

例如，定义一个自定义校验器来验证密码复杂度：

```python
from pydantic import BaseModel, constr, validator

class User(BaseModel):
    username: str
    password: constr(min_length=8)

    @validator("password")
    def validate_password(cls, value):
        if not any(char.isdigit() for char in value):
            raise ValueError("Password must contain at least one digit")
        if not any(char.isalpha() for char in value):
            raise ValueError("Password must contain at least one letter")
        return value
```

上述代码定义了一个用户数据模型，并通过自定义校验器`validate_password`来验证密码的复杂度，确保密码包含字母和数字。

我们尝试验证两条数据：

1. 验证一条合法的数据：

   ```python
   user_data = {"username": "john_doe", "password": "securepassword123"}
   user = User(**user_data)
   print(user)
   ```

   输出：

   ```
   username='john_doe' password='securepassword123'
   ```

2. 验证一条非法的数据：

   ```python
   user_data = {"username": "john_doe", "password": "123456"}
   user = User(**user_data)
   print(user)
   ```

   输出：

   ```
   Traceback (most recent call last):
     File "D:\workingDirectory\pycharmProject\pydemo\main.py", line 16, in <module>
       user = User(**user_data)
     File "pydantic\main.py", line 341, in pydantic.main.BaseModel.__init__
   pydantic.error_wrappers.ValidationError: 1 validation error for User
   password
     ensure this value has at least 8 characters (type=value_error.any_str.min_length; limit_value=8)
   
   Process finished with exit code 1
   ```

   

### 2. 继承和扩展模型

pydantic库支持模型的继承和扩展，可以在现有模型基础上定义新的模型。

例如，定义一个管理员用户模型，继承自普通用户模型：

```python
from pydantic import BaseModel

class User(BaseModel):
    username: str
    email: str

class AdminUser(User):
    role: str = "admin"

# 创建管理员用户对象并进行数据验证
admin_data = {"username": "admin_user", "email": "admin@example.com", "role": "admin"}
admin_user = AdminUser(**admin_data)
print(admin_user)
```

上述代码定义了一个管理员用户模型`AdminUser`，继承自普通用户模型`User`，并添加了角色字段，默认为管理员角色。

# Pydantic 详解

## 一些概念

### 模型 Models

在 Pydantic 中，模型（Models）主要是从 `pydantic.BaseModel` 继承并将字段定义为带注释的属性的类。如我们定义一个 User 模型：

```python
from pydantic import BaseModel

# 定义一个模型类，继承自 BaseModel
class User(BaseModel):
    # 使用类型注解定义字段
    id: int
    name: str = 'Jane Doe'

# 创建一个 User 实例
user = User(id=123, name="John Doe")

# 访问实例的属性
print(user.id)  # 输出: 123
print(user.name)  # 输出: John Doe
assert isinstance(user.id, int)
```

在这个例子中，`User` 是一个具有两个字段（field）的模型：

- `id`，一个整数，必需
- `name`，一个字符串，非必须（因为有一个默认值）

可以使用 `model_dump()` 方法对模型的实例进行序列化：

```python
assert user.model_dump() == {'id': 123, 'name': 'John Doe'}
```

#### `BaseModel` 的一些常用方法和属性

- [`model_validate()`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_validate): Validates the given object against the Pydantic model. See [Validating data](https://docs.pydantic.dev/latest/concepts/models/#validating-data).
- [`model_validate_json()`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_validate_json): Validates the given JSON data against the Pydantic model. See [Validating data](https://docs.pydantic.dev/latest/concepts/models/#validating-data).
- [`model_construct()`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_construct): Creates models without running validation. See [Creating models without validation](https://docs.pydantic.dev/latest/concepts/models/#creating-models-without-validation).
- [`model_dump()`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_dump): Returns a dictionary of the model's fields and values. See [Serialization](https://docs.pydantic.dev/latest/concepts/serialization/#model_dump).
- [`model_dump_json()`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_dump_json): Returns a JSON string representation of [`model_dump()`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_dump). See [Serialization](https://docs.pydantic.dev/latest/concepts/serialization/#model_dump_json).
- [`model_copy()`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_copy): Returns a copy (by default, shallow copy) of the model. See [Serialization](https://docs.pydantic.dev/latest/concepts/serialization/#model_copy).
- [`model_json_schema()`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_json_schema): Returns a jsonable dictionary representing the model's JSON Schema. See [JSON Schema](https://docs.pydantic.dev/latest/concepts/json_schema/).
- [`model_fields`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_fields): A mapping between field names and their definitions ([`FieldInfo`](https://docs.pydantic.dev/latest/api/fields/#pydantic.fields.FieldInfo) instances).
- [`model_computed_fields`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_computed_fields): A mapping between computed field names and their definitions ([`ComputedFieldInfo`](https://docs.pydantic.dev/latest/api/fields/#pydantic.fields.ComputedFieldInfo) instances).
- [`model_extra`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_extra): The extra fields set during validation.
- [`model_fields_set`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_fields_set): The set of fields which were explicitly provided when the model was initialized.
- [`model_parametrized_name()`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_parametrized_name): Computes the class name for parametrizations of generic classes.
- [`model_post_init()`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_post_init): Performs additional actions after the model is initialized.
- [`model_rebuild()`](https://docs.pydantic.dev/latest/api/base_model/#pydantic.BaseModel.model_rebuild): Rebuilds the model schema, which also supports building recursive generic models. See [Rebuilding model schema](https://docs.pydantic.dev/latest/concepts/models/#rebuilding-model-schema).

### 字段 Fields

Field 函数用于自定义和添加模型的 field 元数据。

#### Field函数的相关入参

1. default - 指定一个字段的默认值

   ```python
   from pydantic import BaseModel, Field
   
   class User(BaseModel):
       name: str = Field(default='John Doe')
       
   user = User()
   print(user)
   #> name='John Doe'
   ```

2. `default_factory` - 定义一个可调用对象（callable），该可调用对象将被调用以生成默认值。

   ```python
   from uuid import uuid4
   
   from pydantic import BaseModel, Field
   
   
   class User(BaseModel):
       id: str = Field(default_factory=lambda: uuid4().hex)
   ```

3. 定义别名的相关入参：

   Pydantic 的 `Field` 中有3个入参可以定义别名，分别为 `alias`、`validation_alias` 和 `serialization_alias`。`alias` 参数用于验证*和*序列化。如果要*分别使用不同的别名*进行验证和序列化，则可以使用`validation_alias` 和 `serialization_alias` 参数，这些参数仅适用于各自的用例。

   ```python
   Field(..., alias='foo')
   Field(..., validation_alias='foo')
   Field(..., serialization_alias='foo')
   ```

   以下的示例展示了别名的使用方法：

   ```py
   from pydantic import BaseModel, Field
   
   
   class User(BaseModel):
       name: str = Field(..., alias='username')
   
   
   user = User(username='johndoe')   # 指定入参username，模型也知道是给 name 这个字段赋值
   print(user)
   #> name='johndoe'
   print(user.model_dump(by_alias=True))  # 序列化的时候该字段使用别名
   #> {'username': 'johndoe'}
   ```

   如果我们起的别名只想用于验证阶段，而不想用于序列化：

   ```py
   from pydantic import BaseModel, Field
   
   
   class User(BaseModel):
       name: str = Field(..., validation_alias='username')
   
   
   user = User(username='johndoe')  
   print(user)
   #> name='johndoe'
   print(user.model_dump(by_alias=True))  
   #> {'name': 'johndoe'}
   ```

   如果我们起的别名只想用于序列化，而不想用于验证阶段：

   ```py
   from pydantic import BaseModel, Field
   
   
   class User(BaseModel):
       name: str = Field(..., serialization_alias='username')
   
   
   user = User(name='johndoe')  
   print(user)
   #> name='johndoe'
   print(user.model_dump(by_alias=True))  
   #> {'username': 'johndoe'}
   ```

4. 数值约束

   有一些关键字参数可用于约束数值：

   - `gt` - greater than 大于
   - `lt` - less than 小于
   - `ge` - greater than or equal to 大于或等于
   - `le` - less than or equal to 小于或等于
   - `multiple_of` - a multiple of the given number 给定数字的倍数
   - `allow_inf_nan` - 允许`'inf'`, `'-inf'`, `'nan'` 这样的值

   下面是一个例子：

   ```py
   from pydantic import BaseModel, Field
   
   
   class Foo(BaseModel):
       positive: int = Field(gt=0)
       non_negative: int = Field(ge=0)
       negative: int = Field(lt=0)
       non_positive: int = Field(le=0)
       even: int = Field(multiple_of=2)
       love_for_pydantic: float = Field(allow_inf_nan=True)
   
   
   foo = Foo(
       positive=1,
       non_negative=0,
       negative=-1,
       non_positive=0,
       even=2,
       love_for_pydantic=float('inf'),
   )
   print(foo)
   """
   positive=1 non_negative=0 negative=-1 non_positive=0 even=2 love_for_pydantic=inf
   """
   ```

5. 字符串约束

   有一些字段可用于约束字符串：

   - `min_length`: Minimum length of the string. 字符串最小长度
   - `max_length`: Maximum length of the string. 字符串最大长度
   - `pattern`: A regular expression that the string must match. 字符串必须匹配的正则表达式

   下面是一个示例：

   ```py
   from pydantic import BaseModel, Field
   
   
   class Foo(BaseModel):
       short: str = Field(min_length=3)
       long: str = Field(max_length=10)
       regex: str = Field(pattern=r'^\d*$')  
   
   
   foo = Foo(short='foo', long='foobarbaz', regex='123')
   print(foo)
   #> short='foo' long='foobarbaz' regex='123'
   ```

更多的字段，请见[官网](https://docs.pydantic.dev/latest/concepts/fields/#string-constraints)。



# 参考资料

1. [pydantic，一个超强的 Python 库！ - Sitin - SegmentFault 思否](https://segmentfault.com/a/1190000044856508)
2. [Welcome to Pydantic - Pydantic 官网](https://docs.pydantic.dev/latest/)

