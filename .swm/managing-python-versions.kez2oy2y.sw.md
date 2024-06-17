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

The `update_pythons` function is responsible for updating the Python versions. It uses the `AllVersions` class to update the configurations for each Python version. The configurations are then written to the `build-platforms.toml` file.

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

On Windows, the `build` function calls `setup_python` to install the required Python version. The `setup_python` function uses classes like `WindowsVersions`, `PyPyVersions`, and `CPythonVersions` to handle different Python versions and architectures.

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

On MacOS, the `build` function also calls `setup_python` to install the required Python version. The `setup_python` function uses `install_cpython` and `install_pypy` to handle different Python versions.

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

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="follow-up"><sup>Powered by [Swimm](/)</sup></SwmMeta>
