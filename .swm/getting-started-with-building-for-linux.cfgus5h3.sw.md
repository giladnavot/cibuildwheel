---
title: Getting Started with Building for Linux
---
Building for Linux in cibuildwheel involves several steps. It starts with defining a `BuildStep` class that contains configurations for the platform, container engine, and container image. The `get_build_steps` function groups Python configurations into `BuildStep` instances, each representing a separate container instance. The `build_in_container` function is then used to execute the build process in a container. It checks if all Python configurations exist, copies the project into the container, sets up the build environment, and builds the wheel. If the build is successful, the built wheel is then repaired, tested, and copied back to the host.

<SwmSnippet path="/cibuildwheel/linux.py" line="47">

---

# BuildStep Class

The `BuildStep` class represents a single build step, which includes a list of Python configurations (`platform_configs`), a platform tag (`platform_tag`), a container engine configuration (`container_engine`), and a container image (`container_image`). Each instance of this class represents a separate container instance for the build process.

```python
class BuildStep:
    platform_configs: list[PythonConfiguration]
    platform_tag: str
    container_engine: OCIContainerEngineConfig
    container_image: str
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/linux.py" line="92">

---

# get_build_steps Function

The `get_build_steps` function groups Python configurations into build steps. It iterates over the Python configurations and for each configuration, it creates a `BuildStep` instance if it doesn't exist already for the given platform tag, container image, and container engine. The function returns an iterator over the build steps.

```python
def get_build_steps(
    options: Options, python_configurations: list[PythonConfiguration]
) -> Iterator[BuildStep]:
    """
    Groups PythonConfigurations into BuildSteps. Each BuildStep represents a
    separate container instance.
    """
    steps = OrderedDict[Tuple[str, str, str, OCIContainerEngineConfig], BuildStep]()

    for config in python_configurations:
        _, platform_tag = config.identifier.split("-", 1)

        build_options = options.build_options(config.identifier)

        before_all = build_options.before_all
        container_image = container_image_for_python_configuration(config, build_options)
        container_engine = build_options.container_engine

        step_key = (platform_tag, container_image, before_all, container_engine)

        if step_key in steps:
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/linux.py" line="154">

---

# build_in_container Function

The `build_in_container` function is responsible for building and testing the wheels in a Linux container. It first checks if all Python executables exist in the container, then copies the project into the container, and prepares the build environment. It then iterates over the platform configurations and for each configuration, it builds the wheel, repairs it if necessary, and tests it. Finally, it copies the built wheels back to the host.

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

    check_all_python_exist(platform_configs=platform_configs, container=container)

    log.step("Copying project into container...")
    container.copy_into(Path.cwd(), container_project_path)

    before_all_options_identifier = platform_configs[0].identifier
    before_all_options = options.build_options(before_all_options_identifier)

    if before_all_options.before_all:
        log.step("Running before_all...")

```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/linux.py" line="415">

---

# build Function

The `build` function is the entry point for the build process. It first gets the Python configurations and then iterates over the build steps returned by `get_build_steps`. For each build step, it creates a Linux container and calls `build_in_container` to build and test the wheels in the container.

```python
def build(options: Options, tmp_path: Path) -> None:  # noqa: ARG001
    python_configurations = get_python_configurations(
        options.globals.build_selector, options.globals.architectures
    )

    cwd = Path.cwd()
    abs_package_dir = options.globals.package_dir.resolve()
    if cwd != abs_package_dir and cwd not in abs_package_dir.parents:
        msg = "package_dir must be inside the working directory"
        raise Exception(msg)

    container_project_path = PurePosixPath("/project")
    container_package_dir = container_project_path / abs_package_dir.relative_to(cwd)

    for build_step in get_build_steps(options, python_configurations):
        try:
            # check the container engine is installed
            subprocess.run(
                [build_step.container_engine.name, "--version"],
                check=True,
                stdout=subprocess.DEVNULL,
```

---

</SwmSnippet>

# 

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel"><sup>Powered by [Swimm](https://staging.swimm.cloud/)</sup></SwmMeta>
