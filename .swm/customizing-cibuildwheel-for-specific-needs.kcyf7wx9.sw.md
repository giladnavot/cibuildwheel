---
title: Customizing cibuildwheel for Specific Needs
---
This document will cover the process of customizing cibuildwheel for specific needs, which includes:

1. Understanding the cibuildwheel configuration
2. Customizing the build process
3. Customizing the platform and architecture
4. Customizing the environment variables

<SwmSnippet path="/cibuildwheel/resources/defaults.toml" line="1">

---

# Understanding the cibuildwheel configuration

The `defaults.toml` file contains the default configuration for cibuildwheel. It includes settings for build, skip, test-skip, free-threaded-support, archs, config-settings, environment, build-verbosity, before-all, repair-wheel-command, test-command, test-requires, and container-engine. These settings can be customized to fit specific needs.

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

<SwmSnippet path="/cibuildwheel/__main__.py" line="287">

---

# Customizing the build process

The `build_in_directory` function in `__main__.py` is the main entry point for the build process. It computes the platform, checks for invalid configurations, and creates the output directory. This function can be modified to customize the build process.

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

<SwmSnippet path="/cibuildwheel/__main__.py" line="242">

---

# Customizing the platform and architecture

The `_compute_platform` function in `__main__.py` is used to compute the platform for the build. It can be customized to support different platforms or architectures.

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

<SwmSnippet path="/cibuildwheel/__main__.py" line="317">

---

# Customizing the environment variables

Environment variables can be customized in the `build_in_directory` function. For example, the `CIBUILDWHEEL` environment variable is set to '1' during the build process.

```python
    os.environ["CIBUILDWHEEL"] = "1"

```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="follow-up"><sup>Powered by [Swimm](/)</sup></SwmMeta>
