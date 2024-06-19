---
title: how to build inside a container?
---
Building inside a container in cibuildwheel involves using the <SwmToken path="cibuildwheel/linux.py" pos="17:7:7" line-data="from .oci_container import OCIContainer, OCIContainerEngineConfig" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`OCIContainer`</SwmToken> class and specifying the container engine to use. The <SwmToken path="cibuildwheel/linux.py" pos="17:7:7" line-data="from .oci_container import OCIContainer, OCIContainerEngineConfig" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`OCIContainer`</SwmToken> class is designed to manage the lifecycle of a container, and the container engine can be specified using the CIBW_CONTAINER_ENGINE option.

<SwmPath repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" path="docs/options.md">`(cibuildwheel) docs/options.md`</SwmPath>:\
The `CIBW_CONTAINER_ENGINE` option allows you to specify the container engine to use when building Linux wheels. This could be either 'docker' or 'podman'.

<SwmSnippet path="/cibuildwheel/oci_container.py" line="32" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==">

---

The <SwmToken path="cibuildwheel/oci_container.py" pos="33:2:2" line-data="class OCIContainerEngineConfig:" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`OCIContainerEngineConfig`</SwmToken> class is used to configure the container engine. It takes the name of the engine ('docker' or 'podman'), any additional arguments for the 'create' command, and a flag to disable host mounting.

```python
@dataclass(frozen=True)
class OCIContainerEngineConfig:
    name: ContainerEngineName
    create_args: tuple[str, ...] = field(default_factory=tuple)
    disable_host_mount: bool = False

    @staticmethod
    def from_config_string(config_string: str) -> OCIContainerEngineConfig:
        config_dict = parse_key_value_string(
            config_string,
            ["name"],
            ["create_args", "create-args", "disable_host_mount", "disable-host-mount"],
        )
        name = " ".join(config_dict["name"])
        if name not in {"docker", "podman"}:
            msg = f"unknown container engine {name}"
            raise ValueError(msg)

        name = typing.cast(ContainerEngineName, name)
        # some flexibility in the option names to cope with TOML conventions
        create_args = config_dict.get("create_args") or config_dict.get("create-args") or []
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/oci_container.py" line="78" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==">

---

The <SwmToken path="cibuildwheel/oci_container.py" pos="78:2:2" line-data="class OCIContainer:" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`OCIContainer`</SwmToken> class manages the lifecycle of a container. It starts the container with the specified image and engine configuration, runs commands inside the container, and stops the container when done.

```python
class OCIContainer:
    """
    An object that represents a running OCI (e.g. Docker) container.

    Intended for use as a context manager e.g.
    `with OCIContainer(image = 'ubuntu') as docker:`

    A bash shell is running in the remote container. When `call()` is invoked,
    the command is relayed to the remote shell, and the results are streamed
    back to cibuildwheel.

    Example:
        >>> from cibuildwheel.oci_container import *  # NOQA
        >>> from cibuildwheel.options import _get_pinned_container_images
        >>> image = _get_pinned_container_images()['x86_64']['manylinux2014']
        >>> # Test the default container
        >>> with OCIContainer(image=image) as self:
        ...     self.call(["echo", "hello world"])
        ...     self.call(["cat", "/proc/1/cgroup"])
        ...     print(self.get_environment())
        ...     print(self.debug_info())
```

---

</SwmSnippet>

<SwmMeta version="3.0.0"><sup>Powered by [Swimm](/)</sup></SwmMeta>
