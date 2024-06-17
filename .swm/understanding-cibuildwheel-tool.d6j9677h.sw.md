---
title: Understanding cibuildwheel Tool
---
cibuildwheel is a Python tool designed to simplify the process of building wheels across multiple platforms and Python versions. It provides a unified interface for configuring and running builds on different Continuous Integration (CI) servers. The tool is highly customizable, allowing developers to specify build, skip, test, and repair commands, among other settings, for each platform. It also supports different architectures and provides options for environment settings and verbosity levels. The tool's configuration and settings are primarily managed through the 'tool.cibuildwheel' section in the 'defaults.toml' file.

<SwmSnippet path="/cibuildwheel/resources/defaults.toml" line="1">

---

# Configuration of `tool.cibuildwheel`

This is the default configuration for `tool.cibuildwheel`. It specifies the default values for various settings such as `build`, `skip`, `test-skip`, `free-threaded-support`, and others. These settings can be overridden by the user in their own configuration file.

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

<SwmSnippet path="/cibuildwheel/options.py" line="278">

---

# Usage of `tool.cibuildwheel` in code

In the code, `tool.cibuildwheel` is used to read the configuration settings from the user's configuration file or environment variables. It checks for allowed options and raises an error if an option is not allowed.

```python
      otherwise the value of CIBW_COOL_COLOR, otherwise
      'tool.cibuildwheel.macos.cool-color' or 'tool.cibuildwheel.cool-color'
      from `config_file`, or from cibuildwheel/resources/defaults.toml. An
      error is thrown if there are any unexpected keys or sections in
      tool.cibuildwheel.
    """

    def __init__(
        self,
        config_file_path: Path | None = None,
        *,
        platform: PlatformName,
        env: Mapping[str, str],
        disallow: Mapping[str, Set[str]] | None = None,
    ) -> None:
        self.platform = platform
        self.env = env
        self.disallow = disallow or {}

        # Open defaults.toml, loading both global and platform sections
        defaults_path = resources_dir / "defaults.toml"
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/resources/defaults.toml" line="42">

---

# Platform-specific settings

Platform-specific settings can be specified under sections like `[tool.cibuildwheel.linux]`, `[tool.cibuildwheel.macos]`, or `[tool.cibuildwheel.windows]`. For example, the `repair-wheel-command` can be different for each platform.

```toml
[tool.cibuildwheel.linux]
repair-wheel-command = "auditwheel repair -w {dest_dir} {wheel}"

[tool.cibuildwheel.macos]
repair-wheel-command = "delocate-wheel --require-archs {delocate_archs} -w {dest_dir} -v {wheel}"

[tool.cibuildwheel.windows]

[tool.cibuildwheel.pyodide]
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/resources/cibuildwheel.schema.json" line="98">

---

# Schema for `tool.cibuildwheel`

The schema for `tool.cibuildwheel` is defined in `cibuildwheel.schema.json`. It specifies the allowed values for each setting and provides descriptions for them.

```json
      "default": "default",
      "description": "Set the tool to use to build, either \"pip\" (default for now), \"build\", or \"build[uv]\"",
      "oneOf": [
```

---

</SwmSnippet>

# Functionality of cibuildwheel

This section will cover the main functions of the cibuildwheel tool, focusing on how it builds wheels, computes platforms, handles options, and manages resources.

<SwmSnippet path="/cibuildwheel/__main__.py" line="287">

---

## Building Wheels

The `build_in_directory` function is the main function for building wheels. It takes command line arguments as input and performs a series of operations to build the wheels in a specified directory. It computes the platform, checks for errors, and sets up the environment for the build process. It also handles the output directory and checks for invalid configurations.

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

## Computing Platform

The `_compute_platform` function is used to determine the platform for which the wheels are being built. It takes command line arguments as input and returns the platform name. It checks for any conflicts between the platform and architecture specified in the arguments and raises an error if any are found.

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

<SwmSnippet path="/cibuildwheel/options.py" line="832">

---

## Handling Options

The `compute_options` function is used to compute the options for the build process. It takes the platform, command line arguments, and environment variables as input and returns an Options object. This object contains all the necessary information for the build process, such as the package directory, output directory, build selector, and test selector.

```python
def compute_options(
    platform: PlatformName,
    command_line_arguments: CommandLineArguments,
    env: Mapping[str, str],
) -> Options:
    options = Options(platform=platform, command_line_arguments=command_line_arguments, env=env)
    options.check_for_deprecated_options()
    return options
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/__main__.py" line="274">

---

## Managing Resources

The `get_platform_module` function is used to get the appropriate module for the given platform. It takes the platform name as input and returns the corresponding module. This function is essential for managing the resources required for the build process on different platforms.

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

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
