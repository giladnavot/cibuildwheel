---
title: Working Mechanism of Overrides in Build Configuration
---
This document will cover the role of 'overrides' in build configuration within the cibuildwheel project. We'll cover:

1. What is the 'overrides' concept
2. How 'overrides' are implemented in the codebase
3. Examples of 'overrides' usage

# What is the 'overrides' concept

In the context of cibuildwheel, 'overrides' are used to customize the build configuration. They allow users to specify different settings for different build environments. This is particularly useful when building Python wheels across multiple platforms and Python versions, where certain settings might need to be adjusted based on the target platform or Python version.

<SwmSnippet path="/cibuildwheel/options.py" line="125">

---

# How 'overrides' are implemented in the codebase

The 'overrides' concept is implemented through the `Override` class. This class has three attributes: `select_pattern`, `options`, and `inherit`. The `select_pattern` attribute is used to determine which build environments the override applies to. The `options` attribute is a dictionary of settings that should be overridden. The `inherit` attribute is a dictionary that determines how settings should be inherited from the global configuration.

```python
class Override:
    select_pattern: str
    options: dict[str, Setting]
    inherit: dict[str, InheritRule]
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/options.py" line="318">

---

The `Override` objects are stored in the `overrides` attribute of the `ConfigReader` class. When a new override is added, it is appended to this list.

```python
        self.overrides: list[Override] = []
        self.current_identifier: str | None = None

        config_overrides = self.config_options.get("overrides")

        if config_overrides is not None:
            if not isinstance(config_overrides, list):
                msg = "'tool.cibuildwheel.overrides' must be a list"
                raise ConfigOptionError(msg)

            for config_override in config_overrides:
                select = config_override.pop("select", None)

                if not select:
                    msg = "'select' must be set in an override"
                    raise ConfigOptionError(msg)

                if isinstance(select, list):
                    select = " ".join(select)

                inherit = config_override.pop("inherit", {})
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/options.py" line="395">

---

The `active_config_overrides` method of the `ConfigReader` class returns a list of `Override` objects that match the current build environment. This is determined by the `selector_matches` function, which checks if the current build environment matches the `select_pattern` of the `Override`.

```python
    def active_config_overrides(self) -> list[Override]:
        if self.current_identifier is None:
            return []
        return [
            o for o in self.overrides if selector_matches(o.select_pattern, self.current_identifier)
        ]
```

---

</SwmSnippet>

<SwmSnippet path="/bin/generate_schema.py" line="221">

---

# Examples of 'overrides' usage

This is an example of how 'overrides' are defined in the schema. Each override is an object that must have a 'select' property and at least one other property. The 'select' property determines which build environments the override applies to. The other properties are the settings that should be overridden.

```python
overrides = yaml.safe_load(
    """
type: array
description: An overrides array
items:
  type: object
  required: ["select"]
  minProperties: 2
  additionalProperties: false
  properties:
    select: {}
    inherit:
      type: object
      additionalProperties: false
      properties:
        before-all: {"$ref": "#/$defs/inherit"}
        before-build: {"$ref": "#/$defs/inherit"}
        before-test: {"$ref": "#/$defs/inherit"}
        config-settings: {"$ref": "#/$defs/inherit"}
        container-engine: {"$ref": "#/$defs/inherit"}
        environment: {"$ref": "#/$defs/inherit"}
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/options.py" line="767">

---

The `option_summary` method of the `ConfigReader` class provides a summary of a given option, including any overrides. If an override is active for the current build environment, it will be included in the summary.

```python
    def option_summary(
        self,
        option_name: str,
        option_value: Any,
        default_value: Any,
        overrides: Mapping[str, Any] | None = None,
    ) -> str:
        """
        Return a summary of the option value, including any overrides, with
        ANSI 'dim' color if it's the default.
        """
        value_str = self.option_summary_value(option_value)
        default_value_str = self.option_summary_value(default_value)
        overrides_value_strs = {
            k: self.option_summary_value(v) for k, v in (overrides or {}).items()
        }
        # if the override value is the same as the non-overridden value, don't print it
        overrides_value_strs = {k: v for k, v in overrides_value_strs.items() if v != value_str}

        has_been_set = (value_str != default_value_str) or overrides_value_strs
        c = log.colors
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="follow-up"><sup>Powered by [Swimm](/)</sup></SwmMeta>
