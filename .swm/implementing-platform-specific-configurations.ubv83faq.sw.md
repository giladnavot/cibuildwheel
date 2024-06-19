---
title: 'Implementing Platform-Specific Configurations '
---
This document will cover the implementation of platform-specific configurations in cibuildwheel, which includes:

1. How platform-specific configurations are defined
2. How these configurations are used in the codebase
3. The role of the `tool` constant in platform-specific configurations.

<SwmSnippet path="/cibuildwheel/resources/defaults.toml" line="1">

---

# Defining Platform-Specific Configurations

Platform-specific configurations are defined in the <SwmPath>[cibuildwheel/resources/defaults.toml](/cibuildwheel/resources/defaults.toml)</SwmPath> file. This file contains global constants that define the default settings for the cibuildwheel tool. Each platform (Linux, macOS, Windows, Pyodide) has its own section with specific settings.

```toml
[tool.cibuildwheel]
build = "*"
skip = ""
test-skip = ""
free-threaded-support = false

archs = ["auto"]
build-frontend = "default"
config-settings = {}
dependency-versions = "pinned"
environment = {}
environment-pass = []
build-verbosity = 0

before-all = ""
before-build = ""
repair-wheel-command = ""

test-command = ""
before-test = ""
test-requires = []
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/__main__.py" line="21">

---

# Using Platform-Specific Configurations in the Codebase

The platform-specific configurations are imported and used in the <SwmPath>[cibuildwheel/\__main_\_.py](/cibuildwheel/__main__.py)</SwmPath> file. The <SwmToken path="/cibuildwheel/architecture.py" pos="29:2:2" line-data="class Architecture(Enum):">`Architecture`</SwmToken> and <SwmToken path="/cibuildwheel/__main__.py" pos="28:8:8" line-data="from cibuildwheel.typing import PLATFORMS, GenericPythonConfiguration, PlatformName">`PLATFORMS`</SwmToken> constants from the <SwmToken path="/cibuildwheel/__main__.py" pos="25:2:4" line-data="from cibuildwheel.architecture import Architecture, allowed_architectures_check">`cibuildwheel.architecture`</SwmToken> and <SwmToken path="/cibuildwheel/__main__.py" pos="28:2:4" line-data="from cibuildwheel.typing import PLATFORMS, GenericPythonConfiguration, PlatformName">`cibuildwheel.typing`</SwmToken> modules respectively, are used to determine the platform and architecture for the build process.

```python
import cibuildwheel.util
import cibuildwheel.windows
from cibuildwheel import errors
from cibuildwheel._compat.typing import assert_never
from cibuildwheel.architecture import Architecture, allowed_architectures_check
from cibuildwheel.logger import log
from cibuildwheel.options import CommandLineArguments, Options, compute_options
from cibuildwheel.typing import PLATFORMS, GenericPythonConfiguration, PlatformName
from cibuildwheel.util import (
    CIBW_CACHE_PATH,
    BuildSelector,
    CIProvider,
    Unbuffered,
    chdir,
    detect_ci_provider,
    fix_ansi_codes_for_github_actions,
    strtobool,
)
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/resources/defaults.toml" line="1">

---

# The Role of the <SwmToken path="/cibuildwheel/resources/defaults.toml" pos="1:1:1" line-data="[tool.cibuildwheel]">`tool`</SwmToken> Constant

The <SwmToken path="/cibuildwheel/resources/defaults.toml" pos="1:1:1" line-data="[tool.cibuildwheel]">`tool`</SwmToken> constant is used to specify the settings for each platform. It is used in the <SwmPath>[cibuildwheel/resources/defaults.toml](/cibuildwheel/resources/defaults.toml)</SwmPath> file to define the default settings for each platform. It is also used in the <SwmPath>[cibuildwheel/options.py](/cibuildwheel/options.py)</SwmPath> file to handle any overrides specified by the user.

```toml
[tool.cibuildwheel]
build = "*"
skip = ""
test-skip = ""
free-threaded-support = false

archs = ["auto"]
build-frontend = "default"
config-settings = {}
dependency-versions = "pinned"
environment = {}
environment-pass = []
build-verbosity = 0

before-all = ""
before-build = ""
repair-wheel-command = ""

test-command = ""
before-test = ""
test-requires = []
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
