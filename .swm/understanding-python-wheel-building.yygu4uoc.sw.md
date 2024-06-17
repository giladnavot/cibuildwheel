---
title: Understanding Python Wheel Building
---
Python Wheel Building refers to the process of creating a wheel distribution package for Python applications. A wheel distribution package is a built distribution format that can be installed without needing to go through the 'build' process. In the cibuildwheel project, this process is automated across multiple platforms and Python versions. The project provides a variety of configuration options to customize the wheel building process, such as setting environment variables, specifying build options, and running custom commands before or after the build process.

<SwmSnippet path="/cibuildwheel/linux.py" line="154">

---

# Building Wheel Files

This is the `build_in_container` function, which is responsible for building wheel files in a container. It takes in various parameters like the options for the build, the Python configurations, and the paths for the project and package directories. It sets up the environment, runs the `before_all` command if specified, and then iterates over the Python configurations to build the wheel files.

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

<SwmSnippet path="/cibuildwheel/linux.py" line="125">

---

# Checking Python Existence

The `check_all_python_exist` function is used to ensure that all the required Python versions exist in the container. It iterates over the Python configurations and checks if the Python executable exists for each configuration.

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

The `copy_into` function is used to copy the project into the container. This is necessary to ensure that the project files are available within the container for the build process.

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

<SwmSnippet path="/cibuildwheel/options.py" line="563">

---

# Building Options

The `build_options` function is used to compute the build options for a single run configuration. It reads various options like the `before-all` command, environment variables, `before-build` command, and others from the configuration file.

```python
    def build_options(self, identifier: str | None) -> BuildOptions:
        """
        Compute BuildOptions for a single run configuration.
        """

        with self.reader.identifier(identifier):
            before_all = self.reader.get("before-all", list_sep=" && ")

            environment_config = self.reader.get(
                "environment", table_format={"item": '{k}="{v}"', "sep": " "}
            )
            environment_pass = self.reader.get("environment-pass", list_sep=" ").split()
            before_build = self.reader.get("before-build", list_sep=" && ")
            repair_command = self.reader.get("repair-wheel-command", list_sep=" && ")
            config_settings = self.reader.get(
                "config-settings",
                table_format={"item": "{k}={v}", "sep": " ", "quote": shlex.quote},
            )

            dependency_versions = self.reader.get("dependency-versions")
            test_command = self.reader.get("test-command", list_sep=" && ")
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/environment.py" line="114">

---

# Environment Setup

The `as_dictionary` function is used to convert the environment variables into a dictionary. This is necessary for setting up the environment for the build process.

```python
    def as_dictionary(
        self,
        prev_environment: Mapping[str, str],
        executor: bashlex_eval.EnvironmentExecutor | None = None,
    ) -> dict[str, str]:
        environment = {**prev_environment}

        for assignment in self.assignments:
            value = assignment.evaluated_value(environment=environment, executor=executor)
            environment[assignment.name] = value

        return environment
```

---

</SwmSnippet>

# Python Wheel Building Endpoints

Understanding cibuildwheel's core functions

<SwmSnippet path="/cibuildwheel/schema.py" line="10">

---

## get_schema

The `get_schema` function is used to get the complete schema for cibuildwheel settings. It reads a JSON schema from a file and returns it as a dictionary. This schema is used to validate and guide the configuration of cibuildwheel.

```python
def get_schema(tool_name: str = "cibuildwheel") -> dict[str, Any]:
    "Get the stored complete schema for cibuildwheel settings."
    assert tool_name == "cibuildwheel", "Only cibuildwheel is supported."

    with DIR.joinpath("resources/cibuildwheel.schema.json").open(encoding="utf-8") as f:
        return json.load(f)  # type: ignore[no-any-return]
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/windows.py" line="336">

---

## build

The `build` function is the main entry point for building wheels. It takes in options and a temporary path, and orchestrates the process of building wheels for the specified Python configurations.

```python
def build(options: Options, tmp_path: Path) -> None:
    python_configurations = get_python_configurations(
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
