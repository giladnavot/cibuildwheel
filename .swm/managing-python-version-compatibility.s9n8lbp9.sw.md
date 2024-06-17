---
title: 'Managing Python Version Compatibility '
---
This document will cover the mechanism cibuildwheel uses to manage different Python versions. We'll cover:

1. The role of the 'tool' constant in managing Python versions.
2. How the 'build_in_directory' function contributes to Python version management.
3. The use of 'compute_options' and '\_compute_platform' functions in Python version management.

<SwmSnippet path="/cibuildwheel/resources/defaults.toml" line="1">

---

# The role of the 'tool' constant

The 'tool' constant in the 'defaults.toml' file is a part of the configuration for cibuildwheel. It includes settings like 'build', 'skip', 'test-skip', and 'free-threaded-support'. These settings are used to manage the building process across different Python versions.

```toml
[tool.cibuildwheel]
build = "*"
skip = ""
test-skip = ""
free-threaded-support = false
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/__main__.py" line="287">

---

# The 'build_in_directory' function

The 'build_in_directory' function in the '**main**.py' file is a key part of managing different Python versions. It computes the platform, computes options based on the platform and command line arguments, and then calls the appropriate platform module to build the wheel. This function ensures that the correct settings are used for each Python version.

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

<SwmSnippet path="/cibuildwheel/options.py" line="832">

---

# The 'compute_options' and '\_compute_platform' functions

The 'compute_options' function in the '[options.py](http://options.py)' file is used to compute the options for the build process based on the platform, command line arguments, and environment variables. This includes managing different Python versions. The '\_compute_platform' function in the '**main**.py' file is used to compute the platform based on command line arguments and environment variables. This function helps in determining the correct settings for each Python version.

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

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="follow-up"><sup>Powered by [Swimm](/)</sup></SwmMeta>
