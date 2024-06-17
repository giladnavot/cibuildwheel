---
title: Understanding Wheel Building
---
Wheel Building in cibuildwheel refers to the process of compiling Python source code into a binary wheel distribution. This process is handled by the `build` function in the `windows.py`, `macos.py`, and `linux.py` files. The `build` function sets up the build environment, installs the necessary Python versions, and runs the build commands. It also handles the execution of any pre-build or post-build scripts defined in the project's configuration. If a wheel has already been built for a specific Python configuration, the function will skip the build process for that configuration. This process is crucial for creating distributable Python packages that can be easily installed on different systems.

<SwmSnippet path="/cibuildwheel/windows.py" line="336">

---

# Building Wheels

The `build` function in `cibuildwheel/windows.py` is the main function for building wheels. It first gets the Python configurations and build options, then it runs the `before_all` command if it exists. After that, it loops over each Python configuration, sets up the Python environment, and builds the wheel using either pip or build. If a wheel has already been built for a configuration, it skips the build step for that configuration.

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

<SwmSnippet path="/cibuildwheel/macos.py" line="394">

---

The `build` function in `cibuildwheel/macos.py` is similar to the one in `cibuildwheel/windows.py`, but it includes additional steps for handling macOS-specific configurations and dependencies.

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

<SwmSnippet path="/cibuildwheel/linux.py" line="154">

---

The `build_in_container` function in `cibuildwheel/linux.py` is used for building wheels in a Linux container. It first checks if all Python versions exist in the container, then it copies the project into the container and runs the `before_all` command if it exists. After that, it loops over each Python configuration, sets up the Python environment, and builds the wheel using either pip or build. If a wheel has already been built for a configuration, it skips the build step for that configuration.

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

<SwmSnippet path="/cibuildwheel/linux.py" line="483">

---

# Troubleshooting Wheel Building

The `troubleshoot` function in `cibuildwheel/linux.py` is used to identify and resolve issues that occur during the wheel building process. It checks if the error occurred during the wheel build step or the repair step, and provides helpful messages for common issues, such as shared object (.so) files being built against the wrong OS.

```python
def troubleshoot(options: Options, error: Exception) -> None:
    if isinstance(error, subprocess.CalledProcessError) and (
        error.cmd[0:4] == ["python", "-m", "pip", "wheel"]
        or error.cmd[0:3] == ["python", "-m", "build"]
        or _matches_prepared_command(
            error.cmd, options.build_options(None).repair_command
        )  # TODO allow matching of overrides too?
    ):
        # the wheel build step or the repair step failed
        so_files = list(options.globals.package_dir.glob("**/*.so"))

        if so_files:
            print(
                textwrap.dedent(
                    """
                    NOTE: Shared object (.so) files found in this project.

                    These files might be built against the wrong OS, causing problems with
                    auditwheel. If possible, run cibuildwheel in a clean checkout.

                    If you're using Cython and have previously done an in-place build,
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
