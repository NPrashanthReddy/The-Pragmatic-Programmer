# Pragmatic Python Programmer Guide

This guide turns the ideas in _The Pragmatic Programmer_ into practical Python examples. It is not a replacement for the original summary in [readme.md](./readme.md); it is a companion focused on how to apply the same ideas while writing Python.

The examples are intentionally small. Many sections combine related tips so you can see how pragmatic ideas reinforce each other.

## 1. A Pragmatic Philosophy

### Tip 1: Care About Your Craft
Treat readability, maintainability, and correctness as part of the job.

```python
def format_invoice_total(cents: int) -> str:
    dollars = cents / 100
    return f"${dollars:,.2f}"
```

Small choices such as clear names and type hints are signs of craftsmanship.

### Tip 2: Think! About Your Work
Avoid cargo-cult coding. Ask why code exists and whether it matches the real need.

```python
def find_active_users(users):
    return [user for user in users if user.is_active]
```

This is simpler than building a loop with temporary state just because that is how it was done elsewhere.

### Tip 3: Provide Options, Don't Make Lame Excuses
When something goes wrong, bring a path forward.

```python
from pathlib import Path


def load_config() -> dict:
    primary = Path("config.toml")
    fallback = Path("config.example.toml")

    if primary.exists():
        return {"source": str(primary)}
    if fallback.exists():
        return {"source": str(fallback), "warning": "using example config"}
    raise FileNotFoundError("No config found. Create config.toml from config.example.toml.")
```

### Tip 4: Don't Live with Broken Windows
Clean up small problems before they normalize poor quality.

```python
def slugify(title: str) -> str:
    return title.strip().lower().replace(" ", "-")
```

If `slugify()` is the shared rule, use it everywhere instead of allowing each caller to invent a slightly broken version.

### Tip 5: Be a Catalyst for Change
Start with a small, useful improvement others can join.

```python
def parse_user(row: dict) -> dict:
    return {
        "id": int(row["id"]),
        "name": row["name"].strip(),
        "email": row["email"].lower(),
    }
```

Normalizing one boundary like this often inspires broader cleanup.

### Tip 6: Remember the Big Picture
Optimize for the system, not just the current function.

```python
def fetch_user_profile(user_id: int, client):
    return client.get(f"/users/{user_id}")
```

This function may be fine alone, but if it is called inside a loop it could create an N+1 problem. Keep the larger workflow in mind.

### Tip 7: Make Quality a Requirements Issue
Define quality in terms users care about.

```python
def search_products(query: str, products: list[dict]) -> list[dict]:
    query = query.casefold()
    return [p for p in products if query in p["name"].casefold()]
```

For a real system, "quality" may mean accuracy, latency, accessibility, or clear failure messages. Make those explicit.

### Tip 8: Invest Regularly in Your Knowledge Portfolio
Python rewards steady learning: typing, testing, packaging, async, profiling, data tools.

```python
from collections import Counter

word_counts = Counter("one fish two fish red fish blue fish".split())
```

Learning standard-library tools compounds over time.

### Tip 9: Critically Analyze What You Read and Hear
Not every trend fits your project.

```python
values = [1, 2, 3]
total = sum(values)
```

Use the built-in when it is clearer. Do not replace a simple solution with a fashionable abstraction.

### Tip 10: It's Both What You Say and the Way You Say It
Code communicates too.

```python
def is_eligible_for_refund(order) -> bool:
    return order.is_paid and not order.is_shipped
```

Good names reduce the need for explanation.

## 2. A Pragmatic Approach

### Tips 11 and 12: DRY and Make It Easy to Reuse
Centralize rules so they stay consistent.

```python
TAX_RATE = 0.2


def add_tax(amount: float) -> float:
    return round(amount * (1 + TAX_RATE), 2)


def invoice_total(subtotal: float) -> float:
    return add_tax(subtotal)


def quote_total(subtotal: float) -> float:
    return add_tax(subtotal)
```

One business rule, one authoritative implementation.

### Tip 13: Eliminate Effects Between Unrelated Things
Keep modules focused and independent.

```python
class EmailSender:
    def send(self, address: str, message: str) -> None:
        ...


class UserNotifier:
    def __init__(self, sender: EmailSender) -> None:
        self.sender = sender

    def welcome(self, user) -> None:
        self.sender.send(user.email, "Welcome!")
```

`UserNotifier` depends on an email capability, not on unrelated database or UI details.

### Tip 14: There Are No Final Decisions
Prefer designs that keep change cheap.

```python
def load_data(source: str, loader):
    return loader(source)
```

Passing behavior in keeps you free to swap file, API, or database loaders later.

### Tip 15: Use Tracer Bullets to Find the Target
Build a thin end-to-end path early.

```python
def create_order(raw_order: dict, repository, payment_gateway):
    order = {"user_id": raw_order["user_id"], "total": raw_order["total"]}
    payment_gateway.charge(order["total"])
    repository.save(order)
    return order
```

This is not the final design. It is a visible slice that proves the path works.

### Tip 16: Prototype to Learn
Prototype risky ideas quickly and throw them away.

```python
def benchmark_sort(data):
    import time

    start = time.perf_counter()
    sorted(data)
    return time.perf_counter() - start
```

Useful for learning about performance, not for production architecture.

### Tip 17: Program Close to the Problem Domain
Use domain language in code.

```python
from dataclasses import dataclass


@dataclass
class Money:
    amount: int
    currency: str = "USD"


def apply_discount(price: Money, percentage: int) -> Money:
    discounted = price.amount * (100 - percentage) // 100
    return Money(discounted, price.currency)
```

`Money` is clearer than raw integers spread through the codebase.

### Tips 18 and 19: Estimate to Avoid Surprises, Iterate the Schedule with the Code
Estimate, measure, adjust.

```python
import time


def timed(callable_obj, *args, **kwargs):
    start = time.perf_counter()
    result = callable_obj(*args, **kwargs)
    duration = time.perf_counter() - start
    return result, duration
```

In Python, measurement often teaches more than intuition.

## 3. The Basic Tools

### Tip 20: Keep Knowledge in Plain Text
Text formats are durable and scriptable.

```python
import json

payload = {"name": "Ada", "role": "admin"}
print(json.dumps(payload, indent=2))
```

JSON, TOML, YAML, CSV, and Markdown make automation easy.

### Tip 21: Use the Power of Command Shells
Python and the shell are excellent together.

```python
import subprocess

result = subprocess.run(["python", "--version"], capture_output=True, text=True, check=True)
print(result.stdout.strip())
```

### Tip 22: Use a Single Editor Well
Learn shortcuts, formatting, search, refactoring, test running, and debugging in one editor.

```python
def greet(name: str) -> str:
    return f"Hello, {name}"
```

Even simple code gets faster to write when your editor works with you.

### Tip 23: Always Use Source Code Control
Version control is part of programming, not paperwork.

```python
def calculate_total(items: list[int]) -> int:
    return sum(items)
```

The point is not the code sample; it is that even small changes deserve history and review.

### Tips 24 to 27: Fix the Problem, Don't Panic, "select" Isn't Broken, Prove It
Debug by isolating facts.

```python
def average(values: list[float]) -> float:
    if not values:
        raise ValueError("values must not be empty")
    return sum(values) / len(values)
```

If this fails in production, verify the real input instead of guessing the runtime is broken.

### Tip 28: Learn a Text Manipulation Language
Python itself is a great text manipulation language.

```python
emails = ["  A@example.com", "b@example.com ", "A@example.com "]
normalized = sorted({email.strip().lower() for email in emails})
```

### Tip 29: Write Code That Writes Code
Generate repetitive text from data.

```python
fields = ["id", "name", "email"]
assignments = "\n".join(f"self.{field} = {field}" for field in fields)
print(assignments)
```

Use generation to remove boring duplication, not to hide logic.

## 4. A Pragmatic Paranoia

### Tips 30 and 31: You Can't Write Perfect Software, Design with Contracts
Be explicit about what must be true.

```python
def withdraw(balance: int, amount: int) -> int:
    if amount <= 0:
        raise ValueError("amount must be positive")
    if amount > balance:
        raise ValueError("insufficient funds")
    new_balance = balance - amount
    assert new_balance >= 0
    return new_balance
```

The checks describe the contract, and the assertion protects an invariant.

### Tip 32: Crash Early
Fail as close to the source of corruption as possible.

```python
def parse_port(value: str) -> int:
    port = int(value)
    if not 0 < port < 65536:
        raise ValueError("port out of range")
    return port
```

### Tip 33: Use Assertions to Prevent the Impossible
Use assertions for internal truths, not user-facing validation.

```python
def midpoint(low: int, high: int) -> int:
    assert low <= high
    return (low + high) // 2
```

### Tip 34: Use Exceptions for Exceptional Problems
Do not use exceptions for normal branching.

```python
def read_optional_setting(config: dict, key: str, default=None):
    return config.get(key, default)
```

Missing optional settings are normal; no exception needed.

### Tip 35: Finish What You Start
Use context managers for resources.

```python
from pathlib import Path


def save_report(path: Path, text: str) -> None:
    with path.open("w", encoding="utf-8") as file:
        file.write(text)
```

`with` is one of Python's most pragmatic features.

## 5. Bend or Break

### Tip 36: Minimize Coupling Between Modules
Hide navigation and details behind clear methods.

```python
class Order:
    def __init__(self, items):
        self.items = items

    def total_price(self) -> int:
        return sum(item.price for item in self.items)
```

Call `order.total_price()` instead of reaching through several layers from the outside.

### Tips 37 and 38: Configure, Don't Integrate; Put Abstractions in Code, Details in Metadata
Move decisions into configuration.

```python
PAYMENT_PROVIDERS = {
    "stripe": "payments.stripe.StripeGateway",
    "dummy": "payments.dummy.DummyGateway",
}
```

The application decides _how_ to use a gateway; configuration decides _which_ gateway to use.

### Tips 39 to 41: Analyze Workflow, Design Using Services, Always Design for Concurrency
Break work into independent units.

```python
from concurrent.futures import ThreadPoolExecutor


def fetch_all(fetcher, urls: list[str]):
    with ThreadPoolExecutor() as pool:
        return list(pool.map(fetcher, urls))
```

This example illustrates independent services and concurrency-friendly design.

### Tip 42: Separate Views from Models
Keep domain rules away from presentation.

```python
from dataclasses import dataclass


@dataclass
class Task:
    title: str
    done: bool = False


def render_task(task: Task) -> str:
    status = "✓" if task.done else " "
    return f"[{status}] {task.title}"
```

`Task` is the model; `render_task()` is one possible view.

### Tip 43: Use Blackboards to Coordinate Workflow
Sometimes a queue is the simplest shared coordination space.

```python
from queue import Queue

jobs = Queue()
jobs.put({"kind": "thumbnail", "image_id": 42})
jobs.put({"kind": "email", "user_id": 7})
```

Workers can read from the same queue without tight coupling to each other.

## 6. While You Are Coding

### Tip 44: Don't Program by Coincidence
Make assumptions visible.

```python
def first(items: list[str]) -> str:
    if not items:
        raise ValueError("items must not be empty")
    return items[0]
```

If the code depends on a non-empty list, say so.

### Tips 45 and 46: Estimate the Order of Your Algorithms, Test Your Estimates
Know both theory and real performance.

```python
def contains_duplicate_slow(values: list[int]) -> bool:
    for i, left in enumerate(values):
        for j, right in enumerate(values):
            if i != j and left == right:
                return True
    return False


def contains_duplicate_fast(values: list[int]) -> bool:
    return len(values) != len(set(values))
```

The second version is usually clearer and faster in Python.

### Tip 47: Refactor Early, Refactor Often
Remove duplication when it first becomes obvious.

```python
def normalize_email(email: str) -> str:
    return email.strip().lower()
```

Refactor shared behavior into one function before five call sites drift apart.

### Tips 48 and 49: Design to Test, Test Your Software
Functions with clear inputs and outputs are easier to test.

```python
def apply_coupon(total: int, discount_percent: int) -> int:
    return total * (100 - discount_percent) // 100
```

```python
def test_apply_coupon():
    assert apply_coupon(1000, 10) == 900
```

### Tip 50: Don't Use Wizard Code You Don't Understand
Generated code is acceptable only if you can maintain it.

```python
from dataclasses import dataclass


@dataclass
class User:
    id: int
    name: str
```

This is "generated-like" convenience built into Python, but the result remains easy to understand.

## 7. Before the Project

### Tips 51 and 52: Dig for Requirements, Think Like a User
Represent the problem before rushing into implementation.

```python
def shipping_cost(weight_kg: float, is_express: bool) -> float:
    base = 5.0 if weight_kg <= 1 else 10.0
    return base + (15.0 if is_express else 0.0)
```

The code is simple, but the real work is confirming these rules are actually what users mean.

### Tip 53: Abstractions Live Longer than Details
Abstract stable concepts, not temporary tools.

```python
class MessageStore:
    def save(self, message: str) -> None:
        raise NotImplementedError
```

The abstraction is "store a message," not "write to PostgreSQL version X."

### Tip 54: Use a Project Glossary
Turn shared language into shared names.

```python
class PurchaseOrder:
    ...


class ShoppingCart:
    ...
```

Do not let both names mean the same thing unless the business does.

### Tip 55: Don't Think Outside the Box—Find the Box
Clarify constraints first.

```python
def export_report(rows, writer):
    for row in rows:
        writer.writerow(row)
```

If the real constraint is "must stream, not buffer," the design becomes clearer.

### Tip 56: Start When You're Ready
Nagging doubts often point to missing understanding.

```python
def prototype_parser(raw_text: str) -> list[str]:
    return raw_text.splitlines()
```

When unsure, write a tiny experiment to learn.

### Tip 57: Some Things Are Better Done than Described
Sometimes a small executable example beats a long specification.

```python
def is_weekend(day_name: str) -> bool:
    return day_name.lower() in {"saturday", "sunday"}
```

### Tips 58 and 59: Don't Be a Slave to Formal Methods; Expensive Tools Don't Produce Better Designs
Use tools that fit the team and problem.

```python
def total_seconds(hours: int, minutes: int) -> int:
    return hours * 3600 + minutes * 60
```

Clear code plus lightweight tests may serve better than heavyweight process.

## 8. Pragmatic Projects

### Tip 60: Organize Teams Around Functionality
In Python projects, this often means vertical slices rather than layers nobody owns end to end.

```python
class CheckoutService:
    def __init__(self, inventory, payments, orders):
        self.inventory = inventory
        self.payments = payments
        self.orders = orders
```

One service can reflect one cohesive business capability.

### Tip 61: Don't Use Manual Procedures
Automate repeatable steps.

```python
import pathlib


def iter_python_files(root: str):
    return pathlib.Path(root).rglob("*.py")
```

This kind of small script often replaces error-prone manual work.

### Tips 62 to 66: Test Early, All Tests Run, Use Saboteurs, Test State Coverage, Find Bugs Once
Testing is part of design, not a final phase.

```python
def can_withdraw(balance: int, amount: int) -> bool:
    return 0 < amount <= balance


def test_can_withdraw():
    assert can_withdraw(100, 50) is True
    assert can_withdraw(100, 0) is False
    assert can_withdraw(100, 150) is False
```

These tests cover meaningful states, not just lines.

### Tips 67 and 68: Treat English as a Programming Language, Build Documentation In
Good docstrings and examples belong near the code.

```python
def fahrenheit_to_celsius(value: float) -> float:
    """Convert a Fahrenheit temperature to Celsius."""
    return (value - 32) * 5 / 9
```

Write documentation so it can evolve with the code.

### Tip 69: Gently Exceed Your Users' Expectations
Add thoughtful polish where it matters.

```python
def parse_int(value: str) -> int:
    try:
        return int(value)
    except ValueError as exc:
        raise ValueError(f"Expected a whole number, got {value!r}") from exc
```

The extra mile is often a clearer message, better default, or better example.

### Tip 70: Sign Your Work
Own the behavior of the code you write.

```python
def validate_username(username: str) -> str:
    cleaned = username.strip()
    if len(cleaned) < 3:
        raise ValueError("username must have at least 3 characters")
    return cleaned
```

When you stand behind code like this, you naturally care more about clarity and correctness.

## A Combined Pragmatic Python Example

The following example combines several ideas:

- DRY
- orthogonality
- contracts
- exception handling
- testability
- documentation

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Product:
    name: str
    price_cents: int


def calculate_discounted_total(products: list[Product], discount_percent: int) -> int:
    """Return the final total after applying a percentage discount."""
    if not 0 <= discount_percent <= 100:
        raise ValueError("discount_percent must be between 0 and 100")

    subtotal = sum(product.price_cents for product in products)
    total = subtotal * (100 - discount_percent) // 100
    assert total >= 0
    return total
```

Why this is pragmatic:

- `Product` models the domain directly.
- `calculate_discounted_total()` has one job.
- The contract is explicit.
- The function is easy to test.
- The docstring explains purpose, not implementation noise.
- No UI, database, or framework concerns leak into the logic.

## How to Become a Good Pragmatic Python Programmer

1. Write Python that explains itself through names, boundaries, and small functions.
2. Prefer the standard library until a real problem requires more.
3. Keep business rules in one place.
4. Use exceptions deliberately and assertions internally.
5. Design code so it can be tested without databases, networks, or global state.
6. Automate repetitive work with scripts, tests, and command-line tooling.
7. Measure performance before optimizing.
8. Refactor as soon as duplication or coupling becomes visible.
9. Keep configuration, documentation, and data in text formats.
10. Learn enough Python to recognize the simple solution when it appears.

The core idea is simple: write Python that is easy to change, easy to reason about, and easy for the next programmer to trust.
