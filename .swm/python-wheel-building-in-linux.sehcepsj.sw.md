---
title: Python Wheel Building in Linux
---
This document will cover the process of building Python wheels in a Linux environment using cibuildwheel. The process includes:

1. Troubleshooting build options
2. Building in a container
3. Copying into the OCI container
4. Writing to the OCI container.

```mermaid
graph TD;
  build:::mainFlowStyle --> troubleshoot
  build:::mainFlowStyle --> build_in_container
  troubleshoot --> build_options
  build_in_container:::mainFlowStyle --> build_options
  build_in_container:::mainFlowStyle --> copy_into
  copy_into:::mainFlowStyle --> call
  call:::mainFlowStyle --> write
  write:::mainFlowStyle --> ...

 classDef mainFlowStyle color:#000000,fill:#7CB9F4
  classDef rootsStyle color:#000000,fill:#00FFF4
```

<SwmSnippet path="/cibuildwheel/linux.py" line="483">

---

# Troubleshooting build options

The `troubleshoot` function is used to handle exceptions that occur during the wheel building process. It checks if the error is a `subprocess.CalledProcessError` and if the command that caused the error is related to wheel building or repair. If shared object (.so) files are found in the project, it prints a note to stderr with suggestions on how to avoid potential issues with auditwheel.

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

<SwmSnippet path="/cibuildwheel/linux.py" line="563">

---

# Building in a container

The `build_options` function is used to compute BuildOptions for a single run configuration. It reads various configuration options from the environment, such as the build frontend, dependency versions, test command, and more. It also sets up the environment for the build, including passing through environment variables and setting up manylinux and musllinux images.

```python

```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/oci_container.py" line="308">

---

# Copying into the OCI container

The `call` function is used to execute a command in the OCI container. It sets up the working directory and environment, logs the command being executed, and writes the command to the remote shell. If the command returns a non-zero exit code, it raises a `subprocess.CalledProcessError`.

```python
    def call(
        self,
        args: Sequence[PathOrStr],
        env: Mapping[str, str] | None = None,
        capture_output: bool = False,
        cwd: PathOrStr | None = None,
    ) -> str:
        if cwd is None:
            # Podman does not start the a container in a specific working dir
            # so we always need to specify it when making calls.
            cwd = self.cwd

        chdir = f"cd {cwd}" if cwd else ""
        env_assignments = (
            " ".join(f"{shlex.quote(k)}={shlex.quote(v)}" for k, v in env.items())
            if env is not None
            else ""
        )
        command = " ".join(shlex.quote(str(a)) for a in args)
        end_of_message = str(uuid.uuid4())

```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/util.py" line="313">

---

# Writing to the OCI container

The `write` function is used to write data to the OCI container. It writes the data to the stream and then flushes the stream to ensure the data is sent.

```python
    def write(self, data: str) -> None:
        self.stream.write(data)
        self.stream.flush()
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="flows"><sup>Powered by [Swimm](/)</sup></SwmMeta>
