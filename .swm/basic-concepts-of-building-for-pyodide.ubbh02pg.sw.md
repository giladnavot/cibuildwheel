---
title: Basic Concepts of Building for Pyodide
---
In the cibuildwheel project, 'Build For Pyodide' refers to the process of building Python wheels for the Pyodide platform. Pyodide is a Python interpreter compiled to WebAssembly, allowing Python to be run in the browser.&nbsp;

The build process involves setting up a Python environment, installing necessary build tools, and executing the build command.&nbsp;

The build process is handled by the <SwmToken path="/cibuildwheel/pyodide.py" pos="192:2:2" line-data="def build(options: Options, tmp_path: Path) -&gt; None:">`build`</SwmToken> function in the <SwmPath>[cibuildwheel/pyodide.py](/cibuildwheel/pyodide.py)</SwmPath> file. This function sets up the Python environment, installs necessary build tools, and executes the build command. It uses various helper functions and classes such as <SwmToken path="/cibuildwheel/pyodide.py" pos="181:2:2" line-data="def get_python_configurations(">`get_python_configurations`</SwmToken>, <SwmToken path="/cibuildwheel/pyodide.py" pos="110:2:2" line-data="def setup_python(">`setup_python`</SwmToken>, and <SwmToken path="/cibuildwheel/pyodide.py" pos="42:2:2" line-data="class PythonConfiguration:">`PythonConfiguration`</SwmToken> to accomplish this.

<SwmSnippet path="/cibuildwheel/pyodide.py" line="192">

---

# Build Function

The <SwmToken path="/cibuildwheel/pyodide.py" pos="192:2:2" line-data="def build(options: Options, tmp_path: Path) -&gt; None:">`build`</SwmToken> function orchestrates the entire process of building wheels for Pyodide. It first retrieves the Python configurations to be used for the build process. Then, for each configuration, it sets up the Python environment, installs necessary tools, and executes the build commands. If the build is successful, the resulting wheel is repaired and tested. The function also handles various error conditions and ensures that the build process is properly cleaned up after completion.

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
                before_all_options.before_all, project=".", package=before_all_options.package_dir
            )
            shell(before_all_prepared, env=env)

        built_wheels: list[Path] = []
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/pyodide.py" line="181">

---

# Python Configurations

The <SwmToken path="/cibuildwheel/pyodide.py" pos="181:2:2" line-data="def get_python_configurations(">`get_python_configurations`</SwmToken> function is used to retrieve the Python configurations to be used for the build process. It takes a <SwmToken path="/cibuildwheel/util.py" pos="239:2:2" line-data="class BuildSelector:">`BuildSelector`</SwmToken> object and a set of architectures as arguments, and returns a list of <SwmToken path="/cibuildwheel/pyodide.py" pos="42:2:2" line-data="class PythonConfiguration:">`PythonConfiguration`</SwmToken> objects. Each <SwmToken path="/cibuildwheel/pyodide.py" pos="42:2:2" line-data="class PythonConfiguration:">`PythonConfiguration`</SwmToken> object represents a specific Python version and platform to build for.

```python
def get_python_configurations(
    build_selector: BuildSelector,
    architectures: Set[Architecture],  # noqa: ARG001
) -> list[PythonConfiguration]:
    full_python_configs = read_python_configs("pyodide")

    python_configurations = [PythonConfiguration(**item) for item in full_python_configs]
    python_configurations = [c for c in python_configurations if build_selector(c.identifier)]
    return python_configurations
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/pyodide.py" line="110">

---

# Environment Setup

The <SwmToken path="/cibuildwheel/pyodide.py" pos="110:2:2" line-data="def setup_python(">`setup_python`</SwmToken> function is used to set up the Python environment for a specific Python configuration. It installs the necessary Python version, sets up a virtual environment, and installs necessary tools such as emscripten and Pyodide xbuildenv. The function returns a dictionary representing the environment variables to be used for the build process.

```python
def setup_python(
    tmp: Path,
    python_configuration: PythonConfiguration,
    dependency_constraint_flags: Sequence[PathOrStr],
    environment: ParsedEnvironment,
) -> dict[str, str]:
    base_python = get_base_python(python_configuration.identifier)

    log.step("Setting up build environment...")
    venv_path = tmp / "venv"
    env = virtualenv(python_configuration.version, base_python, venv_path, [], use_uv=False)
    venv_bin_path = venv_path / "bin"
    assert venv_bin_path.exists()
    env["PIP_DISABLE_PIP_VERSION_CHECK"] = "1"

    # upgrade pip to the version matching our constraints
    # if necessary, reinstall it to ensure that it's available on PATH as 'pip'
    call(
        "python",
        "-m",
        "pip",
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/pyodide.py" line="70">

---

# Pyodide xbuildenv Installation

The <SwmToken path="/cibuildwheel/pyodide.py" pos="70:2:2" line-data="def install_xbuildenv(env: dict[str, str], pyodide_version: str) -&gt; str:">`install_xbuildenv`</SwmToken> function is used to install the Pyodide xbuildenv for a specific Pyodide version. It downloads and extracts the xbuildenv from the Pyodide GitHub repository, and sets the <SwmToken path="/cibuildwheel/pyodide.py" pos="82:6:6" line-data="        env.pop(&quot;PYODIDE_ROOT&quot;, None)">`PYODIDE_ROOT`</SwmToken> environment variable to the installation path. The function returns the path to the installed xbuildenv.

```python
def install_xbuildenv(env: dict[str, str], pyodide_version: str) -> str:
    pyodide_root = (
        CIBW_CACHE_PATH
        / f".pyodide-xbuildenv-{pyodide_version}/{pyodide_version}/xbuildenv/pyodide-root"
    )
    with FileLock(CIBW_CACHE_PATH / "xbuildenv.lock"):
        if pyodide_root.exists():
            return str(pyodide_root)

        # We don't want to mutate env but we need to delete any existing
        # PYODIDE_ROOT so copy it first.
        env = dict(env)
        env.pop("PYODIDE_ROOT", None)
        call(
            "pyodide",
            "xbuildenv",
            "install",
            pyodide_version,
            env=env,
            cwd=CIBW_CACHE_PATH,
        )
```

---

</SwmSnippet>

# 

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
