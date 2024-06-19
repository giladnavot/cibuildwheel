---
title: Exploring Wheels Building
---
Wheels Building refers to the process of creating a wheel, which is a built-package format for Python. A wheel is a ZIP-format archive with a specially formatted filename and the .whl extension. It contains all the files necessary for the installation of a Python app.&nbsp;

In cibuildwheel, wheels are built for different Python versions and platforms. The configurations for these builds are specified in the <SwmPath>[cibuildwheel/resources/build-platforms.toml](/cibuildwheel/resources/build-platforms.toml)</SwmPath> file. The actual building process is handled by the 'build' function in the respective platform-specific files (<SwmPath>[cibuildwheel/macos.py](/cibuildwheel/macos.py)</SwmPath>, <SwmPath>[cibuildwheel/windows.py](/cibuildwheel/windows.py)</SwmPath>, <SwmPath>[cibuildwheel/linux.py](/cibuildwheel/linux.py)</SwmPath>, <SwmPath>[cibuildwheel/pyodide.py](/cibuildwheel/pyodide.py)</SwmPath>).&nbsp;

The build process involves setting up the build environment, running pre-build commands, building the wheel, and repairing the wheel if necessary.

<SwmSnippet path="/cibuildwheel/resources/build-platforms.toml" line="1">

---

# Building Wheels in Different Environments

This file contains the configurations for different Python versions and platforms. These configurations are used when building wheels.

```toml
[linux]
python_configurations = [
  { identifier = "cp36-manylinux_x86_64", version = "3.6", path_str = "/opt/python/cp36-cp36m" },
  { identifier = "cp37-manylinux_x86_64", version = "3.7", path_str = "/opt/python/cp37-cp37m" },
  { identifier = "cp38-manylinux_x86_64", version = "3.8", path_str = "/opt/python/cp38-cp38" },
  { identifier = "cp39-manylinux_x86_64", version = "3.9", path_str = "/opt/python/cp39-cp39" },
  { identifier = "cp310-manylinux_x86_64", version = "3.10", path_str = "/opt/python/cp310-cp310" },
  { identifier = "cp311-manylinux_x86_64", version = "3.11", path_str = "/opt/python/cp311-cp311" },
  { identifier = "cp312-manylinux_x86_64", version = "3.12", path_str = "/opt/python/cp312-cp312" },
  { identifier = "cp313-manylinux_x86_64", version = "3.13", path_str = "/opt/python/cp313-cp313" },
  { identifier = "cp313t-manylinux_x86_64", version = "3.13", path_str = "/opt/python/cp313-cp313t" },
  { identifier = "cp36-manylinux_i686", version = "3.6", path_str = "/opt/python/cp36-cp36m" },
  { identifier = "cp37-manylinux_i686", version = "3.7", path_str = "/opt/python/cp37-cp37m" },
  { identifier = "cp38-manylinux_i686", version = "3.8", path_str = "/opt/python/cp38-cp38" },
  { identifier = "cp39-manylinux_i686", version = "3.9", path_str = "/opt/python/cp39-cp39" },
  { identifier = "cp310-manylinux_i686", version = "3.10", path_str = "/opt/python/cp310-cp310" },
  { identifier = "cp311-manylinux_i686", version = "3.11", path_str = "/opt/python/cp311-cp311" },
  { identifier = "cp312-manylinux_i686", version = "3.12", path_str = "/opt/python/cp312-cp312" },
  { identifier = "cp313-manylinux_i686", version = "3.13", path_str = "/opt/python/cp313-cp313" },
  { identifier = "cp313t-manylinux_i686", version = "3.13", path_str = "/opt/python/cp313-cp313t" },
  { identifier = "pp37-manylinux_x86_64", version = "3.7", path_str = "/opt/python/pp37-pypy37_pp73" },
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/windows.py" line="336">

---

# Building Wheels on Different Platforms

The `build` function in `windows.py` is an example of how wheels are built for a specific platform. Similar functions exist in `linux.py`, `macos.py`, and `pyodide.py` for their respective platforms.

```python
def build(options: Options, tmp_path: Path) -> None:
    python_configurations = get_python_configurations(
        options.globals.build_selector, options.globals.architectures
    )

    if not python_configurations:
        return

    try:
        before_all_options_identifier = python_configurations[0].identifier
        before_all_options = options.build_options(before_all_options_identifier)

        if before_all_options.before_all:
            log.step("Running before_all...")
            env = before_all_options.environment.as_dictionary(prev_environment=os.environ)
            before_all_prepared = prepare_command(
                before_all_options.before_all, project=".", package=options.globals.package_dir
            )
            shell(before_all_prepared, env=env)

        built_wheels: list[Path] = []
```

---

</SwmSnippet>

# 

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
