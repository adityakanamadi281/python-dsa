# Senior AI/ML Engineer — Python Interview Handbook
### 150 Code-Based Questions (OOP · File Handling · Exceptions · Tricky Python · Nested Data Structures · Functions/Scope · Iterators/Generators · Decorators/Closures · Context Managers · Concurrency)

**How to use this handbook:**
Show the candidate the code only. Ask them to predict the output *before* running it, explain *why*, identify the trick/bug, and propose a fix. Their reasoning matters more than the final answer.

Legend: 🟢 Easy · 🟡 Medium · 🔴 Hard

---

# PART 1 — OOP (Q1–Q30)

### Q1. Mutable Class Attribute Trap 🟡
```python
class A:
    x = []
    def __init__(self):
        self.x.append(1)

a = A()
b = A()
print(a.x)
print(b.x)
```
**Output:** `[1, 1]` then `[1, 1]`
**Explanation:** `x` is a class variable, shared across all instances. `self.x.append(1)` mutates the shared list rather than creating an instance attribute.
**Fix:** Define `x` inside `__init__` as `self.x = []`.
**Follow-up:** Difference between class variable and instance variable; when Python creates a new instance attribute vs. resolving to the class attribute.

### Q2. Class vs Instance Variable Shadowing 🟢
```python
class Counter:
    count = 0
    def __init__(self):
        Counter.count += 1

c1 = Counter()
c2 = Counter()
c1.count = 100
print(c1.count, c2.count, Counter.count)
```
**Output:** `100 2 2`
**Explanation:** `c1.count = 100` creates a new instance attribute on `c1`, shadowing the class attribute; it does not affect `Counter.count`.
**Follow-up:** How attribute lookup order works (`instance.__dict__` → `type(instance).__mro__`).

### Q3. `__init__` vs `__new__` 🔴
```python
class Singleton:
    _instance = None
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    def __init__(self, value):
        self.value = value

s1 = Singleton(1)
s2 = Singleton(2)
print(s1.value, s2.value, s1 is s2)
```
**Output:** `2 2 True`
**Explanation:** `__new__` returns the same object each time, but `__init__` still runs on every call, overwriting `value`.
**Follow-up:** How to implement a true singleton that ignores subsequent `__init__` calls.

### Q4. Multiple Inheritance & MRO 🔴
```python
class A:
    def greet(self): print("A")

class B(A):
    def greet(self): print("B")

class C(A):
    def greet(self): print("C")

class D(B, C):
    pass

D().greet()
print(D.__mro__)
```
**Output:** `B` then MRO `(D, B, C, A, object)`
**Explanation:** Python uses C3 linearization; `D` inherits `B`'s `greet` first because `B` precedes `C` in the base list.
**Follow-up:** What happens if `B` and `C` had incompatible MROs (would raise `TypeError`).

### Q5. `super()` Diamond Problem 🔴
```python
class A:
    def __init__(self):
        print("A init")
        super().__init__()

class B(A):
    def __init__(self):
        print("B init")
        super().__init__()

class C(A):
    def __init__(self):
        print("C init")
        super().__init__()

class D(B, C):
    def __init__(self):
        print("D init")
        super().__init__()

D()
```
**Output:** `D init`, `B init`, `C init`, `A init`
**Explanation:** `super()` follows MRO cooperatively, not each class's direct parent — this is why every class must call `super().__init__()` for the chain to complete.
**Follow-up:** Why cooperative multiple inheritance requires every class in the hierarchy to call `super()`.

### Q6. Private Attribute Name Mangling 🟡
```python
class A:
    def __init__(self):
        self.__secret = 42
    def reveal(self):
        return self.__secret

a = A()
print(a.reveal())
print(a._A__secret)
try:
    print(a.__secret)
except AttributeError as e:
    print("Error:", e)
```
**Output:** `42`, `42`, then `Error: 'A' object has no attribute '__secret'`
**Explanation:** Double-underscore attributes are name-mangled to `_ClassName__attr`; they're not truly private, just harder to access accidentally.
**Follow-up:** Single underscore vs. double underscore convention; Python has no real access modifiers.

### Q7. Class Method vs Static Method vs Instance Method 🟢
```python
class Demo:
    def instance_method(self):
        return "instance", self
    @classmethod
    def class_method(cls):
        return "class", cls
    @staticmethod
    def static_method():
        return "static"

d = Demo()
print(d.instance_method())
print(d.class_method())
print(d.static_method())
print(Demo.class_method())
```
**Explanation:** `classmethod` binds to the class (useful for alternative constructors); `staticmethod` binds to neither; both can be called via instance or class.
**Follow-up:** When would you use `classmethod` for factory patterns in ML pipelines (e.g., `Model.from_config()`).

### Q8. Property Decorators & Encapsulation 🟡
```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius
    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32
    @fahrenheit.setter
    def fahrenheit(self, value):
        self._celsius = (value - 32) * 5/9

t = Temperature(0)
print(t.fahrenheit)
t.fahrenheit = 212
print(t._celsius)
```
**Output:** `32.0` then `100.0`
**Explanation:** `@property` exposes computed attributes with getter/setter syntax while keeping the underlying value encapsulated.
**Follow-up:** Why prefer properties over public raw attributes for validation logic.

### Q9. Abstract Base Classes 🟡
```python
from abc import ABC, abstractmethod

class Model(ABC):
    @abstractmethod
    def predict(self, x): ...

class LinearModel(Model):
    def predict(self, x):
        return x * 2

try:
    m = Model()
except TypeError as e:
    print("Error:", e)

print(LinearModel().predict(5))
```
**Explanation:** ABCs with `@abstractmethod` cannot be instantiated directly; subclasses must implement all abstract methods.
**Follow-up:** Why ABCs are preferred over duck typing when defining an ML model interface contract.

### Q10. Operator Overloading 🟡
```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)
```
**Output:** `Vector(4, 6)`
**Explanation:** `__add__` overloads `+`; `__repr__` controls how the object prints.
**Follow-up:** Difference between `__repr__` and `__str__`; what happens without `__radd__` if left operand doesn't support `+`.

### Q11. `__eq__` and `__hash__` Consistency 🔴
```python
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

p1, p2 = Point(1, 2), Point(1, 2)
print(p1 == p2)
s = {p1, p2}
print(len(s))
```
**Output:** `True` then `2` (not 1!)
**Explanation:** Overriding `__eq__` without `__hash__` sets `__hash__` to `None`... actually it makes the class unhashable in Py3 by default only if `__hash__` isn't explicitly kept — here default object hash (identity-based) is retained since `__hash__` wasn't touched in this version subtlety; correct behavior is: defining `__eq__` sets `__hash__` to None automatically, so `{p1, p2}` raises `TypeError: unhashable type`.
**Corrected Output:** `TypeError: unhashable type: 'Point'`
**Follow-up:** Why `__eq__` and `__hash__` must be defined together; hashable contract for set/dict membership.

### Q12. Method Resolution with `__getattr__` 🔴
```python
class Lazy:
    def __getattr__(self, name):
        print(f"Computing {name}")
        return 42

l = Lazy()
print(l.anything)
print(l.__dict__)
```
**Explanation:** `__getattr__` is only called when normal attribute lookup fails, so it acts as a fallback/lazy-loader; `__dict__` itself is found normally and doesn't trigger it.
**Follow-up:** Difference between `__getattr__` and `__getattribute__`.

### Q13. Slots and Memory Optimization 🟡
```python
class Point:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x, self.y = x, y

p = Point(1, 2)
try:
    p.z = 3
except AttributeError as e:
    print("Error:", e)
```
**Explanation:** `__slots__` prevents creation of `__dict__`, saving memory and blocking arbitrary new attributes — useful for large numbers of lightweight ML data objects.
**Follow-up:** Trade-offs of `__slots__` with inheritance and pickling.

### Q14. Metaclasses Basics 🔴
```python
class Meta(type):
    def __new__(mcs, name, bases, namespace):
        namespace['created_by'] = 'Meta'
        return super().__new__(mcs, name, bases, namespace)

class MyClass(metaclass=Meta):
    pass

print(MyClass.created_by)
print(type(MyClass))
```
**Output:** `Meta` then `<class '__main__.Meta'>`
**Explanation:** Metaclasses control class creation itself; `type` is the default metaclass.
**Follow-up:** Real-world use cases (Django ORM models, ABC registration, enforcing coding conventions).

### Q15. Composition vs Inheritance 🟢
```python
class Engine:
    def start(self): return "Engine started"

class Car:
    def __init__(self):
        self.engine = Engine()
    def start(self):
        return self.engine.start()

print(Car().start())
```
**Explanation:** Composition ("has-a") over inheritance ("is-a") — more flexible, avoids deep hierarchies.
**Follow-up:** When to prefer composition in ML pipeline design (e.g., a `Pipeline` composed of `Preprocessor`, `Model`, `Evaluator`).

### Q16. Dataclasses and Mutable Defaults 🟡
```python
from dataclasses import dataclass, field

@dataclass
class Config:
    layers: list = field(default_factory=list)

c1 = Config()
c2 = Config()
c1.layers.append(64)
print(c1.layers, c2.layers)
```
**Output:** `[64] []`
**Explanation:** `field(default_factory=list)` gives each instance its own list, avoiding the classic mutable default trap.
**Follow-up:** What happens if you naively write `layers: list = []` in a dataclass (raises `ValueError`).

### Q17. `__call__` Making Objects Callable 🟡
```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor
    def __call__(self, x):
        return x * self.factor

double = Multiplier(2)
print(double(5))
print(callable(double))
```
**Explanation:** `__call__` lets instances behave like functions — commonly used for stateful transforms/callbacks in ML code (e.g., torchvision transforms).
**Follow-up:** How PyTorch's `nn.Module.__call__` wraps `forward()`.

### Q18. Class Attribute Resolution Order with Instance Override 🟡
```python
class Base:
    def show(self):
        return "Base"

class Derived(Base):
    pass

d = Derived()
d.show = lambda: "Instance override"
print(d.show())
print(Derived().show())
```
**Output:** `Instance override` then `Base`
**Explanation:** Assigning a function directly to an instance attribute shadows the class method for that instance only.
**Follow-up:** Why this doesn't work the same way for dunder methods (they bypass instance `__dict__`).

### Q19. Copy vs Deepcopy with Custom Objects 🔴
```python
import copy

class Node:
    def __init__(self, value, children=None):
        self.value = value
        self.children = children or []

root = Node(1, [Node(2)])
shallow = copy.copy(root)
deep = copy.deepcopy(root)
shallow.children[0].value = 99
print(root.children[0].value)
print(deep.children[0].value)
```
**Output:** `99` then `2`
**Explanation:** `copy.copy` duplicates the outer object but shares nested mutable references; `deepcopy` recursively clones everything.
**Follow-up:** Why deep copying large model weights/tensors can be expensive; when shallow copy suffices.

### Q20. Descriptor Protocol 🔴
```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.name = "_" + name
    def __get__(self, instance, owner):
        return getattr(instance, self.name)
    def __set__(self, instance, value):
        if value < 0:
            raise ValueError("Must be positive")
        setattr(instance, self.name, value)

class Product:
    price = PositiveNumber()
    def __init__(self, price):
        self.price = price

p = Product(10)
print(p.price)
try:
    p.price = -5
except ValueError as e:
    print("Error:", e)
```
**Explanation:** Descriptors implement reusable attribute validation logic (this is how `@property` works under the hood).
**Follow-up:** Difference between data descriptors (`__set__`) and non-data descriptors (only `__get__`).

### Q21. Inheritance and `super()` with Arguments 🟡
```python
class Animal:
    def __init__(self, name):
        self.name = name

class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)
        self.breed = breed

d = Dog("Rex", "Labrador")
print(d.name, d.breed)
```
**Follow-up:** Why `super().__init__(name)` is preferred over `Animal.__init__(self, name)` in cooperative hierarchies.

### Q22. Class Variable Mutation via Subclass 🟡
```python
class Base:
    items = []

class Child(Base):
    pass

Child.items.append(1)
print(Base.items, Child.items)
```
**Output:** `[1] [1]`
**Explanation:** `Child` doesn't have its own `items`; it shares the same list object from `Base` until explicitly reassigned.
**Follow-up:** How this differs once `Child.items = []` is explicitly set (breaks the shared reference).

### Q23. Polymorphism with Duck Typing 🟢
```python
class Duck:
    def sound(self): return "Quack"

class Dog:
    def sound(self): return "Woof"

for animal in [Duck(), Dog()]:
    print(animal.sound())
```
**Explanation:** Python doesn't require a common base class for polymorphism — if it "quacks," it works ("duck typing").
**Follow-up:** How this relates to `Protocol` typing (structural subtyping) in `typing` module.

### Q24. Object Identity vs Equality in Inheritance 🟡
```python
class A:
    def __eq__(self, other):
        return isinstance(other, A)

a = A()
print(a == A())
print(a == object())
print(a is A())
```
**Output:** `True`, `False`, `False`
**Follow-up:** `==` calls `__eq__`; `is` checks identity regardless of `__eq__` overrides.

### Q25. Overriding `__len__` and `__bool__` 🟡
```python
class Bucket:
    def __init__(self, items):
        self.items = items
    def __len__(self):
        return len(self.items)

b = Bucket([])
print(bool(b))
b2 = Bucket([1])
print(bool(b2))
```
**Output:** `False` then `True`
**Explanation:** Without `__bool__`, Python falls back to `__len__`; an object with `len() == 0` is falsy.
**Follow-up:** Order of lookup: `__bool__` → `__len__` → default `True`.

### Q26. Class-level vs Instance-level Method Binding 🔴
```python
class A:
    def method(self):
        return "bound"

a = A()
unbound = A.method
bound = a.method
print(unbound(a))
print(bound())
```
**Explanation:** `A.method` is a plain function requiring an explicit instance; `a.method` is already bound to `a`.
**Follow-up:** How this relates to how Python implements methods as descriptors on functions.

### Q27. Interface Segregation via Mixins 🟡
```python
class JSONMixin:
    def to_json(self):
        import json
        return json.dumps(self.__dict__)

class User(JSONMixin):
    def __init__(self, name):
        self.name = name

print(User("Alice").to_json())
```
**Explanation:** Mixins add reusable behavior without deep inheritance chains — common in ML serialization utilities.
**Follow-up:** Mixin ordering rules and MRO conflicts.

### Q28. `__init_subclass__` Hook 🔴
```python
class Plugin:
    registry = []
    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        Plugin.registry.append(cls)

class PluginA(Plugin): pass
class PluginB(Plugin): pass

print(Plugin.registry)
```
**Explanation:** `__init_subclass__` auto-registers subclasses — a lightweight alternative to metaclasses, often used for model/optimizer registries in ML frameworks.
**Follow-up:** Compare to `__init_subclass__` vs decorator-based registration patterns.

### Q29. Multiple Dispatch Simulation with `functools.singledispatchmethod` 🔴
```python
from functools import singledispatchmethod

class Formatter:
    @singledispatchmethod
    def format(self, value):
        return str(value)
    @format.register
    def _(self, value: int):
        return f"Int: {value}"
    @format.register
    def _(self, value: list):
        return f"List of {len(value)} items"

f = Formatter()
print(f.format(5))
print(f.format([1,2,3]))
print(f.format(3.14))
```
**Follow-up:** How this compares to overloading in statically-typed languages; limitations with generics.

### Q30. Circular References and Garbage Collection 🔴
```python
import gc

class Node:
    def __init__(self):
        self.ref = None

a = Node()
b = Node()
a.ref = b
b.ref = a
del a, b
print(gc.collect())
```
**Explanation:** CPython's reference counting alone can't reclaim circular references; the cyclic garbage collector (`gc.collect()`) is required, and it reports how many unreachable objects it collected.
**Follow-up:** Why this matters for large object graphs in ML pipelines (e.g., custom computation graphs holding cyclic parent/child references).

---

# PART 2 — FILE HANDLING (Q31–Q50)

### Q31. Context Manager Auto-close 🟢
```python
with open('/tmp/test.txt', 'w') as f:
    f.write("Hello")
print(f.closed)
```
**Output:** `True`
**Explanation:** The `with` statement guarantees `__exit__` (which closes the file) runs even if an exception occurs.
**Follow-up:** Why raw `f = open(...)` without `with`/`close()` risks file descriptor leaks.

### Q32. Reading Modes: `r` vs `rb` 🟡
```python
with open('/tmp/test.txt', 'w') as f:
    f.write("héllo")

with open('/tmp/test.txt', 'r', encoding='utf-8') as f:
    print(f.read())

with open('/tmp/test.txt', 'rb') as f:
    print(f.read())
```
**Explanation:** Text mode decodes bytes to `str` using the given encoding; binary mode returns raw `bytes`.
**Follow-up:** Why encoding mismatches (e.g., reading UTF-8 as Latin-1) cause silent corruption vs `UnicodeDecodeError`.

### Q33. `readlines()` vs Iterating a File Object 🟢
```python
with open('/tmp/lines.txt', 'w') as f:
    f.write("a\nb\nc\n")

with open('/tmp/lines.txt') as f:
    for line in f:
        print(repr(line))
```
**Explanation:** Iterating a file object is memory-efficient (line-by-line lazy read) vs. `readlines()` which loads everything into a list at once — critical for large datasets/log files in ML preprocessing.
**Follow-up:** How would you process a 50GB CSV without loading it all into memory?

### Q34. File Pointer Position and `seek()` 🟡
```python
with open('/tmp/test.txt', 'w') as f:
    f.write("0123456789")

with open('/tmp/test.txt', 'r') as f:
    print(f.read(3))
    f.seek(0)
    print(f.read(3))
    f.seek(5)
    print(f.read())
```
**Output:** `012`, `012`, `56789`
**Follow-up:** Difference between `seek()` behavior in text mode vs binary mode (text mode restricts arbitrary offsets in some encodings).

### Q35. Appending vs Overwriting 🟢
```python
with open('/tmp/log.txt', 'w') as f:
    f.write("first\n")
with open('/tmp/log.txt', 'a') as f:
    f.write("second\n")
with open('/tmp/log.txt', 'r') as f:
    print(f.read())
```
**Output:** `first\nsecond\n`
**Follow-up:** What happens if two processes append to the same file concurrently (race conditions, need for file locking).

### Q36. FileNotFoundError Handling 🟢
```python
try:
    with open('/tmp/does_not_exist.txt') as f:
        data = f.read()
except FileNotFoundError as e:
    print(f"Error: {e}")
finally:
    print("Attempted read")
```
**Follow-up:** Why catching broad `Exception` instead of `FileNotFoundError` is bad practice.

### Q37. Writing and Reading JSON Files 🟡
```python
import json

data = {"model": "resnet50", "accuracy": 0.94}
with open('/tmp/config.json', 'w') as f:
    json.dump(data, f)

with open('/tmp/config.json') as f:
    loaded = json.load(f)
print(loaded == data, type(loaded))
```
**Follow-up:** How `json.dump` handles non-serializable objects like NumPy arrays (raises `TypeError`; need custom encoder).

### Q38. CSV Handling with `csv` Module 🟡
```python
import csv

rows = [["name", "score"], ["A", 90], ["B", 85]]
with open('/tmp/scores.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerows(rows)

with open('/tmp/scores.csv') as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row)
```
**Explanation:** `newline=''` prevents extra blank lines on Windows due to universal newline translation.
**Follow-up:** Why use `csv` module instead of manual `split(',')` (handles quoting, embedded commas, escaping).

### Q39. Binary File Corruption via Wrong Mode 🔴
```python
data = bytes([0, 159, 146, 150])
with open('/tmp/binary.dat', 'wb') as f:
    f.write(data)

with open('/tmp/binary.dat', 'r') as f:
    try:
        content = f.read()
    except UnicodeDecodeError as e:
        print("Error:", e)
```
**Explanation:** Opening a binary file (e.g., a serialized model checkpoint) in text mode attempts to decode arbitrary bytes as text and fails.
**Follow-up:** Real-world parallel: loading a `.pkl` or `.pt` model file — must always use `'rb'`.

### Q40. Using `pathlib` vs `os.path` 🟢
```python
from pathlib import Path

p = Path('/tmp/models/checkpoint.pt')
print(p.parent, p.name, p.suffix, p.stem)
p.parent.mkdir(parents=True, exist_ok=True)
```
**Follow-up:** Why `pathlib` is considered more Pythonic/readable than string-based `os.path` manipulation.

### Q41. Exception During Write Leaves Partial File 🔴
```python
try:
    with open('/tmp/partial.txt', 'w') as f:
        f.write("Line1\n")
        raise ValueError("Simulated crash")
        f.write("Line2\n")
except ValueError:
    pass

with open('/tmp/partial.txt') as f:
    print(repr(f.read()))
```
**Output:** `'Line1\n'`
**Explanation:** The `with` block still closes/flushes the file on exception, but code after the raised exception never executes — a partial write can occur.
**Follow-up:** How to make file writes atomic (write to temp file, then `os.replace()`).

### Q42. Buffering Behavior 🔴
```python
import time

f = open('/tmp/buffered.txt', 'w')
f.write("Not yet visible on disk reliably")
# f.flush()  # uncomment to force write
time.sleep(0.1)
f.close()
```
**Follow-up:** Why `flush()` and OS-level buffering matter for logging systems that must survive crashes.

### Q43. Reading Large Files in Chunks 🟡
```python
def read_in_chunks(file_path, chunk_size=4):
    with open(file_path, 'r') as f:
        while chunk := f.read(chunk_size):
            yield chunk

with open('/tmp/chunked.txt', 'w') as f:
    f.write("abcdefghij")

for c in read_in_chunks('/tmp/chunked.txt'):
    print(c)
```
**Explanation:** Uses the walrus operator `:=` for compact loop-and-assign; generator avoids loading the whole file into memory (important for large ML datasets).
**Follow-up:** How would you adapt this for streaming tokenization of a large text corpus?

### Q44. Multiple Context Managers 🟡
```python
with open('/tmp/src.txt', 'w') as f:
    f.write("data")

with open('/tmp/src.txt') as src, open('/tmp/dst.txt', 'w') as dst:
    dst.write(src.read())

with open('/tmp/dst.txt') as f:
    print(f.read())
```
**Follow-up:** Why chaining context managers in one `with` is preferable to nested separate `with` blocks.

### Q45. Handling Encoding Errors Gracefully 🟡
```python
with open('/tmp/bad_enc.txt', 'wb') as f:
    f.write(b'\xff\xfe Invalid UTF-8 bytes')

with open('/tmp/bad_enc.txt', encoding='utf-8', errors='replace') as f:
    print(f.read())
```
**Explanation:** `errors='replace'` substitutes undecodable bytes with `�` instead of raising, useful for noisy real-world text data.
**Follow-up:** When would `errors='ignore'` be dangerous for ML text pipelines (silent data loss)?

### Q46. Temporary Files 🟡
```python
import tempfile

with tempfile.NamedTemporaryFile(mode='w+', delete=True) as tmp:
    tmp.write("scratch data")
    tmp.seek(0)
    print(tmp.read())
print("File deleted:", not tmp.name)
```
**Follow-up:** Use cases for `tempfile` in caching intermediate model artifacts during a pipeline run.

### Q47. File Locking Concept (Simulated) 🔴
```python
import os

lock_path = '/tmp/train.lock'
if os.path.exists(lock_path):
    print("Another training job may be running")
else:
    with open(lock_path, 'w') as f:
        f.write(str(os.getpid()))
    print("Lock acquired")
    os.remove(lock_path)
```
**Follow-up:** Why this naive check has a race condition (TOCTOU); mention `fcntl.flock` or atomic `os.open(O_CREAT | O_EXCL)`.

### Q48. Pickle Security Risk 🔴
```python
import pickle

data = {"weights": [0.1, 0.2, 0.3]}
with open('/tmp/model.pkl', 'wb') as f:
    pickle.dump(data, f)

with open('/tmp/model.pkl', 'rb') as f:
    loaded = pickle.load(f)
print(loaded)
```
**Explanation:** `pickle.load` can execute arbitrary code if the file is untrusted/tampered — a real ML security concern (never unpickle models from untrusted sources).
**Follow-up:** Safer alternatives: `safetensors`, `joblib` with checksum validation, or JSON for simple structures.

### Q49. Exception Hierarchy: `OSError` Family 🟡
```python
paths = ['/root/protected.txt', '/tmp/nonexistent_dir/file.txt']
for p in paths:
    try:
        open(p, 'r')
    except PermissionError:
        print(f"{p}: Permission denied")
    except FileNotFoundError:
        print(f"{p}: Not found")
    except OSError as e:
        print(f"{p}: Other OS error - {e}")
```
**Follow-up:** Why catching specific exceptions before the general `OSError` matters (order of `except` clauses).

### Q50. Writing Line-by-Line with Generator + File Handling Combined 🔴
```python
def generate_predictions():
    for i in range(5):
        yield f"sample_{i},{i*0.1:.2f}\n"

with open('/tmp/predictions.csv', 'w') as f:
    f.writelines(generate_predictions())

with open('/tmp/predictions.csv') as f:
    print(f.read())
```
**Follow-up:** Why `writelines(generator)` is more memory-efficient than building a full list first when exporting large batch-inference outputs.

---

# PART 3 — EXCEPTION HANDLING (Q51–Q70)

### Q51. Basic try/except/else/finally Flow 🟢
```python
try:
    x = 10 / 2
except ZeroDivisionError:
    print("A")
else:
    print("B")
finally:
    print("C")
```
**Output:** `B` then `C`
**Explanation:** `else` runs only if no exception occurred; `finally` always runs.
**Follow-up:** Why put code in `else` instead of just after the `try` block.

### Q52. Finally Overriding Return 🔴
```python
def f():
    try:
        return 1
    finally:
        return 2

print(f())
```
**Output:** `2`
**Explanation:** A `return` in `finally` overrides any pending `return`/exception from the `try` block — a classic gotcha.
**Follow-up:** Why this is considered bad practice / anti-pattern in production code.

### Q53. Exception Swallowed Silently 🔴
```python
def divide(a, b):
    try:
        return a / b
    except Exception:
        pass

print(divide(10, 0))
print(divide(10, 2))
```
**Output:** `None` then `5.0`
**Explanation:** Bare `except: pass` silently swallows errors, returning `None` — a dangerous anti-pattern that hides bugs, especially in ML training loops.
**Follow-up:** Why "fail loudly" is generally safer than defensive silent failure in data pipelines.

### Q54. Custom Exception Classes 🟡
```python
class InvalidModelConfigError(Exception):
    def __init__(self, message, config):
        super().__init__(message)
        self.config = config

try:
    raise InvalidModelConfigError("Bad learning rate", {"lr": -1})
except InvalidModelConfigError as e:
    print(e, e.config)
```
**Follow-up:** Why domain-specific exceptions improve debuggability in large ML codebases vs generic `ValueError`.

### Q55. Exception Chaining with `raise ... from` 🔴
```python
def load_config():
    try:
        1 / 0
    except ZeroDivisionError as e:
        raise RuntimeError("Config load failed") from e

try:
    load_config()
except RuntimeError as e:
    print(e)
    print(e.__cause__)
```
**Explanation:** `raise ... from e` preserves the original traceback chain (`__cause__`) for better debugging instead of masking the root cause.
**Follow-up:** Difference between `__cause__` (explicit chaining) and `__context__` (implicit, automatic).

### Q56. Multiple Except Blocks Order Matters 🟡
```python
try:
    raise ValueError("bad value")
except Exception:
    print("Generic handler")
except ValueError:
    print("Specific handler")
```
**Output:** `Generic handler`
**Explanation:** Python checks `except` clauses top-to-bottom; since `Exception` is broader and comes first, `ValueError` never gets its own handler — this actually raises no error but is a logic bug (unreachable code).
**Follow-up:** Why the more specific exception should always be listed first.

### Q57. Catching Multiple Exception Types 🟢
```python
def parse(value):
    try:
        return int(value)
    except (ValueError, TypeError) as e:
        return f"Error: {type(e).__name__}"

print(parse("abc"))
print(parse(None))
print(parse("42"))
```
**Follow-up:** When to use a tuple of exceptions vs separate `except` blocks (different handling logic needed → separate blocks).

### Q58. Re-raising Exceptions 🟡
```python
def risky():
    try:
        1 / 0
    except ZeroDivisionError:
        print("Logging error...")
        raise

try:
    risky()
except ZeroDivisionError:
    print("Caught at top level")
```
**Explanation:** Bare `raise` inside an `except` block re-raises the *same* exception with its original traceback intact.
**Follow-up:** Difference between `raise` and `raise e` (the latter resets the traceback start point).

### Q59. Exception in a Generator 🔴
```python
def gen():
    try:
        yield 1
        yield 2
    except GeneratorExit:
        print("Generator closed")
        raise

g = gen()
print(next(g))
g.close()
```
**Follow-up:** Why `GeneratorExit` should generally not be swallowed (breaking cleanup contracts).

### Q60. Nested try/except with Finally Order 🔴
```python
def f():
    try:
        try:
            raise ValueError("inner")
        finally:
            print("inner finally")
    except ValueError as e:
        print(f"caught: {e}")
    finally:
        print("outer finally")

f()
```
**Output:** `inner finally`, `caught: inner`, `outer finally`
**Follow-up:** Why `finally` blocks always execute regardless of nesting depth, even when an exception propagates.

### Q61. Assertion Errors in Production 🟡
```python
def validate_shape(tensor_shape, expected):
    assert tensor_shape == expected, f"Shape mismatch: {tensor_shape} != {expected}"
    return True

try:
    validate_shape((32, 10), (32, 5))
except AssertionError as e:
    print(e)
```
**Follow-up:** Why `assert` statements are stripped out when Python runs with `-O` (optimized mode) — never use asserts for critical runtime validation like input sanitization.

### Q62. Exception Group / `except*` (Python 3.11+) 🔴
```python
try:
    raise ExceptionGroup("multi", [ValueError("v"), TypeError("t")])
except* ValueError as eg:
    print("Caught ValueErrors:", eg.exceptions)
except* TypeError as eg:
    print("Caught TypeErrors:", eg.exceptions)
```
**Follow-up:** Use case: running parallel validation checks on a batch and collecting all failures instead of stopping at the first.

### Q63. Context Manager Exception Suppression 🔴
```python
class Suppressor:
    def __enter__(self): return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is ValueError:
            print("Suppressing ValueError")
            return True
        return False

with Suppressor():
    raise ValueError("oops")
print("Continued execution")
```
**Explanation:** Returning `True` from `__exit__` suppresses the exception, allowing code after the `with` block to run.
**Follow-up:** Compare to `contextlib.suppress(ValueError)`.

### Q64. Cleanup Guarantees with `finally` + `sys.exit` 🔴
```python
import sys

def f():
    try:
        sys.exit(1)
    finally:
        print("Cleanup runs even on sys.exit")

try:
    f()
except SystemExit:
    print("SystemExit caught")
```
**Explanation:** `sys.exit()` raises `SystemExit`, which is a `BaseException`, not `Exception` — `finally` still executes, and it can be caught.
**Follow-up:** Why `except Exception` alone would NOT catch `SystemExit` or `KeyboardInterrupt`.

### Q65. Custom Exception Hierarchies for ML Pipelines 🟡
```python
class PipelineError(Exception): pass
class DataValidationError(PipelineError): pass
class ModelTrainingError(PipelineError): pass

def run_stage(stage):
    if stage == "data":
        raise DataValidationError("Missing labels column")
    raise ModelTrainingError("NaN loss detected")

for stage in ["data", "train"]:
    try:
        run_stage(stage)
    except PipelineError as e:
        print(f"[{type(e).__name__}] {e}")
```
**Follow-up:** Why a shared base exception class lets calling code catch broadly while still allowing fine-grained handling.

### Q66. Exception Object Reference After Block (Python 3) 🔴
```python
try:
    raise ValueError("test")
except ValueError as e:
    error = e
print(error)
```
**Explanation:** In Python 3, `e` is deleted automatically at the end of the `except` block (to avoid reference cycles), but assigning it to `error` beforehand keeps it accessible.
**Follow-up:** What happens if you try to `print(e)` directly after the block (raises `NameError`).

### Q67. Retrying with Exponential Backoff 🟡
```python
import time

def unreliable_call(attempt_counter=[0]):
    attempt_counter[0] += 1
    if attempt_counter[0] < 3:
        raise ConnectionError("Service unavailable")
    return "Success"

def retry(func, max_attempts=5):
    for attempt in range(1, max_attempts + 1):
        try:
            return func()
        except ConnectionError as e:
            print(f"Attempt {attempt} failed: {e}")
            time.sleep(0.01 * attempt)
    raise RuntimeError("Max retries exceeded")

print(retry(unreliable_call))
```
**Follow-up:** Real ML use case: retrying flaky API calls to a model-serving endpoint or cloud storage.

### Q68. `else` Clause Skipped on Exception 🟢
```python
def f(x):
    try:
        result = 10 / x
    except ZeroDivisionError:
        print("Division error")
        return None
    else:
        print("No error occurred")
        return result

print(f(0))
print(f(5))
```
**Follow-up:** Why separating success-path logic into `else` avoids accidentally catching exceptions raised by that logic itself.

### Q69. Warnings vs Exceptions 🟡
```python
import warnings

def train(learning_rate):
    if learning_rate > 1.0:
        warnings.warn("Learning rate unusually high", UserWarning)
    return learning_rate

with warnings.catch_warnings(record=True) as w:
    warnings.simplefilter("always")
    train(5.0)
    print(len(w), w[0].category)
```
**Follow-up:** When to use `warnings.warn` (non-fatal, recoverable issues) vs raising an exception (fatal, must stop execution).

### Q70. Traceback Inspection 🔴
```python
import traceback

def a(): b()
def b(): c()
def c(): raise ValueError("deep error")

try:
    a()
except ValueError:
    tb_lines = traceback.format_exc()
    print("Number of 'File' references:", tb_lines.count("File"))
```
**Follow-up:** Why full traceback logging (not just `str(e)`) is essential for debugging failures in distributed training jobs.

---

# PART 4 — PYTHON TRICKY QUESTIONS (Q71–Q100)

### Q71. Mutable Default Argument 🔴
```python
def append_to(element, target=[]):
    target.append(element)
    return target

print(append_to(1))
print(append_to(2))
```
**Output:** `[1]` then `[1, 2]`
**Explanation:** Default arguments are evaluated once at function definition time; the same list persists across calls.
**Fix:** Use `target=None` and initialize inside the function.

### Q72. Late Binding Closures in Loops 🔴
```python
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])
```
**Output:** `[2, 2, 2]`
**Explanation:** Closures capture the variable `i`, not its value at creation time; by the time the lambdas execute, the loop has finished and `i == 2`.
**Fix:** `lambda i=i: i` to bind the current value as a default argument.

### Q73. Small Integer Caching (`is` vs `==`) 🔴
```python
a = 256
b = 256
print(a is b)

c = 257
d = 257
print(c is d)
```
**Output:** `True` then `False` (in standard CPython)
**Explanation:** CPython caches small integers (-5 to 256); values outside this range may or may not be the same object depending on context.
**Follow-up:** Why `is` should never be used for value comparison — only for identity/singleton checks (`is None`).

### Q74. String Interning 🟡
```python
a = "hello"
b = "hello"
print(a is b)

c = "hello world!"
d = "hello world!"
print(c is d)
```
**Explanation:** Simple identifier-like string literals are interned automatically; strings with spaces/constructed at runtime may not be.
**Follow-up:** How `sys.intern()` can be used explicitly for memory optimization with repeated string keys (e.g., categorical labels).

### Q75. Chained Comparisons 🟡
```python
x = 5
print(1 < x < 10)
print(1 < x and x < 10)
print(10 < x < 1)
```
**Explanation:** `1 < x < 10` is syntactic sugar equivalent to `1 < x and x < 10`, evaluated with short-circuiting.
**Follow-up:** Trick question: `print(False == 0 < 1)` — evaluates as `False == 0 and 0 < 1` → `True`.

### Q76. `is` vs `==` for Custom Objects 🟡
```python
class Box:
    def __init__(self, val): self.val = val

b1 = Box(5)
b2 = Box(5)
print(b1 == b2)
print(b1.val == b2.val)
```
**Output:** `False` then `True`
**Explanation:** Without a custom `__eq__`, `==` falls back to identity comparison (same as `is`).

### Q77. `*args` and `**kwargs` Unpacking Order 🟡
```python
def f(a, b, *args, c=10, **kwargs):
    print(a, b, args, c, kwargs)

f(1, 2, 3, 4, c=5, d=6)
```
**Output:** `1 2 (3, 4) 5 {'d': 6}`
**Follow-up:** Why keyword-only arguments (after `*args`) can't be passed positionally.

### Q78. Shallow Copy List Slicing Trick 🟡
```python
original = [1, [2, 3], 4]
copy_list = original[:]
copy_list[1].append(99)
print(original)
```
**Output:** `[1, [2, 3, 99], 4]`
**Explanation:** Slicing creates a shallow copy — nested mutable objects are still shared.

### Q79. `+=` on Lists vs Tuples in Objects 🔴
```python
class Container:
    def __init__(self):
        self.data = (1, [2, 3])

c = Container()
try:
    c.data[1] += [4]
except TypeError as e:
    print("Error:", e)
print(c.data)
```
**Output:** `Error: 'tuple' object does not support item assignment` then `(1, [2, 3, 4])`
**Explanation:** `+=` on a list *inside* a tuple works (mutates the list in place) but then fails to rebind the tuple slot — the mutation still happens before the error is raised, which surprises most candidates.

### Q80. `and`/`or` Return Values, Not Booleans 🟡
```python
print(0 or "default")
print("value" and "another")
print([] or {})
print(None and "unreachable")
```
**Explanation:** `and`/`or` short-circuit and return the actual operand value, not a strict `True`/`False`.
**Follow-up:** Common use in default-value idioms: `x = user_input or "fallback"`.

### Q81. Global vs Local Scope with `global` Keyword 🟡
```python
counter = 0
def increment():
    counter += 1
    return counter

try:
    increment()
except UnboundLocalError as e:
    print("Error:", e)
```
**Explanation:** Assigning to `counter` anywhere in the function makes it local for the *entire* function scope, so reading it before assignment fails.
**Fix:** Add `global counter` inside the function.

### Q82. try/except/finally with Return Value Flow 🔴
```python
def f():
    try:
        return 10 / 0
    except Exception:
        print("A")
    finally:
        print("B")
    print("C")

print(f())
```
**Output:** `A`, `B`, `None`
**Explanation:** The exception is caught (no propagation); the `except` block has no explicit `return`, so the function implicitly returns `None`; line `print("C")` never runs because `except` already exited via the function's normal fall-through after `finally`... actually since no return occurs in `except`, control proceeds past `finally` to `print("C")` then implicit `return None`.
**Corrected Output:** `A`, `B`, `C`, then `None` is printed.

### Q83. Nested Function Scope (LEGB Rule) 🟡
```python
x = "global"
def outer():
    x = "enclosing"
    def inner():
        nonlocal x
        x = "modified"
    inner()
    print(x)

outer()
print(x)
```
**Output:** `modified` then `global`
**Follow-up:** LEGB = Local, Enclosing, Global, Built-in resolution order.

### Q84. Boolean is a Subclass of Int 🟢
```python
print(True + True)
print(True == 1)
print(isinstance(True, int))
print([True, False, True].count(True))
```
**Explanation:** `bool` is a subclass of `int`; `True == 1` and `False == 0`.

### Q85. Default Mutable Argument in Recursive Functions 🔴
```python
def collect(n, acc=None):
    if acc is None:
        acc = []
    acc.append(n)
    if n > 0:
        collect(n - 1, acc)
    return acc

print(collect(3))
```
**Explanation:** Correct pattern using `None` sentinel avoids the mutable-default trap while still supporting accumulation across recursive calls.

### Q86. `id()` and Object Reuse 🟡
```python
a = [1, 2, 3]
b = a
b.append(4)
print(a, id(a) == id(b))

c = a.copy()
print(id(a) == id(c))
```
**Follow-up:** Difference between reference assignment (`b = a`) and copying (`c = a.copy()`).

### Q87. `zip()` Exhausting Iterators 🟡
```python
a = iter([1, 2, 3])
b = [10, 20, 30]
zipped = list(zip(a, b))
print(zipped)
print(list(a))
```
**Output:** `[(1, 10), (2, 20), (3, 30)]` then `[]`
**Explanation:** `zip` consumes the iterator `a` fully; once exhausted, further iteration yields nothing.

### Q88. String Immutability and Memory 🟢
```python
s = "hello"
s_id = id(s)
s += " world"
print(id(s) == s_id)
```
**Output:** `False`
**Explanation:** Strings are immutable; `+=` creates a new string object rather than mutating in place.

### Q89. `sorted()` with `key` and Lambda Trick 🟡
```python
data = [("apple", 3), ("banana", 1), ("cherry", 2)]
print(sorted(data, key=lambda x: x[1]))
print(sorted(data, key=lambda x: -x[1]))
print(sorted(data, key=lambda x: x[1], reverse=True))
```
**Follow-up:** Why `key=lambda x: -x[1]` and `reverse=True` don't always give identical results for non-numeric or tie-breaking cases (stability differs).

### Q90. Integer Division and Floor Behavior with Negatives 🟡
```python
print(7 // 2)
print(-7 // 2)
print(7 % -2)
print(-7 % 2)
```
**Output:** `3`, `-4`, `-1`, `1`
**Explanation:** Python's `//` always floors toward negative infinity (unlike C's truncation toward zero), and `%` result takes the sign of the divisor.

### Q91. `is` Comparison Pitfall with Tuples 🔴
```python
t1 = (1, 2, 3)
t2 = (1, 2, 3)
print(t1 is t2)
print(t1 == t2)

t3 = tuple([1, 2, 3])
print(t1 is t3)
```
**Follow-up:** CPython may or may not intern small immutable tuple literals depending on context; never rely on this behavior.

### Q92. Dictionary Ordering Guarantee (3.7+) 🟢
```python
d = {}
d['z'] = 1
d['a'] = 2
d['m'] = 3
print(list(d.keys()))
```
**Output:** `['z', 'a', 'm']`
**Explanation:** Since Python 3.7, dicts preserve insertion order as a language guarantee (not just a CPython implementation detail).

### Q93. Unpacking with Starred Expressions 🟡
```python
first, *middle, last = [1, 2, 3, 4, 5]
print(first, middle, last)

a, b, *rest = [1]
print(a, b, rest)
```
**Output:** first line fine; second line raises `ValueError: not enough values to unpack`.

### Q94. `all()` and `any()` on Empty Iterables 🟡
```python
print(all([]))
print(any([]))
print(all([True, True, []]))
```
**Output:** `True`, `False`, `False`
**Explanation:** `all([])` is vacuously `True`; `all` short-circuits on the first falsy element (`[]` is falsy).

### Q95. Function Argument Evaluation Order 🟡
```python
def f(a, b, c):
    print(a, b, c)

def get_value(x):
    print(f"Evaluating {x}")
    return x

f(get_value(1), get_value(2), get_value(3))
```
**Explanation:** Arguments are evaluated left-to-right before the function call itself executes.

### Q96. `__slots__` Inheritance Trap 🔴
```python
class A:
    __slots__ = ('x',)

class B(A):
    pass

b = B()
b.x = 1
b.y = 2
print(b.x, b.y)
```
**Explanation:** Since `B` doesn't declare `__slots__`, it gets a `__dict__` automatically, allowing arbitrary attributes — defeating the memory-saving purpose of `__slots__` in the parent.

### Q97. `enumerate()` with Custom Start 🟢
```python
for i, val in enumerate(['a', 'b', 'c'], start=1):
    print(i, val)
```
**Follow-up:** Common in ML for 1-indexed epoch/batch logging.

### Q98. Truthiness of Custom Objects 🟡
```python
class AlwaysTrue:
    pass

class NeverTrue:
    def __bool__(self):
        return False

print(bool(AlwaysTrue()))
print(bool(NeverTrue()))
```
**Follow-up:** Without `__bool__` or `__len__`, all custom objects are truthy by default.

### Q99. Floating Point Precision Trap 🟡
```python
print(0.1 + 0.2 == 0.3)
print(round(0.1 + 0.2, 10) == round(0.3, 10))
import math
print(math.isclose(0.1 + 0.2, 0.3))
```
**Explanation:** Classic IEEE-754 floating point representation error; critical for ML loss/metric comparisons — always use tolerance-based comparison (`math.isclose`, `np.allclose`).

### Q100. Walrus Operator in Comprehensions 🟡
```python
data = [1, 2, 3, 4, 5, 6]
result = [y for x in data if (y := x ** 2) > 10]
print(result)
```
**Output:** `[16, 25, 36]`
**Explanation:** `:=` (walrus) assigns and evaluates in the same expression, avoiding redundant computation inside comprehensions.

---

# PART 5 — NESTED LISTS & DATA STRUCTURES (Q101–Q120)

### Q101. Shared Reference via List Multiplication 🔴
```python
matrix = [[0] * 3] * 3
matrix[0][0] = 1
print(matrix)
```
**Output:** `[[1, 0, 0], [1, 0, 0], [1, 0, 0]]`
**Explanation:** `[[0]*3]*3` creates three references to the *same* inner list, not three independent lists.
**Fix:** `[[0]*3 for _ in range(3)]`

### Q102. Correct Nested List Initialization 🟢
```python
matrix = [[0] * 3 for _ in range(3)]
matrix[0][0] = 1
print(matrix)
```
**Output:** `[[1, 0, 0], [0, 0, 0], [0, 0, 0]]`
**Explanation:** List comprehension creates a new inner list on each iteration.

### Q103. Flattening a Nested List Recursively 🟡
```python
def flatten(lst):
    result = []
    for item in lst:
        if isinstance(item, list):
            result.extend(flatten(item))
        else:
            result.append(item)
    return result

print(flatten([1, [2, [3, 4], 5], [6, [7, [8, 9]]]]))
```
**Follow-up:** How to handle arbitrary depth without recursion limit issues (iterative approach with a stack).

### Q104. List Comprehension with Nested Loops 🟡
```python
matrix = [[1, 2, 3], [4, 5, 6]]
flat = [x for row in matrix for x in row]
print(flat)

transposed = [[row[i] for row in matrix] for i in range(3)]
print(transposed)
```
**Follow-up:** Why comprehension order mirrors nested `for` loop order (outer loop first).

### Q105. Sorting Nested Lists by Multiple Keys 🟡
```python
students = [["Alice", 85, 22], ["Bob", 85, 20], ["Carol", 90, 25]]
students.sort(key=lambda s: (-s[1], s[2]))
print(students)
```
**Explanation:** Sorts by score descending, then age ascending, as tie-breaker — using tuple keys for multi-criteria sort.

### Q106. Deep Equality vs Shallow Identity in Nested Structures 🟡
```python
a = [[1, 2], [3, 4]]
b = [[1, 2], [3, 4]]
print(a == b)
print(a is b)
print(a[0] is b[0])
```
**Output:** `True`, `False`, `False`
**Explanation:** `==` recursively compares values; `is` checks object identity at every level.

### Q107. Mutating a List While Iterating 🔴
```python
nums = [1, 2, 3, 4, 5]
for n in nums:
    if n % 2 == 0:
        nums.remove(n)
print(nums)
```
**Output:** `[1, 3, 5]` — but this "accidentally" works only because of index shifting; it's still a well-known anti-pattern.
**Explanation:** Removing elements while iterating shifts subsequent indices, potentially skipping elements — fragile and considered a bug pattern even when the output looks correct for this specific input.
**Fix:** Iterate over a copy `nums[:]` or use a list comprehension to build a new filtered list.

### Q108. Dictionary of Lists — Default Values Trap 🔴
```python
from collections import defaultdict

groups = defaultdict(list)
data = [("a", 1), ("b", 2), ("a", 3)]
for key, val in data:
    groups[key].append(val)
print(dict(groups))

d = {}
try:
    d["missing"].append(1)
except KeyError as e:
    print("Error:", e)
```
**Explanation:** `defaultdict` auto-creates missing keys with the factory (`list`), while a plain `dict` raises `KeyError`.

### Q109. Nested Dictionary Merge Trap 🔴
```python
def merge(d1, d2):
    result = d1.copy()
    result.update(d2)
    return result

config1 = {"model": {"lr": 0.01, "layers": 2}}
config2 = {"model": {"lr": 0.001}}
merged = merge(config1, config2)
print(merged)
```
**Output:** `{'model': {'lr': 0.001}}`
**Explanation:** Shallow `.update()` replaces the entire nested dict rather than merging keys inside it — `layers` is lost.
**Fix:** Recursive/deep merge logic needed for nested config dictionaries (common in ML hyperparameter configs).

### Q110. List of Dicts — Sorting and Grouping 🟡
```python
from itertools import groupby

records = [{"dept": "AI", "name": "A"}, {"dept": "AI", "name": "B"}, {"dept": "ML", "name": "C"}]
records.sort(key=lambda r: r["dept"])
for dept, group in groupby(records, key=lambda r: r["dept"]):
    print(dept, [r["name"] for r in group])
```
**Follow-up:** Why `groupby` requires the data to be pre-sorted by the grouping key to work correctly.

### Q111. Tuple Immutability with Nested Mutable Elements 🟡
```python
t = ([1, 2], [3, 4])
t[0].append(3)
print(t)
try:
    t[0] = [9, 9]
except TypeError as e:
    print("Error:", e)
```
**Explanation:** Tuples are immutable at the container level (can't reassign slots) but nested mutable objects inside remain mutable.

### Q112. Set Operations on Nested-Derived Data 🟡
```python
list_a = [1, 2, 2, 3, 4]
list_b = [3, 4, 4, 5]
print(set(list_a) & set(list_b))
print(set(list_a) | set(list_b))
print(set(list_a) - set(list_b))
```
**Follow-up:** Why sets can't contain unhashable items like lists — relevant when deduplicating nested/complex ML feature records.

### Q113. Matrix Transpose with `zip(*matrix)` 🟡
```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
transposed = list(zip(*matrix))
print(transposed)
print([list(row) for row in transposed])
```
**Explanation:** `zip(*matrix)` unpacks rows as separate arguments and zips them column-wise — elegant transpose without NumPy.

### Q114. Recursive Sum of Nested Lists 🟡
```python
def deep_sum(lst):
    total = 0
    for item in lst:
        if isinstance(item, list):
            total += deep_sum(item)
        else:
            total += item
    return total

print(deep_sum([1, [2, 3, [4, 5]], 6, [7, [8, [9]]]]))
```
**Output:** `45`

### Q115. List Slicing Edge Cases 🟡
```python
lst = [1, 2, 3, 4, 5]
print(lst[10:])
print(lst[-100:2])
print(lst[::-1])
print(lst[1:100:2])
```
**Explanation:** Python slicing never raises `IndexError`, silently clamps out-of-range bounds — very different from direct indexing.

### Q116. Nested List Comprehension with Conditionals 🟡
```python
matrix = [[1, -2, 3], [-4, 5, -6], [7, -8, 9]]
positives_only = [[x for x in row if x > 0] for row in matrix]
print(positives_only)

flat_positive_sum = sum(x for row in matrix for x in row if x > 0)
print(flat_positive_sum)
```

### Q117. `copy.deepcopy` vs Manual Nested Copy Performance 🔴
```python
import copy
import time

nested = [[i for i in range(100)] for _ in range(1000)]

start = time.perf_counter()
d1 = copy.deepcopy(nested)
t1 = time.perf_counter() - start

start = time.perf_counter()
d2 = [row[:] for row in nested]
t2 = time.perf_counter() - start

print(f"deepcopy faster: {t1 < t2}")
```
**Follow-up:** Why `deepcopy` is generally slower for simple 2-level nesting due to its generality (handles cycles, arbitrary objects) — manual comprehension-based copying is often faster for known-shallow structures like plain numeric matrices.

### Q118. Heterogeneous Nested Structures (JSON-like) 🟡
```python
data = {
    "model": "cnn",
    "layers": [{"type": "conv", "filters": 32}, {"type": "pool", "size": 2}],
    "metrics": {"accuracy": 0.95, "loss": [0.5, 0.3, 0.1]}
}
print(data["layers"][0]["filters"])
print(data["metrics"]["loss"][-1])
print(sum(l["filters"] for l in data["layers"] if l["type"] == "conv"))
```
**Follow-up:** Real-world parallel: parsing model architecture configs or API responses (nested JSON) — common in MLOps tooling.

### Q119. Circular Reference in Nested List 🔴
```python
a = [1, 2, 3]
a.append(a)
print(a[3][3][0])
print(a is a[3])
```
**Explanation:** Self-referential lists are legal in Python; `print(a)` directly would show `[1, 2, 3, [...]]` to avoid infinite recursion.
**Follow-up:** Why `json.dumps(a)` would raise `ValueError: Circular reference detected`.

### Q120. Sparse Matrix Representation Trade-offs 🔴
```python
dense = [[0]*1000 for _ in range(1000)]
dense[5][10] = 1
dense[500][999] = 2

sparse = {(5, 10): 1, (500, 999): 2}

print(sparse.get((5, 10), 0))
print(sparse.get((0, 0), 0))
```
**Follow-up:** When to prefer sparse dict-of-coordinates representation over dense nested lists for large, mostly-zero ML feature matrices (memory: O(nonzeros) vs O(n²)).

---

# PART 6 — FUNCTIONS & SCOPE (Q121–Q130)

### Q121. Default Argument Evaluated Once — Function Object 🟡
```python
def counter(count=[0]):
    count[0] += 1
    return count[0]

print(counter())
print(counter())
print(counter())
```
**Output:** `1 2 3`
**Explanation:** Same mutable-default trap as Q71, here intentionally exploited as a (fragile) memoized counter.

### Q122. First-Class Functions Passed as Arguments 🟢
```python
def apply_twice(func, x):
    return func(func(x))

print(apply_twice(lambda x: x * 2, 3))
```
**Output:** `12`

### Q123. `functools.partial` 🟡
```python
from functools import partial

def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)
print(square(4), cube(2))
```
**Follow-up:** Real ML use: pre-configuring a loss function with fixed hyperparameters before passing to a training loop.

### Q124. Variable Scope in List Comprehensions (Py3) 🔴
```python
x = 10
result = [x for x in range(5)]
print(x)
print(result)
```
**Output:** `10` then `[0, 1, 2, 3, 4]`
**Explanation:** Unlike Python 2, list comprehensions have their own scope in Python 3 — the loop variable `x` doesn't leak out.

### Q125. Function Annotations Are Not Enforced 🟡
```python
def add(a: int, b: int) -> int:
    return a + b

print(add("hello", "world"))
print(add.__annotations__)
```
**Output:** `helloworld` then the annotations dict
**Explanation:** Type hints are purely advisory at runtime (unless enforced by tools like `mypy` or `pydantic`); Python doesn't type-check automatically.

### Q126. Keyword-Only Arguments 🟡
```python
def train(model, *, epochs=10, lr=0.01):
    print(model, epochs, lr)

train("resnet", epochs=5)
try:
    train("resnet", 5)
except TypeError as e:
    print("Error:", e)
```
**Explanation:** Arguments after `*` must be passed by keyword only.

### Q127. Recursive Function with Memoization 🟡
```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)

print(fib(30))
print(fib.cache_info())
```
**Follow-up:** Why memoization drastically improves recursive Fibonacci from O(2^n) to O(n); caveats with mutable/unhashable arguments.

### Q128. Variable Number of Return Values / Tuple Unpacking 🟢
```python
def stats(data):
    return min(data), max(data), sum(data)/len(data)

lo, hi, avg = stats([1, 2, 3, 4, 5])
print(lo, hi, avg)
```

### Q129. Closures Capturing Mutable State 🔴
```python
def make_accumulator():
    total = 0
    def add(x):
        nonlocal total
        total += x
        return total
    return add

acc = make_accumulator()
print(acc(10))
print(acc(20))

acc2 = make_accumulator()
print(acc2(5))
```
**Explanation:** Each call to `make_accumulator()` creates a fresh closure with its own `total`; `nonlocal` allows the inner function to mutate the enclosing scope's variable.

### Q130. Argument Passing: Pass-by-Object-Reference 🔴
```python
def modify_list(lst):
    lst.append(4)

def reassign_list(lst):
    lst = [100, 200]

my_list = [1, 2, 3]
modify_list(my_list)
print(my_list)

reassign_list(my_list)
print(my_list)
```
**Output:** `[1, 2, 3, 4]` then `[1, 2, 3, 4]` (unchanged by reassignment)
**Explanation:** Python passes object references by value — mutating the object affects the caller, but rebinding the local name inside the function does not.

---

# PART 7 — ITERATORS & GENERATORS (Q131–Q140)

### Q131. Iterator Protocol from Scratch 🟡
```python
class Counter:
    def __init__(self, limit):
        self.limit = limit
        self.current = 0
    def __iter__(self):
        return self
    def __next__(self):
        if self.current >= self.limit:
            raise StopIteration
        self.current += 1
        return self.current

for num in Counter(3):
    print(num)
```
**Follow-up:** Difference between an *iterable* (`__iter__`) and an *iterator* (`__iter__` + `__next__`).

### Q132. Generator Functions and Lazy Evaluation 🟡
```python
def infinite_counter():
    n = 0
    while True:
        yield n
        n += 1

gen = infinite_counter()
print(next(gen), next(gen), next(gen))
```
**Follow-up:** Why generators are memory-efficient for streaming large/infinite ML data sources (e.g., data loaders).

### Q133. Generator Exhaustion 🔴
```python
def gen():
    yield 1
    yield 2

g = gen()
print(list(g))
print(list(g))
```
**Output:** `[1, 2]` then `[]`
**Explanation:** Generators are single-use/one-shot iterators; once exhausted, they cannot be reset or reused.

### Q134. `yield from` Delegation 🟡
```python
def inner():
    yield 1
    yield 2

def outer():
    yield 0
    yield from inner()
    yield 3

print(list(outer()))
```
**Output:** `[0, 1, 2, 3]`

### Q135. Generator Expressions vs List Comprehensions Memory 🟡
```python
import sys

list_comp = [x**2 for x in range(10000)]
gen_exp = (x**2 for x in range(10000))
print(sys.getsizeof(list_comp) > sys.getsizeof(gen_exp))
```
**Output:** `True`
**Explanation:** Generators store only the iteration state, not all computed values — critical for processing huge ML datasets without exhausting RAM.

### Q136. Sending Values into a Generator 🔴
```python
def echo():
    while True:
        received = yield
        print(f"Received: {received}")

gen = echo()
next(gen)
gen.send("hello")
gen.send("world")
```
**Explanation:** `.send()` resumes the generator, injecting a value as the result of the current `yield` expression; `next(gen)` primes it to the first `yield`.

### Q137. Custom Iterable with `__getitem__` Fallback 🔴
```python
class LegacyIterable:
    def __getitem__(self, index):
        if index > 3:
            raise IndexError
        return index * 10

for val in LegacyIterable():
    print(val)
```
**Explanation:** Old-style iteration protocol: if `__iter__` is absent, Python falls back to calling `__getitem__(0), __getitem__(1), ...` until `IndexError`.

### Q138. Infinite Generator with `itertools.islice` 🟡
```python
from itertools import count, islice

gen = count(start=1, step=2)
first_five = list(islice(gen, 5))
print(first_five)
```
**Output:** `[1, 3, 5, 7, 9]`
**Follow-up:** Why `islice` is necessary instead of slicing (`gen[:5]`) which doesn't work on generators.

### Q139. Chaining Multiple Iterables 🟢
```python
from itertools import chain

a = [1, 2, 3]
b = (4, 5)
c = {6, 7}
print(list(chain(a, b, c)))
```
**Follow-up:** Why `chain` avoids materializing a combined intermediate list, unlike `a + list(b) + list(c)`.

### Q140. Generator-based Pipeline (Data Streaming Pattern) 🔴
```python
def read_numbers():
    for i in range(1, 6):
        yield i

def square(gen):
    for n in gen:
        yield n * n

def filter_even(gen):
    for n in gen:
        if n % 2 == 0:
            yield n

pipeline = filter_even(square(read_numbers()))
print(list(pipeline))
```
**Output:** `[4, 16]`
**Follow-up:** This chained-generator pattern mirrors real ETL/ML preprocessing pipelines (read → transform → filter) with constant memory usage regardless of dataset size.

---

# PART 8 — DECORATORS & CLOSURES (Q141–Q145)

### Q141. Basic Function Decorator 🟢
```python
def logger(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with {args}, {kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@logger
def add(a, b):
    return a + b

add(2, 3)
```
**Follow-up:** Why decorators are widely used for logging, timing, and caching in ML training loops.

### Q142. Decorator Losing Metadata (`functools.wraps`) 🟡
```python
from functools import wraps

def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@decorator
def train_model():
    """Trains the model."""
    pass

print(train_model.__name__)
print(train_model.__doc__)
```
**Explanation:** Without `@wraps`, `train_model.__name__` would incorrectly show `'wrapper'`, breaking introspection/documentation tools.

### Q143. Decorators with Arguments (Decorator Factory) 🔴
```python
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(times):
                results.append(func(*args, **kwargs))
            return results
        return wrapper
    return decorator

@repeat(3)
def roll_dice():
    import random
    return random.randint(1, 6)

print(len(roll_dice()))
```
**Explanation:** Three levels of nested functions: the factory takes arguments and returns the actual decorator.

### Q144. Class-based Decorators 🔴
```python
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.count = 0
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"Call #{self.count}")
        return self.func(*args, **kwargs)

@CountCalls
def predict(x):
    return x * 2

predict(1)
predict(2)
print(predict.count)
```
**Follow-up:** Why class-based decorators are useful for maintaining state (e.g., call counters, rate limiters) across invocations.

### Q145. Stacking Multiple Decorators — Execution Order 🔴
```python
def bold(func):
    def wrapper():
        return f"<b>{func()}</b>"
    return wrapper

def italic(func):
    def wrapper():
        return f"<i>{func()}</i>"
    return wrapper

@bold
@italic
def text():
    return "Hello"

print(text())
```
**Output:** `<b><i>Hello</i></b>`
**Explanation:** Decorators apply bottom-up (closest to the function first), so `italic` wraps first, then `bold` wraps the result.

---

# PART 9 — CONTEXT MANAGERS (Q146–Q150)

### Q146. Custom Context Manager Class 🟡
```python
class Timer:
    def __enter__(self):
        import time
        self.start = time.perf_counter()
        return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        import time
        self.elapsed = time.perf_counter() - self.start
        print(f"Elapsed: {self.elapsed:.6f}s")
        return False

with Timer() as t:
    sum(range(1000000))
```
**Follow-up:** Real ML use: timing training epochs or inference latency without cluttering business logic with manual start/stop calls.

### Q147. `contextlib.contextmanager` Decorator 🟡
```python
from contextlib import contextmanager

@contextmanager
def managed_resource(name):
    print(f"Acquiring {name}")
    try:
        yield name
    finally:
        print(f"Releasing {name}")

with managed_resource("GPU-0") as res:
    print(f"Using {res}")
```
**Explanation:** Code before `yield` acts as `__enter__`; code after (in `finally`) acts as `__exit__` — a much simpler way to write context managers than a full class.

### Q148. Exception Propagation Through `__exit__` 🔴
```python
class SafeDivider:
    def __enter__(self):
        return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is ZeroDivisionError:
            print("Handled division by zero")
            return True
        return False

with SafeDivider():
    print(10 / 0)
print("Program continues")

with SafeDivider():
    print(10 / "a")
```
**Explanation:** The first `with` suppresses `ZeroDivisionError` and execution continues; the second raises `TypeError` uncaught since `__exit__` only suppresses `ZeroDivisionError`, terminating the program at that line.

### Q149. Nested Context Managers with Shared Resource 🔴
```python
from contextlib import contextmanager

@contextmanager
def gpu_context(device_id):
    print(f"Allocating GPU {device_id}")
    try:
        yield device_id
    except Exception as e:
        print(f"Error during GPU {device_id} usage: {e}")
        raise
    finally:
        print(f"Freeing GPU {device_id}")

try:
    with gpu_context(0), gpu_context(1):
        raise RuntimeError("CUDA OOM")
except RuntimeError:
    print("Caught at top level")
```
**Follow-up:** Order of `__exit__` calls in multiple context managers (reverse of entry order, like a stack) — GPU 1 frees before GPU 0.

### Q150. `contextlib.suppress` and ExitStack 🔴
```python
from contextlib import suppress, ExitStack

with suppress(FileNotFoundError):
    open('/tmp/does_not_exist_ever.txt')
print("Continued after suppress")

files_to_open = ['/tmp/a.txt', '/tmp/b.txt']
for f in files_to_open:
    open(f, 'w').close()

with ExitStack() as stack:
    handles = [stack.enter_context(open(f)) for f in files_to_open]
    print(f"Opened {len(handles)} files simultaneously")
```
**Explanation:** `ExitStack` dynamically manages a variable number of context managers (unknown at write-time, e.g., opening N files based on a runtime list) and guarantees all are closed in reverse order on exit.
**Follow-up:** Real ML use: managing a dynamic number of open dataset shard files or distributed process groups.

---

## Interviewer's Scoring Rubric

| Signal | What it tells you |
|---|---|
| Predicts wrong output but explains reasoning well | Understands concept, needs more hands-on exposure — often still hireable for senior roles with mentorship |
| Predicts right output but can't explain why | Pattern-memorization risk — probe deeper with a variant question |
| Identifies the "trick" unprompted | Strong fundamentals — good signal for senior-level code review responsibility |
| Proposes a correct fix | Practical engineering maturity, not just theoretical knowledge |
| Connects the question to a real ML/production scenario | High signal for senior IC roles — shows systems thinking beyond syntax |

**Suggested interview flow:** Pick 8–12 questions spanning all categories (don't restrict to just tricky syntax) — include at least 2 OOP design questions, 2 exception-handling questions, 1 file-handling question, and 2–3 nested-data-structure questions, since these best predict real production code quality for a Senior AI/ML Engineer.
