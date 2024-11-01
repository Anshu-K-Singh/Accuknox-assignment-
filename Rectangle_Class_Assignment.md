
# Assignment: Rectangle Class with Iteration

You need to make a `Rectangle` class. It has some rules. Here they go:

1. When you make a `Rectangle`, you give it two numbers: `length` and `width`, which have to be integers (`length: int` and `width: int`).
2. You can use a `for` loop on `Rectangle` objects.
3. When you use the loop, it will first give you the length in a dictionary format like `{'length': <VALUE_OF_LENGTH>}` and then the width like `{'width': <VALUE_OF_WIDTH>}`.

Here’s a simple way to do this.

```python
class Rectangle:
    def __init__(self, length: int, width: int):
        self.length = length
        self.width = width

    def __iter__(self):
        yield {'length': self.length}
        yield {'width': self.width}

# Example usage
rect = Rectangle(10, 5)
for value in rect:
    print(value)
```

### Expected Output:
```
{'length': 10}
{'width': 5}
```

Make sure it works like this.

---

## Alternative Solution Without `yield`

If you don't use `yield`, it won’t work the same way. The `yield` statement makes `__iter__` a generator, allowing it to produce values one at a time when iterating over the `Rectangle` instance.

Without `yield`, we’d have to return an iterator explicitly, like this:

```python
class Rectangle:
    def __init__(self, length: int, width: int):
        self.length = length
        self.width = width

    def __iter__(self):
        return iter([{'length': self.length}, {'width': self.width}])

# Example usage
rect = Rectangle(10, 5)
for value in rect:
    print(value)
```

This will give the same output when iterating:

```
{'length': 10}
{'width': 5}
```

Using `yield` is often simpler when we want to create a sequence of values without building a list first, but returning an iterator also works.
