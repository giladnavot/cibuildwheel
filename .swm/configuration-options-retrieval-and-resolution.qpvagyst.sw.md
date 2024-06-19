---
title: Configuration Options Retrieval and Resolution
---
This document will cover the process of retrieving and resolving configuration options in the cibuildwheel project. The process includes the following steps:

1. Invoking the `get` function to retrieve a configuration option.
2. Resolving the cascade of configuration values using the `_resolve_cascade` function.
3. Converting the configuration setting to a string with the `_stringify_setting` function.
4. Formatting the configuration setting with the `_inner_fmt` function.

## Where is this flow used?

The flow starts with the function `build_options`. It is called from multiple entry points as represented in the following diagram:

```mermaid
graph TD;
  build:::rootsStyle --> build_in_container
  build_in_container --> build_options:::mainFlowStyle
  main:::rootsStyle --> main_inner
  main_inner --> build_in_directory
  build_in_directory --> print_preamble
  print_preamble --> summary
  summary --> build_options:::mainFlowStyle
  build:::rootsStyle --> build_options:::mainFlowStyle

 classDef mainFlowStyle color:#000000,fill:#7CB9F4
  classDef rootsStyle color:#000000,fill:#00FFF4
```

## The flow itself

```mermaid
graph TD;
subgraph cibuildwheel/options.py
  build_options:::mainFlowStyle --> get
end
subgraph cibuildwheel/options.py
  get:::mainFlowStyle --> _resolve_cascade
end
subgraph cibuildwheel/options.py
  _resolve_cascade:::mainFlowStyle --> _stringify_setting
end
subgraph cibuildwheel/options.py
  _stringify_setting:::mainFlowStyle --> _inner_fmt
end
  _inner_fmt:::mainFlowStyle --> ...

 classDef mainFlowStyle color:#000000,fill:#7CB9F4
  classDef rootsStyle color:#000000,fill:#00FFF4
```

<SwmSnippet path="/cibuildwheel/options.py" line="410">

---

# Invoking the `get` function

The `get` function is used to retrieve the value for a named option from the environment, configuration file, or the default. It constructs an environment variable form and gets the option from the default, then the config file, and finally the environment. Platform-specific options are preferred if they're allowed.

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

<SwmSnippet path="/cibuildwheel/options.py" line="162">

---

# Resolving the cascade of configuration values

The `_resolve_cascade` function is used to resolve a cascade of values with inherit rules into a single value. It handles 'None' values and empty values based on the `ignore_empty` parameter. It also handles the merging of values based on the rule provided.

```python
def _resolve_cascade(
    *pairs: tuple[Setting | None, InheritRule],
    ignore_empty: bool = False,
    list_sep: str | None = None,
    table_format: TableFmt | None = None,
) -> str:
    """
    Given a cascade of values with inherit rules, resolve them into a single
    value.

    'None' values mean that the option was not set at that level, and are
    ignored. If `ignore_empty` is True, empty values are ignored too.

    Values start with defaults, followed by more specific rules. If rules are
    NONE, the last non-null value is returned. If a rule is APPEND or PREPEND,
    the value is concatenated with the previous value.

    The following idiom can be used to get the first matching value:

        _resolve_cascade(("value1", Inherit.NONE), ("value2", Inherit.NONE), ...)))
    """
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/options.py" line="245">

---

# Converting the configuration setting to a string

The `_stringify_setting` function is used to convert the configuration setting to a string. It handles different types of settings including mappings, sequences, booleans, and integers.

```python
def _stringify_setting(
    setting: Setting, list_sep: str | None, table_format: TableFmt | None
) -> str:
    if isinstance(setting, Mapping):
        if table_format is None:
            msg = f"Error converting {setting!r} to a string: this setting doesn't accept a table"
            raise ConfigOptionError(msg)
        return table_format["sep"].join(
            item for k, v in setting.items() for item in _inner_fmt(k, v, table_format)
        )

    if not isinstance(setting, str) and isinstance(setting, Sequence):
        if list_sep is None:
            msg = f"Error converting {setting!r} to a string: this setting doesn't accept a list"
            raise ConfigOptionError(msg)
        return list_sep.join(setting)

    if isinstance(setting, (bool, int)):
        return str(setting)

    return setting
```

---

</SwmSnippet>

<SwmSnippet path="/cibuildwheel/options.py" line="457">

---

# Formatting the configuration setting

The `_inner_fmt` function is used to format the configuration setting. It handles different types of values including lists, booleans, and other types.

```python
def _inner_fmt(k: str, v: Any, table: TableFmt) -> Iterator[str]:
    quote_function = table.get("quote", lambda a: a)

    if isinstance(v, list):
        for inner_v in v:
            qv = quote_function(inner_v)
            yield table["item"].format(k=k, v=qv)
    elif isinstance(v, bool):
        qv = quote_function(str(v))
        yield table["item"].format(k=k, v=qv)
    else:
        qv = quote_function(v)
        yield table["item"].format(k=k, v=qv)
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm AI 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" doc-type="flows"><sup>Powered by [Swimm](/)</sup></SwmMeta>
