---
title: Understanding Resources in cibuildwheel
---
In cibuildwheel, 'resources' refers to a collection of files that are used to support the building of Python wheels. These files include constraints for different Python versions, configuration files, and scripts. For example, the 'constraints-pythonXX.txt' files contain constraints for building wheels with specific Python versions, while 'install_certifi.py' is a script used to install certificates. These resources are essential for the correct and efficient operation of cibuildwheel.

# Constraints Files

The constraints files, such as `constraints-python310.txt`, specify the versions of dependencies for each Python version. They are referenced in the build scripts to ensure that the correct versions of dependencies are used.

<SwmSnippet path="/cibuildwheel/resources/install_certifi.py" line="1">

---

# Certificates Installation Script

The `install_certifi.py` script is used to install certificates. It is based on a script from the Python cpython repository.

```python
# Based on: https://github.com/python/cpython/blob/master/Mac/BuildScript/resources/install_certificates.command
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/resources/cibuildwheel.schema.json" line="3">

---

# Configuration Schema

The `cibuildwheel.schema.json` file defines the schema for the cibuildwheel configuration. It includes descriptions for various configuration options, such as the build tool to use and how cibuildwheel controls the versions of the tools it uses.

```json
  "$id": "https://github.com/pypa/cibuildwheel/blob/main/cibuildwheel/resources/cibuildwheel.schema.json",
  "$defs": {
    "inherit": {
      "enum": [
        "none",
        "prepend",
        "append"
      ],
      "default": "none",
      "description": "How to inherit the parent's value."
    }
  },
  "additionalProperties": false,
  "description": "cibuildwheel's settings.",
  "type": "object",
  "properties": {
    "archs": {
      "description": "Change the architectures built on your machine by default.",
      "oneOf": [
        {
          "type": "string"
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="overview"><sup>Powered by [Swimm](/)</sup></SwmMeta>
