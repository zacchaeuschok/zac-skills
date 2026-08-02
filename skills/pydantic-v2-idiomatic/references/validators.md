# Pydantic v2 Validators — Complete Reference

## Validator Types Overview

Four field validator types, each for a different stage of the validation pipeline:

### 1. AfterValidator (default, most common)

Runs after Pydantic's internal type validation. Input is already the correct type.

```python
from typing import Annotated
from pydantic import AfterValidator, BaseModel

def is_even(value: int) -> int:
    if value % 2 == 1:
        raise ValueError(f"{value} is not even")
    return value

EvenNumber = Annotated[int, AfterValidator(is_even)]

class Model(BaseModel):
    number: EvenNumber
```

### 2. BeforeValidator

Runs before type coercion. Input is raw (usually `Any`). Use for normalization.

```python
from typing import Annotated, Any
from pydantic import BeforeValidator, BaseModel

def ensure_list(value: Any) -> Any:
    return [value] if not isinstance(value, list) else value

class Model(BaseModel):
    items: Annotated[list[int], BeforeValidator(ensure_list)]

# Model(items=5) → Model(items=[5])
```

### 3. PlainValidator

Replaces Pydantic's internal validation entirely. No type coercion happens.

```python
from typing import Annotated, Any
from pydantic import PlainValidator, BaseModel

def parse_bool(value: Any) -> bool:
    if isinstance(value, str):
        return value.lower() in ("yes", "true", "1")
    return bool(value)

FlexBool = Annotated[bool, PlainValidator(parse_bool)]
```

### 4. WrapValidator

Most flexible. Gets a `handler` to call Pydantic's validation conditionally.

```python
from typing import Annotated, Any
from pydantic import BaseModel, Field, WrapValidator, ValidatorFunctionWrapHandler

def truncate(value: Any, handler: ValidatorFunctionWrapHandler) -> str:
    try:
        return handler(value)
    except ValidationError:
        return handler(str(value)[:5])

class Model(BaseModel):
    text: Annotated[str, Field(max_length=5), WrapValidator(truncate)]
```

## Decorator Pattern: @field_validator

Use when you need `cls` access or want to apply the same logic across multiple fields:

```python
from pydantic import BaseModel, field_validator

class User(BaseModel):
    first_name: str
    last_name: str
    email: str

    @field_validator("first_name", "last_name", mode="before")
    @classmethod
    def capitalize(cls, v: str) -> str:
        return v.strip().title()

    @field_validator("*", mode="before")  # all fields
    @classmethod
    def no_empty_strings(cls, v: Any) -> Any:
        if isinstance(v, str) and not v.strip():
            raise ValueError("empty string not allowed")
        return v
```

**Important:** always use `@classmethod` with `@field_validator`.

## Model Validators

### mode="after" — cross-field validation

The model is fully constructed. Access fields as attributes.

```python
from typing_extensions import Self
from pydantic import BaseModel, model_validator

class Transfer(BaseModel):
    from_account: str
    to_account: str
    amount: float

    @model_validator(mode="after")
    def accounts_differ(self) -> Self:
        if self.from_account == self.to_account:
            raise ValueError("cannot transfer to same account")
        return self
```

### mode="before" — transform raw input

Input is the raw dict/object before any field validation.

```python
from typing import Any
from pydantic import BaseModel, model_validator

class Config(BaseModel):
    host: str
    port: int

    @model_validator(mode="before")
    @classmethod
    def parse_url(cls, data: Any) -> Any:
        if isinstance(data, str):
            host, port = data.rsplit(":", 1)
            return {"host": host, "port": int(port)}
        return data

# Config.model_validate("localhost:8080") works
```

### mode="wrap" — full control with error handling

```python
from pydantic import BaseModel, ModelWrapValidatorHandler, model_validator

class SafeModel(BaseModel):
    value: int

    @model_validator(mode="wrap")
    @classmethod
    def catch_errors(cls, data: Any, handler: ModelWrapValidatorHandler) -> Self:
        try:
            return handler(data)
        except ValidationError:
            logging.warning(f"Validation failed for {data}")
            raise
```

## Reusable Validators via Annotated

The best pattern for sharing validation logic across models:

```python
from typing import Annotated
from pydantic import AfterValidator

def must_be_positive(v: int) -> int:
    if v <= 0:
        raise ValueError("must be positive")
    return v

def must_be_ascii(v: str) -> str:
    if not v.isascii():
        raise ValueError("must be ASCII")
    return v

PositiveInt = Annotated[int, AfterValidator(must_be_positive)]
AsciiStr = Annotated[str, AfterValidator(must_be_ascii)]

# Reuse anywhere:
class Order(BaseModel):
    quantity: PositiveInt
    sku: AsciiStr

class Inventory(BaseModel):
    count: PositiveInt
    location: AsciiStr
```

Validators compose — stack multiple in `Annotated`:

```python
SafePositiveInt = Annotated[int, AfterValidator(must_be_positive), AfterValidator(must_be_even)]
```

## Validation Context

Pass runtime context to validators:

```python
from pydantic import BaseModel, ValidationInfo, field_validator

class Document(BaseModel):
    content: str

    @field_validator("content")
    @classmethod
    def check_language(cls, v: str, info: ValidationInfo) -> str:
        lang = info.context.get("language", "en") if info.context else "en"
        # validate based on language
        return v

doc = Document.model_validate({"content": "hello"}, context={"language": "en"})
```

## Custom Error Types

```python
from pydantic_core import PydanticCustomError

def validate_age(v: int) -> int:
    if v < 0 or v > 150:
        raise PydanticCustomError(
            "invalid_age",
            "age must be between 0 and 150, got {age}",
            {"age": v},
        )
    return v
```

## Execution Order

When using `Annotated` with multiple validators:
1. **Before/Wrap validators:** right-to-left
2. **After validators:** left-to-right

Decorator validators (`@field_validator`) run after all `Annotated` validators.
