---
name: pydantic-v2-idiomatic
description: "Write idiomatic Pydantic v2 code — BaseModel, validators, serialization, Field, types, and config. Distilled from official Pydantic v2 docs. Use whenever writing or reviewing Pydantic models, adding validators, serializing data, defining API schemas, or when the user says 'pydantic', 'BaseModel', 'field_validator', 'model_validator', 'model_dump', 'Annotated validator', 'pydantic config', 'serialization', 'discriminated union', or is writing any Python code that uses pydantic for data validation. Also trigger when reviewing code that uses Pydantic v1 patterns (class Config, conint, constr, validator decorator) to modernize it. Do NOT use this for Pydantic AI agent building — use building-pydantic-ai-agents for that."
metadata:
  version: "1.0.0"
  source: "pydantic.dev/docs (June 2026)"
---

# Pydantic v2 — Idiomatic Patterns

Write Pydantic models the way the library intends. This skill covers BaseModel, validators, serialization, fields, types, and config — distilled from the official docs.

For building AI agents with Pydantic AI, use the `building-pydantic-ai-agents` skill instead.

## Quick reference: v1 → v2 migration

| v1 (deprecated) | v2 (use this) |
|---|---|
| `class Config:` | `model_config = ConfigDict(...)` |
| `conint(gt=0)` | `Annotated[int, Field(gt=0)]` |
| `constr(max_length=10)` | `Annotated[str, Field(max_length=10)]` |
| `@validator('field')` | `@field_validator('field')` |
| `@root_validator` | `@model_validator(mode='after')` |
| `.dict()` | `.model_dump()` |
| `.json()` | `.model_dump_json()` |
| `.parse_obj(data)` | `.model_validate(data)` |
| `.parse_raw(json_str)` | `.model_validate_json(json_str)` |
| `.schema()` | `.model_json_schema()` |
| `.copy(update={...})` | `.model_copy(update={...})` |

## Models

### Basic model with config

```python
from pydantic import BaseModel, ConfigDict

class User(BaseModel):
    model_config = ConfigDict(
        str_strip_whitespace=True,
        frozen=True,          # immutable instances
        extra='forbid',       # reject unknown fields
    )
    
    id: int
    name: str
    email: str
```

### Key model methods

```python
# Validate from dict (use for external data)
user = User.model_validate({"id": 1, "name": "Alex", "email": "alex@example.com"})

# Validate from JSON string
user = User.model_validate_json('{"id": 1, "name": "Alex", "email": "alex@example.com"}')

# Serialize
user.model_dump()                    # → dict
user.model_dump_json()               # → JSON string
user.model_dump(exclude_unset=True)  # only fields explicitly provided

# Copy with changes
updated = user.model_copy(update={"name": "New Name"})

# Introspect
User.model_fields          # field definitions
user.model_fields_set      # which fields were explicitly provided
User.model_json_schema()   # JSON Schema
```

### Nested models

```python
class Address(BaseModel):
    street: str
    city: str

class Person(BaseModel):
    name: str
    address: Address  # validates nested dicts automatically

p = Person(name="Alex", address={"street": "123 Main", "city": "Example City"})
```

### Generic models

```python
from typing import Generic, TypeVar

T = TypeVar("T")

class Response(BaseModel, Generic[T]):
    data: T
    count: int

# Usage
Response[list[User]](data=[user], count=1)
```

Python 3.12+: `class Response[T](BaseModel): ...`

## Fields

### Field() essentials

```python
from pydantic import BaseModel, Field

class Product(BaseModel):
    id: int
    name: str = Field(min_length=1, max_length=100)
    price: float = Field(gt=0, description="Price in USD")
    tags: list[str] = Field(default_factory=list)
    internal_id: str = Field(exclude=True)  # never serialized
```

### Aliases

```python
class ApiResponse(BaseModel):
    # validation_alias: accept this key from input
    # serialization_alias: use this key in output
    user_name: str = Field(validation_alias="userName", serialization_alias="user_name")
```

Use `by_alias=True` in `model_dump()` to activate serialization aliases.

### Discriminated unions

```python
from typing import Literal, Union

class Cat(BaseModel):
    pet_type: Literal["cat"]
    meows: int

class Dog(BaseModel):
    pet_type: Literal["dog"]
    barks: float

class Pet(BaseModel):
    animal: Union[Cat, Dog] = Field(discriminator="pet_type")
```

### Default factories with access to other fields

```python
class User(BaseModel):
    email: str
    username: str = Field(default_factory=lambda data: data["email"].split("@")[0])
```

## Validators

Read `references/validators.md` for the complete guide. Quick summary:

### Annotated pattern (preferred for reusability)

```python
from typing import Annotated
from pydantic import AfterValidator, BaseModel

def must_be_positive(v: int) -> int:
    if v <= 0:
        raise ValueError("must be positive")
    return v

PositiveInt = Annotated[int, AfterValidator(must_be_positive)]

class Order(BaseModel):
    quantity: PositiveInt  # reusable across models
```

### Field validator (decorator pattern)

```python
from pydantic import BaseModel, field_validator

class User(BaseModel):
    name: str
    email: str

    @field_validator("name", "email", mode="before")
    @classmethod
    def strip_whitespace(cls, v: str) -> str:
        return v.strip()
```

### Model validator (cross-field validation)

```python
from typing_extensions import Self
from pydantic import BaseModel, model_validator

class DateRange(BaseModel):
    start: date
    end: date

    @model_validator(mode="after")
    def end_after_start(self) -> Self:
        if self.end <= self.start:
            raise ValueError("end must be after start")
        return self
```

### When to use which

| Pattern | When |
|---|---|
| `Annotated[T, AfterValidator(fn)]` | Reusable single-field validation |
| `@field_validator` decorator | Multi-field, same logic; needs access to `cls` |
| `@model_validator(mode="after")` | Cross-field business rules |
| `@model_validator(mode="before")` | Transform raw input before any validation |
| `BeforeValidator` | Coerce/normalize input before type checking |
| `WrapValidator` | Conditional fallback logic |

## Serialization

### model_dump() essentials

```python
user.model_dump()                      # full dict
user.model_dump(exclude_unset=True)    # partial update payloads
user.model_dump(exclude_defaults=True) # compact output
user.model_dump(exclude_none=True)     # strip nulls
user.model_dump(by_alias=True)         # use aliases
user.model_dump(include={"name", "email"})  # whitelist
user.model_dump(exclude={"internal_id"})    # blacklist
```

**Use `model_dump()` not `dict()`.** `dict()` doesn't recursively convert nested models.

### Custom field serializer

```python
from pydantic import BaseModel, field_serializer

class Product(BaseModel):
    price: float

    @field_serializer("price")
    def format_price(self, value: float) -> str:
        return f"${value:.2f}"
```

### Computed fields

Prefer computed fields over `@property` — they're included in serialization automatically:

```python
from pydantic import BaseModel, computed_field

class Rectangle(BaseModel):
    width: float
    height: float

    @computed_field
    @property
    def area(self) -> float:
        return self.width * self.height

r = Rectangle(width=3, height=4)
r.model_dump()  # {"width": 3.0, "height": 4.0, "area": 12.0}
```

## Types

### Constrained types (v2 style)

```python
from typing import Annotated
from pydantic import Field

PositiveInt = Annotated[int, Field(gt=0)]
ShortStr = Annotated[str, Field(max_length=50)]
Percentage = Annotated[float, Field(ge=0, le=100)]
```

### TypeAdapter for standalone validation

Validate without a model:

```python
from pydantic import TypeAdapter

ta = TypeAdapter(list[PositiveInt])
ta.validate_python([1, 2, 3])    # ✓
ta.validate_python([1, -2, 3])   # ValidationError
```

### Strict mode

Reject type coercion (e.g., `"123"` → `123`):

```python
class StrictModel(BaseModel):
    model_config = ConfigDict(strict=True)
    count: int  # "123" will fail, only int accepted
```

Or per-field: `count: int = Field(strict=True)`

## Config reference

```python
from pydantic import ConfigDict

model_config = ConfigDict(
    strict=False,                # True = no type coercion
    frozen=False,                # True = immutable instances
    extra="ignore",              # "forbid" | "allow" | "ignore"
    str_strip_whitespace=False,  # True = auto-strip strings
    str_min_length=None,         # global minimum string length
    validate_default=False,      # True = validate default values
    populate_by_name=False,      # True = accept both alias and field name
    use_enum_values=False,       # True = store enum values not enum members
    validate_assignment=False,   # True = validate on attribute assignment
)
```

## Common mistakes

| Mistake | Fix |
|---|---|
| Using `class Config:` | Use `model_config = ConfigDict(...)` |
| Using `dict(model)` for serialization | Use `model.model_dump()` |
| Using `@validator` | Use `@field_validator` (v2) |
| Using `conint(gt=0)` | Use `Annotated[int, Field(gt=0)]` |
| Using `@property` for computed values | Use `@computed_field` |
| Overriding `__init__` | Use `model_post_init` |
| Catching `ValidationError` from `pydantic` | Import from `pydantic` (it re-exports from `pydantic_core`) |
