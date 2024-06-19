---
title: Introduction to Environment Configuration
---
Environment Configuration in cibuildwheel refers to the setup and management of environment variables that influence the behavior of the build process. This is achieved through classes and functions that parse, evaluate, and manage environment variables.

&nbsp;The <SwmToken path="/cibuildwheel/environment.py" pos="50:2:2" line-data="class EnvironmentAssignment(Protocol):">`EnvironmentAssignment`</SwmToken> protocol defines the structure for environment variable assignments, while the <SwmToken path="/cibuildwheel/environment.py" pos="143:6:6" line-data="    assignments = [EnvironmentAssignmentBash(item) for item in env_items]">`EnvironmentAssignmentBash`</SwmToken> and <SwmToken path="/cibuildwheel/environment.py" pos="62:2:2" line-data="class EnvironmentAssignmentRaw:">`EnvironmentAssignmentRaw`</SwmToken> classes handle bash-syntax and simple name/value pair environment variables respectively.&nbsp;

The <SwmToken path="/cibuildwheel/environment.py" pos="108:2:2" line-data="class ParsedEnvironment:">`ParsedEnvironment`</SwmToken> class is used to manage a list of environment assignments, providing functionality to convert these assignments into a dictionary, add new assignments, and get a summary of the options.&nbsp;

The <SwmToken path="/cibuildwheel/environment.py" pos="114:3:3" line-data="    def as_dictionary(">`as_dictionary`</SwmToken> method is particularly important as it is used across different modules to apply the environment configuration.

<SwmSnippet path="/cibuildwheel/environment.py" line="50">

---

## Environment Variables

The <SwmToken path="/cibuildwheel/environment.py" pos="50:2:2" line-data="class EnvironmentAssignment(Protocol):">`EnvironmentAssignment`</SwmToken> class represents an environment variable. It has a <SwmToken path="/cibuildwheel/environment.py" pos="51:1:1" line-data="    name: str">`name`</SwmToken> and a method <SwmToken path="/cibuildwheel/environment.py" pos="53:3:3" line-data="    def evaluated_value(">`evaluated_value`</SwmToken> that returns the value of the variable as evaluated in the environment.

```python
class EnvironmentAssignment(Protocol):
    name: str

    def evaluated_value(
        self,
        *,
        environment: Mapping[str, str],
        executor: bashlex_eval.EnvironmentExecutor | None = None,
    ) -> str:
        """Returns the value of this assignment, as evaluated in the environment"""
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/environment.py" line="141">

---

## Parsing Environment Variables

The <SwmToken path="/cibuildwheel/environment.py" pos="141:2:2" line-data="def parse_environment(env_string: str) -&gt; ParsedEnvironment:">`parse_environment`</SwmToken> function is used to parse environment variables from a string. It uses the <SwmToken path="/cibuildwheel/environment.py" pos="78:2:2" line-data="class EnvironmentAssignmentBash:">`EnvironmentAssignmentBash`</SwmToken> class to create an assignment for each item in the environment string.

```python
def parse_environment(env_string: str) -> ParsedEnvironment:
    env_items = split_env_items(env_string)
    assignments = [EnvironmentAssignmentBash(item) for item in env_items]
    return ParsedEnvironment(assignments=assignments)
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/macos.py" line="272">

---

## Environment Configuration in Action

The <SwmToken path="/cibuildwheel/environment.py" pos="114:3:3" line-data="    def as_dictionary(">`as_dictionary`</SwmToken> method of the <SwmToken path="/cibuildwheel/environment.py" pos="108:2:2" line-data="class ParsedEnvironment:">`ParsedEnvironment`</SwmToken> class is used to convert the environment configuration into a dictionary that can be used in the build process. This is an example of how it's used in the macOS build process.

```python
    # Apply our environment after pip is ready
    env = environment.as_dictionary(prev_environment=env)
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
