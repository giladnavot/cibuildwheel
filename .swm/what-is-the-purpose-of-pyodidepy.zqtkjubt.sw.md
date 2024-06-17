---
title: what is the purpose of pyodide.py?
---
The <SwmPath repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" path="cibuildwheel/pyodide.py">`(cibuildwheel) cibuildwheel/pyodide.py`</SwmPath> file in the cibuildwheel repository seems to be involved in setting up a Python environment for building Python wheels. It includes functions for setting up Python, getting Python configurations, and handling errors.

<SwmSnippet path="/cibuildwheel/pyodide.py" line="110" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==">

---

The <SwmToken path="cibuildwheel/pyodide.py" pos="110:2:2" line-data="def setup_python(" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`setup_python`</SwmToken> function appears to be responsible for setting up the Python environment for building wheels. It uses a <SwmToken path="cibuildwheel/pyodide.py" pos="112:4:4" line-data="    python_configuration: PythonConfiguration," repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`PythonConfiguration`</SwmToken> object to determine the Python version, Pyodide version, Emscripten version, and Node version to use.

```python
def setup_python(
    tmp: Path,
    python_configuration: PythonConfiguration,
    dependency_constraint_flags: Sequence[PathOrStr],
    environment: ParsedEnvironment,
) -> dict[str, str]:
    base_python = get_base_python(python_configuration.identifier)

    log.step("Setting up build environment...")
    venv_path = tmp / "venv"
    env = virtualenv(python_configuration.version, base_python, venv_path, [], use_uv=False)
    venv_bin_path = venv_path / "bin"
    assert venv_bin_path.exists()
    env["PIP_DISABLE_PIP_VERSION_CHECK"] = "1"

    # upgrade pip to the version matching our constraints
    # if necessary, reinstall it to ensure that it's available on PATH as 'pip'
    call(
        "python",
        "-m",
        "pip",
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/pyodide.py" line="181" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==">

---

The <SwmToken path="cibuildwheel/pyodide.py" pos="181:2:2" line-data="def get_python_configurations(" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`get_python_configurations`</SwmToken> function seems to be used to get a list of Python configurations that match the build selector. It uses the <SwmToken path="cibuildwheel/pyodide.py" pos="185:5:5" line-data="    full_python_configs = read_python_configs(&quot;pyodide&quot;)" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`read_python_configs`</SwmToken> function to read the Python configurations from a file.

```python
def get_python_configurations(
    build_selector: BuildSelector,
    architectures: Set[Architecture],  # noqa: ARG001
) -> list[PythonConfiguration]:
    full_python_configs = read_python_configs("pyodide")

    python_configurations = [PythonConfiguration(**item) for item in full_python_configs]
    python_configurations = [c for c in python_configurations if build_selector(c.identifier)]
    return python_configurations
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/pyodide.py" line="94" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==">

---

The <SwmToken path="cibuildwheel/pyodide.py" pos="94:2:2" line-data="def get_base_python(identifier: str) -&gt; Path:" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`get_base_python`</SwmToken> function is used within <SwmToken path="cibuildwheel/pyodide.py" pos="110:2:2" line-data="def setup_python(" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`setup_python`</SwmToken> to determine the base Python path based on the identifier of the Python configuration.

```python
def get_base_python(identifier: str) -> Path:
    implementation_id = identifier.split("-")[0]
    majorminor = implementation_id[len("cp") :]
    version_info = (int(majorminor[0]), int(majorminor[1:]))
    if version_info == sys.version_info[:2]:
        return Path(sys.executable)

    major_minor = ".".join(str(v) for v in version_info)
    python_name = f"python{major_minor}"
    which_python = shutil.which(python_name)
    if which_python is None:
        msg = f"CPython {major_minor} is not installed."
        raise errors.FatalError(msg)
    return Path(which_python)
```

---

</SwmSnippet>

<SwmMeta version="3.0.0"><sup>Powered by [Swimm](/)</sup></SwmMeta>
