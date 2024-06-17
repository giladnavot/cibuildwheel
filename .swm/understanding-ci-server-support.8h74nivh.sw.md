---
title: 'Understanding CI Server Support '
---
This document will cover how cibuildwheel supports various Continuous Integration (CI) servers. We'll cover:

1. The structure of the cibuildwheel module
2. How cibuildwheel determines the platform and architecture
3. The role of the build_in_directory function
4. The configuration options for different platforms.

<SwmSnippet path="/cibuildwheel/__main__.py" line="21">

---

# Structure of the cibuildwheel module

The cibuildwheel module imports several other modules and functions, including errors, Architecture, log, CIProvider, chdir, detect_ci_provider, and strtobool. These are used to handle errors, determine the architecture, log events, handle CI providers, change directories, detect the CI provider, and convert strings to boolean values respectively.

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

<SwmSnippet path="/cibuildwheel/__main__.py" line="242">

---

# Determining the platform and architecture

The \_compute_platform function is used to determine the platform based on the command line arguments and environment variables. If the --only argument is used, the platform is computed based on this argument. Otherwise, the platform is either specified by the --platform argument or the CIBW_PLATFORM environment variable, or it is automatically determined.

```python
def _compute_platform(args: CommandLineArguments) -> PlatformName:
    platform_option_value = args.platform or os.environ.get("CIBW_PLATFORM", "auto")

    if args.only and args.platform is not None:
        msg = "--platform cannot be specified with --only, it is computed from --only"
        raise errors.ConfigurationError(msg)
    if args.only and args.archs is not None:
        msg = "--arch cannot be specified with --only, it is computed from --only"
        raise errors.ConfigurationError(msg)

    if platform_option_value not in PLATFORMS | {"auto"}:
        msg = f"Unsupported platform: {platform_option_value}"
        raise errors.ConfigurationError(msg)

    if args.only:
        return _compute_platform_only(args.only)
    elif platform_option_value != "auto":
        return typing.cast(PlatformName, platform_option_value)

    return _compute_platform_auto()
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/__main__.py" line="287">

---

# Role of the build_in_directory function

The build_in_directory function is the main function that handles the building of wheels in the specified directory. It first computes the platform and then the options based on the platform, command line arguments, and environment variables. It then checks if the package directory contains the necessary files for building. The function then gets the platform module and build identifiers, and starts the building process.

```python
def build_in_directory(args: CommandLineArguments) -> None:
    platform: PlatformName = _compute_platform(args)
    if platform == "pyodide" and sys.platform == "win32":
        msg = "cibuildwheel: Building for pyodide is not supported on Windows"
        print(msg, file=sys.stderr)
        sys.exit(2)

    options = compute_options(platform=platform, command_line_arguments=args, env=os.environ)

    package_dir = options.globals.package_dir
    package_files = {"setup.py", "setup.cfg", "pyproject.toml"}

    if not any(package_dir.joinpath(name).exists() for name in package_files):
        names = ", ".join(sorted(package_files, reverse=True))
        msg = f"Could not find any of {{{names}}} at root of package"
        raise errors.ConfigurationError(msg)

    platform_module = get_platform_module(platform)
    identifiers = get_build_identifiers(
        platform_module=platform_module,
        build_selector=options.globals.build_selector,
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/resources/defaults.toml" line="1">

---

# Configuration options for different platforms

The defaults.toml file contains the default configuration options for cibuildwheel. These options can be overridden by the user. The file contains sections for the tool.cibuildwheel, tool.cibuildwheel.linux, tool.cibuildwheel.macos, tool.cibuildwheel.windows, and tool.cibuildwheel.pyodide, each containing platform-specific configuration options.

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

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="follow-up"><sup>Powered by [Swimm](/)</sup></SwmMeta>
