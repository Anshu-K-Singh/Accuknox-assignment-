
# Assignment: Rectangle Class with Iteration

You need to make a `Rectangle` class. It has some rules. Here they go:

1. When you make a `Rectangle`, you give it two numbers: `length` and `width`, which have to be integers (`length: int` and `width: int`).
2. You can use a `for` loop on `Rectangle` objects.
3. When you use the loop, it will first give you the length in a dictionary format like `{'length': <VALUE_OF_LENGTH>}` and then the width like `{'width': <VALUE_OF_WIDTH>}`.

solution



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

This will give the  output when iterating:

```
{'length': 10}
{'width': 5}
```
