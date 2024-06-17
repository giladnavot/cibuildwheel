---
title: Understanding Build Configuration
---
Build Configuration in cibuildwheel refers to the setup and customization of the build process for Python wheels across different platforms. It is primarily managed through the `BuildOptions` class, which contains various settings such as the build environment, build verbosity, test commands, and more. These options can be set through environment variables or a configuration file, allowing developers to tailor the build process to their specific needs. The configuration also includes platform-specific settings, such as the images used for manylinux and musllinux builds. Furthermore, the build configuration supports the concept of 'overrides', which allows for more granular control of the build process for specific wheel builds.

<SwmSnippet path="/cibuildwheel/options.py" line="82">

---

# BuildOptions Class

The `BuildOptions` class is where the build configuration is defined. It includes various attributes such as `globals`, `environment`, `before_all`, `before_build`, `repair_command`, `manylinux_images`, `musllinux_images`, `dependency_constraints`, `test_command`, `before_test`, `test_requires`, `test_extras`, `build_verbosity`, `build_frontend`, `config_settings`, and `container_engine`. Each of these attributes represents a specific aspect of the build configuration.

```python
class BuildOptions:
    globals: GlobalOptions
    environment: ParsedEnvironment
    before_all: str
    before_build: str | None
    repair_command: str
    manylinux_images: dict[str, str] | None
    musllinux_images: dict[str, str] | None
    dependency_constraints: DependencyConstraints | None
    test_command: str | None
    before_test: str | None
    test_requires: list[str]
    test_extras: str
    build_verbosity: int
    build_frontend: BuildFrontendConfig | None
    config_settings: str
    container_engine: OCIContainerEngineConfig

    @property
    def package_dir(self) -> Path:
        return self.globals.package_dir
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/options.py" line="563">

---

# build_options Function

The `build_options` function is used to compute the `BuildOptions` for a single run configuration. It reads the options from the environment, configuration file, or the default values, and returns a `BuildOptions` object.

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

<SwmSnippet path="/cibuildwheel/options.py" line="410">

---

# get Function

The `get` function is used within the `build_options` function to get and return the value for a named option from the environment, configuration file, or the default. It also handles the formatting of array and table values.

```python
    def get(
        self,
        name: str,
        *,
        env_plat: bool = True,
        list_sep: str | None = None,
        table_format: TableFmt | None = None,
        ignore_empty: bool = False,
    ) -> str:
        """
        Get and return the value for the named option from environment,
        configuration file, or the default. If env_plat is False, then don't
        accept platform versions of the environment variable. If this is an
        array it will be merged with "sep" before returning. If it is a table,
        it will be formatted with "table['item']" using {k} and {v} and merged
        with "table['sep']". If sep is also given, it will be used for arrays
        inside the table (must match table['sep']). Empty variables will not
        override if ignore_empty is True.
        """

        if name not in self.default_options and name not in self.default_platform_options:
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
