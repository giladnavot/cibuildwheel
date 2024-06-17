---
title: FatalError Class Overview
---
This document will cover the `FatalError` class in the cibuildwheel project. We'll cover:

1. What is `FatalError`
2. Variables and functions in `FatalError`
3. An example of how to use `FatalError`

```mermaid
graph TD;
  FatalError:::currentBaseStyle
FatalError --> ConfigurationError
FatalError --> DeprecationError
FatalError --> 1[...]

 classDef currentBaseStyle color:#000000,fill:#7CB9F4
```

# What is FatalError

`FatalError` is a custom exception class in the cibuildwheel project. It is designed to handle errors that can cause the build to fail. When an error of this type is raised, the error message is printed to stderr and the process is terminated. This provides a better error message and optional traceback within cibuildwheel.

<SwmSnippet path="/cibuildwheel/errors.py" line="15">

---

# Variables and functions

`return_code` is a class variable in `FatalError` that stores the return code of the error. It is an integer and its default value is 1.

```python
    return_code: int = 1
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/errors.py" line="18">

---

# Usage example

`ConfigurationError` is an example of how `FatalError` is used. It is a subclass of `FatalError` and it overrides the `return_code` variable with a value of 2.

```python
class ConfigurationError(FatalError):
    return_code = 2
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="class"><sup>Powered by [Swimm](/)</sup></SwmMeta>
