
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6643e1656e3447c8aa9044c8811430a4.png)




## 1. 什么是 Pydantic？

Pydantic 是一个 Python 数据验证和数据设置库，基于 Python 类型提示。它提供了一种优雅的方式来解析、验证和转换数据，使代码更清晰、更安全。在处理 JSON 数据、配置文件或外部输入时，Pydantic 是一个非常强大的工具。

------

## 2. Pydantic 的核心功能

1. **数据验证**：基于类型提示（type hints）对输入数据进行验证。
2. **数据解析**：将原始数据转换为指定格式。
3. **错误处理**：提供详细的错误信息，方便调试。
4. **可扩展性**：支持自定义验证器，适应复杂场景。

------

## 3. 实战案例

### 3.1 定义数据模型

使用 `BaseModel` 定义结构化数据模型：

```python

from pydantic import BaseModel, Field

class User(BaseModel):
    id: int
    name: str
    email: str
    age: int = Field(..., gt=0, description="年龄必须为正整数")
    is_active: bool = True

# 测试
data = {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com",
    "age": 25
}

user = User(**data)
print(user.json())
```
输出：

```bash
{"id":1,"name":"Alice","email":"alice@example.com","age":25,"is_active":true}
```

### 3.2 数据验证与错误处理

当数据不符合定义时，Pydantic 会抛出详细的验证错误：

```python

invalid_data = {
    "id": "abc",  # 错误：id 应该是整数
    "name": "Alice",
    "email": "alice@example.com",
    "age": -5  # 错误：年龄为负数
}

try:
    user = User(**invalid_data)
except Exception as e:
    print(e)
```
输出：

```bash
 validation errors for User
id
  Input should be a valid integer, unable to parse string as an integer [type=int_parsing, input_value='abc', input_type=str]
    For further information visit https://errors.pydantic.dev/2.9/v/int_parsing
age
  Input should be greater than 0 [type=greater_than, input_value=-5, input_type=int]
    For further information visit https://errors.pydantic.dev/2.9/v/greater_than
```
输出示例：

```python
1 validation error for User
age
  ensure this value is greater than 0 (type=value_error.number.not_gt; limit_value=0)
```

------

### 3.3 自定义验证器

通过 `@validator` 修饰器实现复杂规则：

```python

from pydantic import BaseModel, validator

class Product(BaseModel):
    name: str
    price: float
    quantity: int

    @validator("price")
    def price_must_be_positive(cls, value):
        if value <= 0:
            raise ValueError("价格必须为正数")
        return value

    @validator("quantity")
    def quantity_must_be_non_negative(cls, value):
        if value < 0:
            raise ValueError("数量不能为负数")
        return value
```

### 3.4 默认字段

Pydantic还提供了一些高级功能，如默认值、可选字段和自定义数据类型。让我们扩展我们的用户模型以包含这些功能：

```bash
from typing import Optional
from pydantic import BaseModel, EmailStr

class User(BaseModel):
    name: str
    age: int
    email: EmailStr
    phone: Optional[str] = None

```
在这个新模型中，我们：

- 将email字段更改为EmailStr类型，以便Pydantic对电子邮件进行验证。
- 添加了一个可选的phone字段，它具有一个默认值None。这意味着我们在创建用户时可以忽略该字段。

现在我们可以创建一个包含电子邮件验证和可选电话号码的用户：

```bash
user = User(name="Alice", age=30, email="alice@example.com")  # 有效
user = User(name="Alice", age=30, email="invalid_email")  # 错误：无效的电子邮件
```

### 3.5 Field对象


```bash
from pydantic import BaseModel, Field

class Product(BaseModel):
    name: str
    price: float = Field(..., description="商品价格", gt=1, lt=1000)
```
在这个例子中，我们为price字段添加了Field对象，其中：

- ...表示该字段是必填项。
- description定义了字段的描述信息。
- gt 表示字段的取值必须大于1。
- lt 表示字段的取值必须小于1000。

现在，当我们尝试使用无效的价格创建商品时，Pydantic将会引发一个异常：

```bash
product = Product(name="Laptop", price=0)  # 错误：价格必须大于1
```

Field函数提供了许多参数来定制字段的行为。以下是一些常用的参数：

- default：定义字段的默认值。如果未提供该值，则默认为None。
- alias：定义字段的别名。这在处理不符合Python变量命名规则的字段名时非常有用（例如，包含空格或连字符的字段名）。
- title：定义字段的标题。这在生成文档时非常有用。
- description：定义字段的描述信息。这在生成文档时非常有用。
- min_length和max_length：针对字符串类型的字段定义最小和最大长度限制。
- gt、ge、lt和le：针对数值类型的字段定义大于（gt）、大于等于（ge）、小于（lt）和小于等于（le）的限制。


------


## 4. 使用场景

1. **API 数据验证**：快速验证请求体数据。
2. **配置管理**：加载并验证环境变量。
3. **数据解析**：从 JSON 或数据库结果中提取和验证数据。
4. **表单校验**：用于用户输入的高效验证。

------

## 5. 总结

Pydantic 是一个高效且强大的数据验证工具，结合 Python 类型提示提供了优雅的解决方案。通过丰富的功能（如 `BaseModel`、`BaseSettings` 和自定义验证器），开发者可以更轻松地编写健壮的代码。



- [Pydantic 官方文档](https://docs.pydantic.dev/latest/)
- [Pydantic GitHub 仓库](https://github.com/pydantic/pydantic)
