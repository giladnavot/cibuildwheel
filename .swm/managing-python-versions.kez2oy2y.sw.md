---
title: 'Managing Python Versions '
---
This document will cover how cibuildwheel handles different Python versions. We'll cover:

1. How Python versions are defined and updated
2. How Python versions are installed on different platforms
3. How Python versions are used in the build process.

<SwmSnippet path="/bin/update_pythons.py" line="298">

---

# Defining and Updating Python Versions

The <SwmToken path="/bin/update_pythons.py" pos="298:2:2" line-data="def update_pythons(force: bool, level: str) -&gt; None:">`update_pythons`</SwmToken> function is responsible for updating the Python versions. It uses the <SwmToken path="/bin/update_pythons.py" pos="239:2:2" line-data="class AllVersions:">`AllVersions`</SwmToken> class to update the configurations for each Python version. The configurations are then written to the <SwmPath>[cibuildwheel/resources/build-platforms.toml](/cibuildwheel/resources/build-platforms.toml)</SwmPath> file.

```python
def update_pythons(force: bool, level: str) -> None:
    logging.basicConfig(
        level="INFO",
        format="%(message)s",
        datefmt="[%X]",
        handlers=[RichHandler(rich_tracebacks=True, markup=True)],
    )
    log.setLevel(level)

    all_versions = AllVersions()
    toml_file_path = RESOURCES_DIR / "build-platforms.toml"

    original_toml = toml_file_path.read_text()
    with toml_file_path.open("rb") as f:
        configs = tomllib.load(f)

    for config in configs["windows"]["python_configurations"]:
        all_versions.update_config(config)

    for config in configs["macos"]["python_configurations"]:
        all_versions.update_config(config)
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/windows.py" line="336">

---

# Installing Python Versions on Windows

On Windows, the <SwmToken path="/cibuildwheel/windows.py" pos="336:2:2" line-data="def build(options: Options, tmp_path: Path) -&gt; None:">`build`</SwmToken> function calls <SwmToken path="/cibuildwheel/windows.py" pos="220:2:2" line-data="def setup_python(">`setup_python`</SwmToken> to install the required Python version. The <SwmToken path="/cibuildwheel/windows.py" pos="220:2:2" line-data="def setup_python(">`setup_python`</SwmToken> function uses classes like <SwmToken path="/bin/update_pythons.py" pos="60:2:2" line-data="class WindowsVersions:">`WindowsVersions`</SwmToken>, <SwmToken path="/bin/update_pythons.py" pos="109:2:2" line-data="class PyPyVersions:">`PyPyVersions`</SwmToken>, and <SwmToken path="/bin/update_pythons.py" pos="188:2:2" line-data="class CPythonVersions:">`CPythonVersions`</SwmToken> to handle different Python versions and architectures.

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

<SwmSnippet path="/cibuildwheel/macos.py" line="198">

---

# Installing Python Versions on MacOS

On MacOS, the `build` function also calls <SwmToken path="/cibuildwheel/macos.py" pos="198:2:2" line-data="def setup_python(">`setup_python`</SwmToken> to install the required Python version. The <SwmToken path="/cibuildwheel/macos.py" pos="198:2:2" line-data="def setup_python(">`setup_python`</SwmToken> function uses <SwmToken path="/cibuildwheel/macos.py" pos="143:2:2" line-data="def install_cpython(tmp: Path, version: str, url: str, free_threading: bool) -&gt; Path:">`install_cpython`</SwmToken> and <SwmToken path="/cibuildwheel/macos.py" pos="183:2:2" line-data="def install_pypy(tmp: Path, url: str) -&gt; Path:">`install_pypy`</SwmToken> to handle different Python versions.

```python
def setup_python(
    tmp: Path,
    python_configuration: PythonConfiguration,
    dependency_constraint_flags: Sequence[PathOrStr],
    environment: ParsedEnvironment,
    build_frontend: BuildFrontendName,
) -> tuple[Path, dict[str, str]]:
    if build_frontend == "build[uv]" and Version(python_configuration.version) < Version("3.8"):
        build_frontend = "build"

    uv_path = find_uv()
    use_uv = build_frontend == "build[uv]" and Version(python_configuration.version) >= Version(
        "3.8"
    )

    tmp.mkdir()
    implementation_id = python_configuration.identifier.split("-")[0]
    log.step(f"Installing Python {implementation_id}...")
    if implementation_id.startswith("cp"):
        free_threading = "t-macos" in python_configuration.identifier
        base_python = install_cpython(
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/windows.py" line="336">

---

# Using Python Versions in the Build Process

During the build process, the installed Python version is used to set up a virtual environment. The `setup_python` function returns the path to the Python executable and a dictionary of environment variables to be used in the build process.

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

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
