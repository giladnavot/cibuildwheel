---
title: Understanding Platform Compatibility in cibuildwheel
---
Platform Compatibility in cibuildwheel refers to the ability of the tool to build Python wheels across multiple platforms such as Windows, Linux, and MacOS. This is achieved through various functions and modules that handle the specifics of each platform. For instance, the `container_image_for_python_configuration` function in `linux.py` determines the appropriate container image for the Python configuration based on the platform architecture. Similarly, the `setup_setuptools_cross_compile` function in `windows.py` sets up the environment for cross-compilation on Windows. The `get_nuget_args` function also in `windows.py` retrieves arguments for the NuGet package manager based on the platform architecture. The `platform_module` imported in `windows.py` is a built-in Python module used for accessing underlying platform’s identifying data.

<SwmSnippet path="/cibuildwheel/windows.py" line="4">

---

# Importing the platform module

The platform module is imported as `platform_module`. This module provides a portable way of using operating system dependent functionality, which is essential for ensuring platform compatibility.

```python
import platform as platform_module
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/windows.py" line="143">

---

# Setting up cross-compilation

The `setup_setuptools_cross_compile` function sets up the environment for cross-compilation. It maps the architecture to the platform name, sets the platform name for build, build_ext, and bdist_wheel, and sets up other necessary environment variables for cross-compilation.

```python
def setup_setuptools_cross_compile(
    tmp: Path,
    python_configuration: PythonConfiguration,
    python_libs_base: Path,
    env: MutableMapping[str, str],
) -> None:
    distutils_cfg = tmp / "extra-setup.cfg"
    env["DIST_EXTRA_CONFIG"] = str(distutils_cfg)
    log.notice(f"Setting DIST_EXTRA_CONFIG={distutils_cfg} for cross-compilation")

    # Ensure our additional import libraries are made available, and explicitly
    # set the platform name
    map_plat = {"32": "win32", "64": "win-amd64", "ARM64": "win-arm64"}
    plat_name = map_plat[python_configuration.arch]
    # (This file must be default/locale encoding, so we can't pass 'encoding')
    distutils_cfg.write_text(
        textwrap.dedent(
            f"""\
            [build]
            plat_name={plat_name}
            [build_ext]
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/windows.py" line="330">

---

# Using the setup_setuptools_cross_compile function

The `setup_setuptools_cross_compile` function is used within the `setup_python` function to set up the environment for cross-compilation.

```python
        setup_setuptools_cross_compile(tmp, python_configuration, python_libs_base, env)
        setup_rust_cross_compile(tmp, python_configuration, python_libs_base, env)
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
