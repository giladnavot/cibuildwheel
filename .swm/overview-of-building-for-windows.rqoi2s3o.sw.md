---
title: Overview of Building for Windows
---
Building for Windows in cibuildwheel involves a series of steps to prepare, compile, and package Python wheels. It starts with the `build` function, which retrieves Python configurations for the build using the `get_python_configurations` function. These configurations are based on the build selector and architectures provided.

The `build` function also sets up the build environment for each Python configuration. This includes installing the appropriate Python version, setting up virtual environments, and installing build tools. The `setup_python` function is used for this purpose.

For cross-compilation, additional setup is required. The `setup_setuptools_cross_compile` function is used to set up the environment for setuptools, and the `setup_rust_cross_compile` function is used for Rust. These functions configure the environment variables and settings necessary for cross-compilation.

Once the environment is set up, the wheel is built using the appropriate build frontend (pip or build). If the wheel is successfully built, it is added to a list of built wheels. If any step in the build process fails, an error is raised and the build process is halted.

<SwmSnippet path="/cibuildwheel/windows.py" line="336">

---

# Build Function

The `build` function is the main entry point for building wheels for Windows. It takes in options and a temporary path as arguments. The function first retrieves the Python configurations for the build using `get_python_configurations`. It then sets up the environment and installs Python using `setup_python`. Finally, it sets up the build tools and performs the build.

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

<SwmSnippet path="/cibuildwheel/windows.py" line="81">

---

# Get Python Configurations

`get_python_configurations` is used to retrieve the Python configurations for the build. It takes a `BuildSelector` and a set of architectures as arguments and returns a list of `PythonConfiguration` objects.

```python
def get_python_configurations(
    build_selector: BuildSelector,
    architectures: Set[Architecture],
) -> list[PythonConfiguration]:
    full_python_configs = read_python_configs("windows")

    python_configurations = [PythonConfiguration(**item) for item in full_python_configs]

    map_arch = {"32": Architecture.x86, "64": Architecture.AMD64, "ARM64": Architecture.ARM64}

    # skip builds as required
    python_configurations = [
        c
        for c in python_configurations
        if build_selector(c.identifier) and map_arch[c.arch] in architectures
    ]

    return python_configurations
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/windows.py" line="220">

---

# Setup Python

`setup_python` is used to set up the Python environment for the build. It installs the appropriate version of Python and sets up the environment variables needed for the build.

```python
def setup_python(
    tmp: Path,
    python_configuration: PythonConfiguration,
    dependency_constraint_flags: Sequence[PathOrStr],
    environment: ParsedEnvironment,
    build_frontend: BuildFrontendName,
) -> tuple[Path, dict[str, str]]:
    tmp.mkdir()
    implementation_id = python_configuration.identifier.split("-")[0]
    python_libs_base = None
    log.step(f"Installing Python {implementation_id}...")
    if implementation_id.startswith("cp"):
        native_arch = platform_module.machine()
        base_python = install_cpython(python_configuration)
        if python_configuration.arch == "ARM64" != native_arch:
            # To cross-compile for ARM64, we need a native CPython to run the
            # build, and a copy of the ARM64 import libraries ('.\libs\*.lib')
            # for any extension modules.
            python_libs_base = base_python.parent / "libs"
            log.step(f"Installing native Python {native_arch} for cross-compilation...")
            base_python = install_cpython(python_configuration, arch=native_arch)
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/windows.py" line="143">

---

# Setup Setuptools Cross Compile

`setup_setuptools_cross_compile` is used to set up the environment for cross-compilation. It configures the platform name and library directories for the build, and sets environment variables needed for cross-compilation.

```python
def setup_setuptools_cross_compile(
    tmp: Path,
    python_configuration: PythonConfiguration,
    python_libs_base: Path,
    env: MutableMapping[str, str],
) -> None:
    distutils_cfg = tmp / "extra-setup.cfg"
    env["DIST_EXTRA_CONFIG"] = str(distutils_cfg)
    log.notice(f"Setting DIST_EXTRA_CONFIG={distutils_cfg} for cross-compilation")

    # Ensure our additional import libraries are made available, and explicitly
    # set the platform name
    map_plat = {"32": "win32", "64": "win-amd64", "ARM64": "win-arm64"}
    plat_name = map_plat[python_configuration.arch]
    # (This file must be default/locale encoding, so we can't pass 'encoding')
    distutils_cfg.write_text(
        textwrap.dedent(
            f"""\
            [build]
            plat_name={plat_name}
            [build_ext]
```

---

</SwmSnippet>

# 

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel"><sup>Powered by [Swimm](https://staging.swimm.cloud/)</sup></SwmMeta>
