---
title: Introduction to Platform Support
---
Platform Support in cibuildwheel refers to the ability of the tool to build Python wheels across multiple platforms. This includes Linux, macOS, Windows, and Pyodide. The tool identifies the platform it's running on and uses the appropriate module for that platform. It also supports different architectures for each platform. For instance, on Linux, it can build wheels for x86_64, i686, and aarch64 architectures. The platform and architecture are determined automatically, but can also be specified by the user.

<SwmSnippet path="/cibuildwheel/resources/build-platforms.toml" line="1">

---

# Platform Configuration

This is the <SwmPath>[cibuildwheel/resources/build-platforms.toml](/cibuildwheel/resources/build-platforms.toml)</SwmPath> file which contains the configurations for each platform. Each platform has a list of Python configurations, specifying the Python versions and architectures supported. For example, for Linux, it supports Python versions from 3.6 to 3.13 on various architectures like x86_64, i686, aarch64, ppc64le, and s390x.

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

<SwmSnippet path="/cibuildwheel/__main__.py" line="274">

---

# Platform Module

The <SwmToken path="/cibuildwheel/__main__.py" pos="275:2:2" line-data="def get_platform_module(platform: PlatformName) -&gt; PlatformModule:">`get_platform_module`</SwmToken> function in <SwmPath>[cibuildwheel/\__main_\_.py](/cibuildwheel/__main__.py)</SwmPath> returns the platform-specific module based on the platform name. These modules are used to build the Python wheels according to the configurations specified in the <SwmPath>[cibuildwheel/resources/build-platforms.toml](/cibuildwheel/resources/build-platforms.toml)</SwmPath> file.

```python
# pylint: disable-next=inconsistent-return-statements
def get_platform_module(platform: PlatformName) -> PlatformModule:
    if platform == "linux":
        return cibuildwheel.linux
    if platform == "windows":
        return cibuildwheel.windows
    if platform == "macos":
        return cibuildwheel.macos
    if platform == "pyodide":
        return cibuildwheel.pyodide
    assert_never(platform)
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/architecture.py" line="79">

---

# Native Architecture

The <SwmToken path="/cibuildwheel/architecture.py" pos="79:3:3" line-data="    def native_arch(platform: PlatformName) -&gt; Architecture | None:">`native_arch`</SwmToken> function in <SwmPath>[cibuildwheel/architecture.py](/cibuildwheel/architecture.py)</SwmPath> determines the native architecture of the platform. This is used to ensure that the Python wheels are built for the correct architecture.

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

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
