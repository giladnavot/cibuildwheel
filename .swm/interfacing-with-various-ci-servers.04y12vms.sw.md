---
title: 'Interfacing with Various CI Servers '
---
This document will cover how cibuildwheel interfaces with various Continuous Integration (CI) servers. We'll cover:

1. The CI services supported by cibuildwheel
2. How cibuildwheel detects the CI provider
3. How cibuildwheel tests example configurations for each CI service.

<SwmSnippet path="/bin/run_example_ci_configs.py" line="43">

---

# Supported CI Services

cibuildwheel supports a variety of CI services, including Appveyor, Azure Pipelines, CircleCI, and GitHub. Each service is represented as a `CIService` object, which includes the name of the service, the path to the configuration file, and a markdown string for the build status badge.

```python
services = [
    CIService(
        name="appveyor",
        dst_config_path="appveyor.yml",
        badge_md="[![Build status](https://ci.appveyor.com/api/projects/status/gt3vwl88yt0y3hur/branch/{branch}?svg=true)](https://ci.appveyor.com/project/joerick/cibuildwheel/branch/{branch})",
    ),
    CIService(
        name="azure-pipelines",
        dst_config_path="azure-pipelines.yml",
        badge_md="[![Build Status](https://dev.azure.com/joerick0429/cibuildwheel/_apis/build/status/pypa.cibuildwheel?branchName={branch})](https://dev.azure.com/joerick0429/cibuildwheel/_build/latest?definitionId=2&branchName={branch})",
    ),
    CIService(
        name="circleci",
        dst_config_path=".circleci/config.yml",
        badge_md="[![CircleCI](https://circleci.com/gh/pypa/cibuildwheel/tree/{branch_escaped}.svg?style=svg)](https://circleci.com/gh/pypa/cibuildwheel/tree/{branch})",
    ),
    CIService(
        name="github",
        dst_config_path=".github/workflows/example.yml",
        badge_md="[![Build](https://github.com/pypa/cibuildwheel/workflows/Build/badge.svg?branch={branch})](https://github.com/pypa/cibuildwheel/actions)",
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/oci_container.py" line="22">

---

# Detecting the CI Provider

The `detect_ci_provider` function is used to identify the CI provider. This function is imported from the `util` module.

```python
    CIProvider,
    call,
    detect_ci_provider,
    parse_key_value_string,
    strtobool,
```

---

</SwmSnippet>

<SwmSnippet path="/bin/run_example_ci_configs.py" line="94">

---

# Testing Example Configurations

The `run_example_ci_configs` function is used to test the example configurations for each CI service. If no configuration files are specified, it defaults to testing `examples/*-minimal.yml`. The function checks that each CI service has at most one configuration file, and raises an exception if more than one is found. It then checks for uncommitted changes in the git repo, creates a new branch, generates a basic project, and tests the configuration files. If all tests pass, it commits the changes and pushes the branch.

```python
def run_example_ci_configs(config_files=None):
    """
    Test the example configs. If no files are specified, will test
    examples/*-minimal.yml
    """

    if len(config_files) == 0:
        config_files = glob("examples/*-minimal.yml")

    # check each CI service has at most 1 config file
    configs_by_service = {}
    for config_file in config_files:
        service = ci_service_for_config_file(config_file)
        if service.name in configs_by_service:
            msg = "You cannot specify more than one config per CI service"
            raise Exception(msg)
        configs_by_service[service.name] = config_file

    if git_repo_has_changes():
        print("Your git repo has uncommitted changes. Commit or stash before continuing.")
        sys.exit(1)
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="follow-up"><sup>Powered by [Swimm](/)</sup></SwmMeta>
