---
title: Python Wheel Building Process in Linux
---
This document will cover the process of building Python wheels in a Linux environment using cibuildwheel. The process includes:

1. Troubleshooting potential issues
2. Building the wheel in a container
3. Writing the build output

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

# Troubleshooting potential issues

The `troubleshoot` function is used to identify potential issues that might occur during the wheel building process. It checks if the error is related to the wheel build or repair step and provides detailed instructions on how to resolve common issues.

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

# Building the wheel in a container

The `build_options` function is used to compute the build options for a single run configuration. It sets up the environment, configures the build settings, and prepares the container for the build process.

```python

```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/util.py" line="313">

---

# Writing the build output

The `write` function is used to write the build output. This is important for logging and debugging purposes.

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
