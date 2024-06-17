---
title: 'Balancing Platform-Specific Needs with Unified Code Base '
---
This document will cover how cibuildwheel handles platform-specific needs while maintaining a unified code base. We'll cover:

1. How cibuildwheel identifies the platform
2. How it handles platform-specific architectures
3. How it manages platform-specific Python configurations.

<SwmSnippet path="/cibuildwheel/__main__.py" line="213">

---

# Identifying the Platform

The function `_compute_platform_only` is used to identify the platform based on the `only` argument. It checks for platform-specific strings in the `only` argument and returns the corresponding platform name.

```python
def _compute_platform_only(only: str) -> PlatformName:
    if "linux_" in only:
        return "linux"
    if "macosx_" in only:
        return "macos"
    if "win_" in only or "win32" in only:
        return "windows"
    if "pyodide_" in only:
        return "pyodide"
    msg = f"Invalid --only='{only}', must be a build selector with a known platform"
    raise errors.ConfigurationError(msg)
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/__main__.py" line="226">

---

The function `_compute_platform_auto` is used to automatically identify the platform based on the system's platform. It checks the `sys.platform` and returns the corresponding platform name.

```python
def _compute_platform_auto() -> PlatformName:
    if sys.platform.startswith("linux"):
        return "linux"
    elif sys.platform == "darwin":
        return "macos"
    elif sys.platform == "win32":
        return "windows"
    else:
        msg = (
            'cibuildwheel: Unable to detect platform from "sys.platform". cibuildwheel doesn\'t '
            "support building wheels for this platform. You might be able to build for a different "
            "platform using the --platform argument. Check --help output for more information."
        )
        raise errors.ConfigurationError(msg)
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/architecture.py" line="79">

---

# Handling Platform-Specific Architectures

The function `native_arch` is used to determine the native architecture of the platform. It checks the platform and returns the corresponding architecture. It also handles the case where the same architecture can have different names on different platforms.

```python
    def native_arch(platform: PlatformName) -> Architecture | None:
        if platform == "pyodide":
            return Architecture.wasm32

        # Cross-platform support. Used for --print-build-identifiers or docker builds.
        host_platform: PlatformName = (
            "windows"
            if sys.platform.startswith("win")
            else ("macos" if sys.platform.startswith("darwin") else "linux")
        )

        native_machine = platform_module.machine()
        native_architecture = Architecture(native_machine)

        # we might need to rename the native arch to the machine we're running
        # on, as the same arch can have different names on different platforms
        if host_platform != platform:
            for arch_synonym in ARCH_SYNONYMS:
                if native_machine == arch_synonym.get(host_platform):
                    synonym = arch_synonym[platform]

```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/architecture.py" line="109">

---

The function `auto_archs` is used to automatically determine the architectures that can be built on the platform. It uses the `native_arch` function to get the native architecture and adds additional architectures based on the platform.

```python
    def auto_archs(platform: PlatformName) -> set[Architecture]:
        native_arch = Architecture.native_arch(platform)
        if native_arch is None:
            return set()  # can't build anything on this platform
        result = {native_arch}

        if platform == "linux" and Architecture.x86_64 in result:
            # x86_64 machines can run i686 containers
            result.add(Architecture.i686)

        if platform == "windows" and Architecture.AMD64 in result:
            result.add(Architecture.x86)

        return result
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/resources/build-platforms.toml" line="1">

---

# Managing Platform-Specific Python Configurations

The `build-platforms.toml` file contains platform-specific Python configurations. Each platform has a list of Python configurations with identifiers, versions, and other platform-specific details.

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

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="follow-up"><sup>Powered by [Swimm](/)</sup></SwmMeta>
