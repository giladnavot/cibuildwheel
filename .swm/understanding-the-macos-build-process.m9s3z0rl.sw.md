---
title: Understanding the MacOS Build Process
---
Building for MacOS in cibuildwheel involves a series of steps to ensure compatibility across different Python versions and architectures. The process starts with the identification of the Python configurations that match the selected architectures. This is followed by setting up the Python environment, which includes installing the appropriate Python version and setting up the build environment. The build process also takes into consideration the MacOS version and adjusts the deployment target accordingly. If necessary, the process also involves cross-compilation configurations. The build tools are then installed and the wheel is built. If a wheel compatible with the current configuration has been built previously, the build step is skipped. The built wheel is then repaired if necessary, and finally tested.

<SwmSnippet path="/cibuildwheel/macos.py" line="394">

---

# Build Function

The `build` function is the main entry point for the MacOS build process. It first retrieves the Python configurations using the `get_python_configurations` function. Then, it sets up the build environment and executes the build process for each Python configuration. If any error occurs during the build process, it raises a `FatalError`.

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
            env.setdefault("MACOSX_DEPLOYMENT_TARGET", "10.9")
            before_all_prepared = prepare_command(
                before_all_options.before_all, project=".", package=before_all_options.package_dir
            )
            shell(before_all_prepared, env=env)

```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/macos.py" line="101">

---

# get_python_configurations Function

The `get_python_configurations` function is used to retrieve the Python configurations for the build process. It filters out configurations that don't match the selected architectures and those that are not required by the build selector.

```python
def get_python_configurations(
    build_selector: BuildSelector, architectures: Set[Architecture]
) -> list[PythonConfiguration]:
    full_python_configs = read_python_configs("macos")

    python_configurations = [PythonConfiguration(**item) for item in full_python_configs]

    # filter out configs that don't match any of the selected architectures
    python_configurations = [
        c
        for c in python_configurations
        if any(c.identifier.endswith(a.value) for a in architectures)
    ]

    # skip builds as required by BUILD/SKIP
    python_configurations = [c for c in python_configurations if build_selector(c.identifier)]

    # filter-out some cross-compilation configs with PyPy:
    # can't build arm64 on x86_64
    # rosetta allows to build x86_64 on arm64
    if platform.machine() == "x86_64":
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/macos.py" line="198">

---

# setup_python Function

The `setup_python` function sets up the Python environment for the build process. It installs the necessary Python version, sets up the virtual environment, and configures the environment variables for the build process.

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

<SwmSnippet path="/cibuildwheel/macos.py" line="89">

---

# get_macos_sdks Function

The `get_macos_sdks` function is used to retrieve the MacOS SDKs available for the build process. It calls the `xcodebuild -showsdks` command and parses the output to get the list of SDKs.

```python
def get_macos_sdks() -> list[str]:
    output = call("xcodebuild", "-showsdks", capture_stdout=True)
    return [m.group(1) for m in re.finditer(r"-sdk (macosx\S+)", output)]
```

---

</SwmSnippet>

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel"><sup>Powered by [Swimm](https://staging.swimm.cloud/)</sup></SwmMeta>
