---
title: Python的特殊变量
created: 2026-07-18
source: gpt-5.6-terra
tags: 
---
Python 会给每个“模块”自动设置一个名为 `__name__` 的变量；它的值取决于这个模块是“被直接运行”，还是“被其他代码导入”。

原句中有一点不够准确：`__name__` 并不总是“Python 文件的名称”，而是当前模块在 Python 中的模块名。

例如有两个文件：

```text
project/
├── main.py
└── helper.py
```

`helper.py` 内容：

```python
print(__name__)
```

`main.py` 内容：

```python
import helper
```

如果执行：

```bash
python main.py
```

输出是：

```text
helper
```

原因是：

- Python 执行 `main.py`。
- `main.py` 导入了 `helper.py`。
- `helper.py` 是作为名为 `helper` 的模块被导入的，所以 `helper.py` 中的 `__name__` 是 `"helper"`。

而如果直接执行：

```bash
python helper.py
```

输出则是：

```text
__main__
```

原因是：

- `helper.py` 此时不是被 `import` 导入，而是 Python 命令的直接执行入口。
- Python 会把“当前直接运行的入口模块”的 `__name__` 设为 `"__main__"`。
- 这不表示文件名真的叫 `__main__.py`，只是一个特殊标记，表示“程序从这里开始执行”。

最常见的用途是这一段：

```python
def main():
    print("开始执行程序")

if __name__ == "__main__":
    main()
```

它的含义是：

> 只有当这个文件被直接执行时，才调用 `main()`；如果它只是被其他文件 `import`，就不要自动执行 `main()`。

例如 `tool.py`：

```python
def add(a, b):
    return a + b

if __name__ == "__main__":
    print(add(1, 2))
```

直接执行：

```bash
python tool.py
```

会输出：

```text
3
```

但在另一个文件中：

```python
import tool

print(tool.add(3, 4))
```

输出只有：

```text
7
```

不会额外输出 `3`，因为此时：

```python
tool.__name__ == "tool"
```

所以这段不会运行：

```python
if __name__ == "__main__":
    print(add(1, 2))
```

可以这样记忆：

| 使用方式 | `__name__` 的值 |
|---|---|
| 直接运行 `python file.py` | `"__main__"` |
| 用 `import file` 导入 | `"file"` |
| 以包方式导入，如 `import package.file` | `"package.file"` |
| 用模块方式运行，如 `python -m package.file` | `"__main__"` |

最后一种容易混淆：

```bash
python -m package.file
```

虽然运行的是 `package.file`，但它仍然是“直接执行入口”，因此其 `__name__` == `"__main__"`。

更准确地说：

> `__name__` 是 Python 自动为每个模块提供的特殊变量。模块被导入时，它的值通常是模块的导入名称；模块作为程序入口被直接运行时，它的值为 `"__main__"`。
