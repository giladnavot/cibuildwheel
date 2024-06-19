---
title: what are the options of the--platform flag?
---
<SwmPath repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" path="docs/options.md">`(cibuildwheel) docs/options.md`</SwmPath>:\
The <SwmToken path="cibuildwheel/__main__.py" pos="79:2:3" line-data="        &quot;--platform&quot;," repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`--platform`</SwmToken> flag in cibuildwheel is used to override the <SwmToken path="cibuildwheel/__main__.py" pos="84:1:3" line-data="            auto-detected platform. Specifying &quot;macos&quot; or &quot;windows&quot; only works" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`auto-detected`</SwmToken> target platform. This allows you to specify the platform for which you want to build your Python wheels.

<SwmSnippet path="/cibuildwheel/__main__.py" line="274" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==">

---

The function <SwmToken path="cibuildwheel/__main__.py" pos="275:2:2" line-data="def get_platform_module(platform: PlatformName) -&gt; PlatformModule:" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`get_platform_module`</SwmToken> in the codebase suggests that the possible options for the <SwmToken path="cibuildwheel/__main__.py" pos="79:2:3" line-data="        &quot;--platform&quot;," repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`--platform`</SwmToken> flag could be 'linux', 'windows', 'macos', and 'pyodide'.

```python
# pylint: disable-next=inconsistent-return-statements
def get_platform_module(platform: PlatformName) -> PlatformModule:
    if platform == "linux":
        return cibuildwheel.linux
    if platform == "windows":
        return cibuildwheel.windows
    if platform == "macos":
        return cibuildwheel.macos
    if platform == "pyodide":
        return cibuildwheel.pyodide
```

---

</SwmSnippet>

<SwmMeta version="3.0.0"><sup>Powered by [Swimm](/)</sup></SwmMeta>
