> From: https://yuanbao.tencent.com/chat/naQivTmsDa/0PbSckkf0RU

# you asked

message time: 2026-08-14 22:50:42

based on this python script , use model_validate() to replace usermodel() class, explain why better

"""
from typing import Literal
from pydantic import BaseModel, ValidationError

# Testing data
mock_orders = [
    {"order_id": 101, "price": "29.99", "item_name": "Logitech Mouse", "status": "PENDING"},
    {"order_id": 102, "price": 1099.00, "item_name": "4K Monitor", "status": "INVALID_STATUS"},
    {"order_id": "one-hundred-three", "price": 5.50, "item_name": "HDMI Cable", "status": "SHIPPED"}
]

# 1. Define the Firewall Schema, THIS IS THE CORE CONFIG
class OrderModel(BaseModel):
    order_id: int
    price: float
    item_name: str
    status: Literal["PENDING", "SHIPPED", "DELIVERED"]


# 2. Pass data through the firewall
for o in mock_orders:
    print(o)
    try:
        order = OrderModel(**o)    
        print(f"Order {order.order_id} is clean.")
    except ValidationError as e:
        # Loop through the structured errors Pydantic provides
        for error in e.errors():
            field = " -> ".join(str(loc) for loc in error["loc"])
            error_msg = error["msg"]
            print(f"   • Field [{field}]: {error_msg}")
"""

---

# yuanbao response

Here is the updated script using `model_validate()`, followed by the explanation.

## Updated Script

```python
from typing import Literal
from pydantic import BaseModel, ValidationError

# Testing data
mock_orders = [
    {"order_id": 101, "price": "29.99", "item_name": "Logitech Mouse", "status": "PENDING"},
    {"order_id": 102, "price": 1099.00, "item_name": "4K Monitor", "status": "INVALID_STATUS"},
    {"order_id": "one-hundred-three", "price": 5.50, "item_name": "HDMI Cable", "status": "SHIPPED"}
]

# 1. Define the Firewall Schema
class OrderModel(BaseModel):
    order_id: int
    price: float
    item_name: str
    status: Literal["PENDING", "SHIPPED", "DELIVERED"]


# 2. Pass data through the firewall
for o in mock_orders:
    print(o)
    try:
        order = OrderModel.model_validate(o)
        print(f"Order {order.order_id} is clean.")
    except ValidationError as e:
        for error in e.errors():
            field = " -> ".join(str(loc) for loc in error["loc"])
            error_msg = error["msg"]
            print(f"   - Field [{field}]: {error_msg}")
```

The only change is on line 22, replacing `OrderModel(**o)` with `OrderModel.model_validate(o)`.

---

## Why `model_validate()` Is Better

As a data engineer, you are typically ingesting data from external sources — APIs, Kafka topics, CSV files, databases. That data arrives as dictionaries, JSON strings, or row objects. Here is why `model_validate()` is the right tool for that job:

### 1. Explicit Intent

`OrderModel(**o)` looks like standard Python object instantiation. It silently relies on dictionary unpacking. `OrderModel.model_validate(o)` makes it crystal clear to anyone reading your pipeline code that you are **validating external data against a schema**. In a data engineering context, readability and intent matter — your future self and your teammates will immediately understand what is happening.

### 2. Consistent API with `model_validate_json()`

In real pipelines, data often arrives as raw JSON strings (from a message broker, an HTTP response, or a file read). Pydantic provides a parallel method for that:

```python
order = OrderModel.model_validate_json(raw_json_string)
```

Using `model_validate()` for dicts and `model_validate_json()` for strings gives you a consistent, symmetrical API. Compare that to the alternative, where you would have to manually parse JSON with `json.loads()` and then call `OrderModel(**parsed_dict)` — two steps, two different patterns.

### 3. Handles Arbitrary Mappings, Not Just Dicts

`OrderModel(**o)` requires `o` to be a `dict` (or at least support `**` unpacking). `model_validate()` accepts any mapping-like object — including `collections.abc.Mapping` implementations, `SimpleNamespace`, or custom objects returned by certain libraries. This makes your ingestion layer more flexible and resilient to changes in upstream data formats.

### 4. Proper Separation of Concerns

`**kwargs` instantiation calls `__init__` directly, mixing the concern of "construct a Python object" with "validate incoming data." `model_validate()` is a classmethod purpose-built for the validation pathway. It ensures all of Pydantic's validation machinery — coercers, validators, strict mode handling — is invoked consistently. If you ever enable `strict=True` on your model config, `model_validate()` respects it properly, whereas direct instantiation can bypass certain checks.

### 5. Future-Proofing (Pydantic V2)

In Pydantic V2, `model_validate()` is the officially recommended entry point for creating model instances from data. Direct `**kwargs` instantiation still works, but it is increasingly treated as an internal implementation detail. Writing your pipelines with `model_validate()` today means fewer surprises during future upgrades.

---

### Summary

| Approach | Readability | Handles JSON strings | Handles non-dict mappings | Recommended |
|----------|------------|---------------------|--------------------------|-------------|
| `OrderModel(**o)` | Unclear intent | No (needs manual parse) | No | No |
| `OrderModel.model_validate(o)` | Clear intent | No (use `model_validate_json`) | Yes | **Yes** |

In short: `model_validate()` tells the truth about what your code is doing. You are validating data, not just constructing objects.

---

Would you like me to extend this example to show how `model_validate_json()` fits into a realistic data ingestion pipeline, or perhaps demonstrate how to use Pydantic's `TypeAdapter` for validating lists of records at once?

---

# you asked

message time: 2026-08-14 22:59:26

how to use pydantic for functions

---

# yuanbao response

There are two main ways to use Pydantic with functions: the `validate_call` decorator (the built-in approach), and manual validation using models or `TypeAdapter` inside your function body. Let me walk through both.

---

## Approach 1: `validate_call` Decorator (Pydantic V2)

This is the most direct way. You decorate a function, and Pydantic automatically validates both **arguments** and **return values** based on your type hints.

```python
from typing import Literal
from pydantic import BaseModel, validate_call, ValidationError

class Order(BaseModel):
    order_id: int
    price: float
    item_name: str
    status: Literal["PENDING", "SHIPPED", "DELIVERED"]

@validate_call
def process_order(
    order_id: int,
    price: float,
    item_name: str,
    status: Literal["PENDING", "SHIPPED", "DELIVERED"]
) -> dict:
    # At this point, all inputs are already validated and coerced.
    # order_id is guaranteed to be int, price is float, status is one of the three literals.
    discount = 0.0 if price > 500 else 0.10
    return {
        "order_id": order_id,
        "final_price": round(price * (1 - discount), 2),
        "discount_applied": discount > 0
    }

# Valid call - works fine, price gets coerced from string
result = process_order(101, "29.99", "Logitech Mouse", "PENDING")
print(result)
# {'order_id': 101, 'final_price': 26.99, 'discount_applied': True}

# Invalid call - raises ValidationError
try:
    process_order(102, 1099.00, "4K Monitor", "INVALID_STATUS")
except ValidationError as e:
    print(e)
```

### What happens under the hood:

1. Before the function body runs, Pydantic checks each argument against its type hint. `"29.99"` is coerced to `29.99` because it is a valid numeric string. `"INVALID_STATUS"` fails because it is not in the `Literal`.
2. If validation fails, the function body **never executes**.
3. By default, return values are NOT validated. You need to opt in (see below).

### Validating Return Values

```python
@validate_call(validate_return=True)
def fetch_order(order_id: int) -> Order:
    # Imagine this comes from a database or API
    raw = {"order_id": order_id, "price": "49.99", "item_name": "Keyboard", "status": "PENDING"}
    return Order.model_validate(raw)

# The return value is validated against the Order type hint.
# Since Order is a BaseModel, Pydantic recognizes it and validates accordingly.
order = fetch_order(201)
print(type(order))  # <class '__main__.Order'>
```

### Configuration

You can pass a `config` dict to control strictness:

```python
@validate_call(config={"strict": True})
def strict_add(a: int, b: int) -> int:
    return a + b

strict_add(1, 2)      # OK
strict_add(1, "2")    # ValidationError - no coercion in strict mode
```

---

## Approach 2: Manual Validation with Models (More Control)

In data engineering pipelines, you often want more explicit control — different error handling, logging, partial success, dead-letter queues. In that case, define a model and call `model_validate()` at the boundary of your function.

```python
from pydantic import BaseModel, ValidationError
from typing import Literal

class ProcessOrderInput(BaseModel):
    order_id: int
    price: float
    item_name: str
    status: Literal["PENDING", "SHIPPED", "DELIVERED"]

class ProcessOrderOutput(BaseModel):
    order_id: int
    final_price: float
    discount_applied: bool

def process_order(raw_data: dict) -> ProcessOrderOutput | None:
    try:
        # Validate input explicitly
        validated = ProcessOrderInput.model_validate(raw_data)
    except ValidationError as e:
        # You control what happens here: log, send to dead-letter, skip, alert
        print(f"Skipping invalid order: {e}")
        return None

    # Business logic with fully trusted data
    discount = 0.0 if validated.price > 500 else 0.10
    final_price = round(validated.price * (1 - discount), 2)

    # Validate output before returning
    return ProcessOrderOutput(
        order_id=validated.order_id,
        final_price=final_price,
        discount_applied=discount > 0
    )

# Usage
mock_orders = [
    {"order_id": 101, "price": "29.99", "item_name": "Logitech Mouse", "status": "PENDING"},
    {"order_id": 102, "price": 1099.00, "item_name": "4K Monitor", "status": "INVALID_STATUS"},
]

for o in mock_orders:
    result = process_order(o)
    if result:
        print(f"Processed: {result.model_dump()}")
```

### Why this is often better for data engineering:

| Aspect | `validate_call` | Manual model validation |
|--------|----------------|------------------------|
| Error handling | Raises immediately, function does not run | You decide: skip, log, retry, dead-letter |
| Batch processing | One error stops everything | You can process N-1 out of N records |
| Schema reuse | Type hints only, harder to reuse | Model can be shared across functions, services |
| Self-documenting | Type hints serve as docs | Model acts as explicit contract/schema |
| Serialization | Not built in | `.model_dump()`, `.model_dump_json()` ready |

---

## Approach 3: `TypeAdapter` for Lightweight Validation

When defining a full `BaseModel` feels heavy but you still want validation, use `TypeAdapter`:

```python
from pydantic import TypeAdapter, ValidationError
from typing import Literal

# Define a TypeAdapter for a single complex type
OrderStatus = Literal["PENDING", "SHIPPED", "DELIVERED"]
status_adapter = TypeAdapter(OrderStatus)

def update_status(order_id: int, status: str) -> None:
    try:
        validated_status = status_adapter.validate_python(status)
    except ValidationError:
        print(f"Invalid status '{status}' for order {order_id}")
        return
    print(f"Order {order_id} updated to {validated_status}")

update_status(101, "PENDING")          # OK
update_status(102, "CANCELLED")        # Prints error, returns gracefully
```

This is useful when you only need to validate one or two fields, not a full structured record.

---

## Which Should You Use?

| Use Case | Recommended Approach |
|----------|---------------------|
| Quick scripts, CLI tools, small utilities | `@validate_call` |
| Data pipeline ingestion functions | Manual `model_validate()` with try/except |
| Batch record processing | Manual with models, loop-level error handling |
| Validating a single field or simple type | `TypeAdapter` |
| API endpoint handlers (FastAPI-style) | Models as request bodies (FastAPI uses `model_validate` internally) |

The key principle: **validate at the boundary, not in the middle of your logic.** Whether you use `@validate_call` or manual `model_validate()`, the goal is the same — by the time your business logic runs, the data is already trusted.

---

Would you like me to show how to combine `TypeAdapter` with `model_validate()` to validate a list of orders in a single batch call, or demonstrate how to implement a dead-letter pattern for failed validations?

---
Powered by [AI Exporter](https://saveai.net)