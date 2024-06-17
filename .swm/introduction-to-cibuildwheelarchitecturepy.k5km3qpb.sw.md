---
title: Introduction to cibuildwheel/architecture.py
---
The `cibuildwheel/architecture.py` file in the cibuildwheel repository is responsible for handling different system architectures. It defines a class `Architecture` which is an enumeration of different system architectures like x86_64, i686, aarch64, etc. It also provides methods to parse architecture configurations, determine native architecture, and check for allowed architectures. This file is crucial for ensuring that the wheels are built correctly for the target system architecture.

<SwmSnippet path="/cibuildwheel/architecture.py" line="29">

---

# Architecture Class

The `Architecture` class is an enumeration of different architectures. It includes methods to parse the architecture from the configuration, determine the native architecture, and get all architectures for a platform. It also provides methods to sort and represent the architecture as a string.

```python
class Architecture(Enum):
    value: str

    # mac/linux archs
    x86_64 = "x86_64"

    # linux archs
    i686 = "i686"
    aarch64 = "aarch64"
    ppc64le = "ppc64le"
    s390x = "s390x"

    # mac archs
    universal2 = "universal2"
    arm64 = "arm64"

    # windows archs
    x86 = "x86"
    AMD64 = "AMD64"
    ARM64 = "ARM64"

```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/architecture.py" line="14">

---

# Architecture Constants

The `PRETTY_NAMES` and `ARCH_SYNONYMS` constants are dictionaries that map platform names to their pretty names and architecture synonyms respectively. These are used throughout the codebase to handle platform-specific naming conventions.

```python
PRETTY_NAMES: Final[dict[PlatformName, str]] = {
    "linux": "Linux",
    "macos": "macOS",
    "windows": "Windows",
    "pyodide": "Pyodide",
}

ARCH_SYNONYMS: Final[list[dict[PlatformName, str | None]]] = [
    {"linux": "x86_64", "macos": "x86_64", "windows": "AMD64"},
    {"linux": "i686", "macos": None, "windows": "x86"},
    {"linux": "aarch64", "macos": "arm64", "windows": "ARM64"},
]
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/__main__.py" line="25">

---

# Architecture Usage

The `Architecture` class and its related functions are imported and used in other modules. For example, in `__main__.py`, the `Architecture` class is imported and used to check the allowed architectures.

```python
from cibuildwheel.architecture import Architecture, allowed_architectures_check
```

---

</SwmSnippet>

# Architecture Class and Functions

This section covers the Architecture class and the allowed_architectures_check function in the cibuildwheel/architecture.py file.

<SwmSnippet path="/cibuildwheel/architecture.py" line="28">

---

## Architecture Class

The Architecture class is an enumeration of the different architectures supported by cibuildwheel. It includes methods for parsing and checking architectures, as well as properties for accessing the name and value of the architecture.

```python
@functools.total_ordering
class Architecture(Enum):
    value: str

    # mac/linux archs
    x86_64 = "x86_64"

    # linux archs
    i686 = "i686"
    aarch64 = "aarch64"
    ppc64le = "ppc64le"
    s390x = "s390x"

    # mac archs
    universal2 = "universal2"
    arm64 = "arm64"

    # windows archs
    x86 = "x86"
    AMD64 = "AMD64"
    ARM64 = "ARM64"
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/architecture.py" line="154">

---

## allowed_architectures_check Function

The allowed_architectures_check function checks if the given architectures are allowed for the specified platform. It raises a ValueError if any of the architectures are not allowed.

```python
def allowed_architectures_check(
    platform: PlatformName,
    architectures: Set[Architecture],
) -> None:
    allowed_architectures = Architecture.all_archs(platform)

    msg = f"{PRETTY_NAMES[platform]} only supports {sorted(allowed_architectures)} at the moment."

    if platform != "linux":
        msg += " If you want to set emulation architectures on Linux, use CIBW_ARCHS_LINUX instead."

    if not architectures <= allowed_architectures:
        msg = f"Invalid archs option {architectures}. " + msg
        raise ValueError(msg)

    if not architectures:
        msg = "Empty archs option set. " + msg
        raise ValueError(msg)
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
