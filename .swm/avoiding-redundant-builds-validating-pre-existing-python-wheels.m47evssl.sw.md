---
title: 'Avoiding Redundant Builds: Validating Pre-existing Python Wheels '
---
This document will cover the process of how cibuildwheel validates if a wheel has already been built for a specific Python configuration. The topics covered include:

1. The role of the <SwmToken path="/cibuildwheel/util.py" pos="483:2:2" line-data="class AlreadyBuiltWheelError(Exception):">`AlreadyBuiltWheelError`</SwmToken> class
2. The use of the <SwmToken path="/cibuildwheel/util.py" pos="760:2:2" line-data="def find_compatible_wheel(wheels: Sequence[T], identifier: str) -&gt; T | None:">`find_compatible_wheel`</SwmToken> function
3. The build process across different platforms.

<SwmSnippet path="/cibuildwheel/util.py" line="483">

---

# The role of the <SwmToken path="/cibuildwheel/util.py" pos="483:2:2" line-data="class AlreadyBuiltWheelError(Exception):">`AlreadyBuiltWheelError`</SwmToken> class

The <SwmToken path="/cibuildwheel/util.py" pos="483:2:2" line-data="class AlreadyBuiltWheelError(Exception):">`AlreadyBuiltWheelError`</SwmToken> class is used to raise an exception when a wheel with the same name has already been generated in the current run. The error message suggests checking the project configuration or running cibuildwheel with <SwmToken path="/cibuildwheel/util.py" pos="490:5:7" line-data="            cibuildwheel with CIBW_BUILD_VERBOSITY=1 to view build logs.">`CIBW_BUILD_VERBOSITY=1`</SwmToken> to view build logs.

```python
class AlreadyBuiltWheelError(Exception):
    def __init__(self, wheel_name: str) -> None:
        message = textwrap.dedent(
            f"""
            cibuildwheel: Build failed because a wheel named {wheel_name} was already generated in the current run.

            If you expected another wheel to be generated, check your project configuration, or run
            cibuildwheel with CIBW_BUILD_VERBOSITY=1 to view build logs.
            """
        )

        super().__init__(message)
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/util.py" line="760">

---

# The use of the <SwmToken path="/cibuildwheel/util.py" pos="760:2:2" line-data="def find_compatible_wheel(wheels: Sequence[T], identifier: str) -&gt; T | None:">`find_compatible_wheel`</SwmToken> function

The <SwmToken path="/cibuildwheel/util.py" pos="760:2:2" line-data="def find_compatible_wheel(wheels: Sequence[T], identifier: str) -&gt; T | None:">`find_compatible_wheel`</SwmToken> function is used to find a wheel with an <SwmToken path="/cibuildwheel/util.py" pos="773:10:10" line-data="            if tag.abi == &quot;abi3&quot; and not free_threaded:">`abi3`</SwmToken> or a none ABI tag in <SwmToken path="/cibuildwheel/util.py" pos="760:4:4" line-data="def find_compatible_wheel(wheels: Sequence[T], identifier: str) -&gt; T | None:">`wheels`</SwmToken> that is compatible with the Python interpreter specified by <SwmToken path="/cibuildwheel/util.py" pos="760:13:13" line-data="def find_compatible_wheel(wheels: Sequence[T], identifier: str) -&gt; T | None:">`identifier`</SwmToken> and that has been previously built. It checks various conditions to determine if a wheel is compatible.

```python
def find_compatible_wheel(wheels: Sequence[T], identifier: str) -> T | None:
    """
    Finds a wheel with an abi3 or a none ABI tag in `wheels` compatible with the Python interpreter
    specified by `identifier` that is previously built.
    """

    interpreter, platform = identifier.split("-")
    free_threaded = interpreter.endswith("t")
    if free_threaded:
        interpreter = interpreter[:-1]
    for wheel in wheels:
        _, _, _, tags = parse_wheel_filename(wheel.name)
        for tag in tags:
            if tag.abi == "abi3" and not free_threaded:
                # ABI3 wheels must start with cp3 for impl and tag
                if not (interpreter.startswith("cp3") and tag.interpreter.startswith("cp3")):
                    continue
            elif tag.abi == "none":
                # CPythonless wheels must include py3 tag
                if tag.interpreter[:3] != "py3":
                    continue
```

---

</SwmSnippet>

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
