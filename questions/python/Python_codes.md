# سوال های ساده (warm up)
1-تابعی بنویسید که یه رشته بگیرد و معکوسش را برگرداند ( بدون استفاده از [-1:])

```python
def reverse_string(s: str) -> str:
    result = ""
    for ch in s:
        result = ch + result 
    return result

print(reverse_string("python"))  # out put: nohtyp
```

```python
def reverse_string(s: str) -> str:
    chars = list(s)
    chars_reversed = []
    for ch in reversed(chars):
        chars_reversed.append(ch)
    return "".join(chars_reversed)

print(reverse_string("hello"))  # out put: olleh
```
---
