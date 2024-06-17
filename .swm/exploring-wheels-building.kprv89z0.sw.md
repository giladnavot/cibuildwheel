---
title: Exploring Wheels Building
---
Wheels Building refers to the process of creating a wheel, which is a built-package format for Python. A wheel is a ZIP-format archive with a specially formatted filename and the .whl extension. It contains all the files necessary for the installation of a Python app. In cibuildwheel, wheels are built for different Python versions and platforms. The configurations for these builds are specified in the 'build-platforms.toml' file. The actual building process is handled by the 'build' function in the respective platform-specific files ([macos.py](http://macos.py), [windows.py](http://windows.py), [linux.py](http://linux.py), [pyodide.py](http://pyodide.py)). The build process involves setting up the build environment, running pre-build commands, building the wheel, and repairing the wheel if necessary.

<SwmSnippet path="/cibuildwheel/resources/build-platforms.toml" line="1">

---

# Building Wheels in Different Environments

This file contains the configurations for different Python versions and platforms. These configurations are used when building wheels.

```toml
[linux]
python_configurations = [
  { identifier = "cp36-manylinux_x86_64", version = "3.6", path_str = "/opt/python/cp36-cp36m" },
  { identifier = "cp37-manylinux_x86_64", version = "3.7", path_str = "/opt/python/cp37-cp37m" },
  { identifier = "cp38-manylinux_x86_64", version = "3.8", path_str = "/opt/python/cp38-cp38" },
  { identifier = "cp39-manylinux_x86_64", version = "3.9", path_str = "/opt/python/cp39-cp39" },
  { identifier = "cp310-manylinux_x86_64", version = "3.10", path_str = "/opt/python/cp310-cp310" },
  { identifier = "cp311-manylinux_x86_64", version = "3.11", path_str = "/opt/python/cp311-cp311" },
  { identifier = "cp312-manylinux_x86_64", version = "3.12", path_str = "/opt/python/cp312-cp312" },
  { identifier = "cp313-manylinux_x86_64", version = "3.13", path_str = "/opt/python/cp313-cp313" },
  { identifier = "cp313t-manylinux_x86_64", version = "3.13", path_str = "/opt/python/cp313-cp313t" },
  { identifier = "cp36-manylinux_i686", version = "3.6", path_str = "/opt/python/cp36-cp36m" },
  { identifier = "cp37-manylinux_i686", version = "3.7", path_str = "/opt/python/cp37-cp37m" },
  { identifier = "cp38-manylinux_i686", version = "3.8", path_str = "/opt/python/cp38-cp38" },
  { identifier = "cp39-manylinux_i686", version = "3.9", path_str = "/opt/python/cp39-cp39" },
  { identifier = "cp310-manylinux_i686", version = "3.10", path_str = "/opt/python/cp310-cp310" },
  { identifier = "cp311-manylinux_i686", version = "3.11", path_str = "/opt/python/cp311-cp311" },
  { identifier = "cp312-manylinux_i686", version = "3.12", path_str = "/opt/python/cp312-cp312" },
  { identifier = "cp313-manylinux_i686", version = "3.13", path_str = "/opt/python/cp313-cp313" },
  { identifier = "cp313t-manylinux_i686", version = "3.13", path_str = "/opt/python/cp313-cp313t" },
  { identifier = "pp37-manylinux_x86_64", version = "3.7", path_str = "/opt/python/pp37-pypy37_pp73" },
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/linux.py" line="154">

---

# Building Wheels in a Container

The `build_in_container` function is used to build wheels in a containerized environment. It takes in various parameters like options, platform configurations, and container details, and uses them to set up the environment and build the wheels.

```python
def build_in_container(
    *,
    options: Options,
    platform_configs: Sequence[PythonConfiguration],
    container: OCIContainer,
    container_project_path: PurePath,
    container_package_dir: PurePath,
) -> None:
    container_output_dir = PurePosixPath("/output")
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/linux.py" line="125">

---

# Checking Python Existence in Container

The `check_all_python_exist` function is used to ensure that all required Python versions exist in the container before the wheel building process starts.

```python
def check_all_python_exist(
    *, platform_configs: Iterable[PythonConfiguration], container: OCIContainer
) -> None:
    exist = True
    has_manylinux_interpreters = True
    messages = []

    try:
        # use capture_output to keep quiet
        container.call(["manylinux-interpreters", "--help"], capture_output=True)
    except subprocess.CalledProcessError:
        has_manylinux_interpreters = False

    for config in platform_configs:
        python_path = config.path / "bin" / "python"
        try:
            if has_manylinux_interpreters:
                container.call(["manylinux-interpreters", "ensure", config.path.name])
            container.call(["test", "-x", python_path])
        except subprocess.CalledProcessError:
            messages.append(
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/oci_container.py" line="223">

---

# Copying Project into Container

The `copy_into` function is used to copy the project files into the container. This is necessary to make the project files available for the wheel building process inside the container.

```python
    def copy_into(self, from_path: Path, to_path: PurePath) -> None:
        # `docker cp` causes 'no space left on device' error when
        # a container is running and the host filesystem is
        # mounted. https://github.com/moby/moby/issues/38995
        # Use `docker exec` instead.

        if from_path.is_dir():
            self.call(["mkdir", "-p", to_path])
            subprocess.run(
                f"tar cf - . | {self.engine.name} exec -i {self.name} tar --no-same-owner -xC {shell_quote(to_path)} -f -",
                shell=True,
                check=True,
                cwd=from_path,
            )
        else:
            exec_process: subprocess.Popen[bytes]
            with subprocess.Popen(
                [
                    self.engine.name,
                    "exec",
                    "-i",
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/windows.py" line="336">

---

# Building Wheels on Different Platforms

The `build` function in `windows.py` is an example of how wheels are built for a specific platform. Similar functions exist in `linux.py`, `macos.py`, and `pyodide.py` for their respective platforms.

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

# 

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel"><sup>Powered by [Swimm](https://staging.swimm.cloud/)</sup></SwmMeta>
