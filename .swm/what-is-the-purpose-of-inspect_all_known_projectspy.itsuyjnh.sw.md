---
title: what is the purpose of inspect_all_known_projects.py
---
The script <SwmPath repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel" path="bin/inspect_all_known_projects.py">`(cibuildwheel) bin/inspect_all_known_projects.py`</SwmPath> seems to be designed to inspect Python projects and check for Python version requirements. It appears to do this by parsing the project files and looking for specific keywords related to Python version requirements.

<SwmSnippet path="/bin/inspect_all_known_projects.py" line="30" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==">

---

The <SwmToken path="bin/inspect_all_known_projects.py" pos="30:2:2" line-data="def parse(contents: str) -&gt; str | None:" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`parse`</SwmToken> function is used to parse the contents of a Python file. It uses the <SwmToken path="bin/inspect_all_known_projects.py" pos="32:5:7" line-data="        tree = ast.parse(contents)" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`ast.parse`</SwmToken> function to parse the contents into an abstract syntax tree, which is then visited by an <SwmToken path="bin/inspect_all_known_projects.py" pos="33:1:1" line-data="        analyzer = Analyzer()" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`analyzer`</SwmToken> object. The function returns the Python version requirement if it exists.

```python
def parse(contents: str) -> str | None:
    try:
        tree = ast.parse(contents)
        analyzer = Analyzer()
        analyzer.visit(tree)
        return analyzer.requires_python or ""
    except Exception:
        return None
```

---

</SwmSnippet>

<SwmSnippet path="/bin/inspect_all_known_projects.py" line="40" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==">

---

The <SwmToken path="bin/inspect_all_known_projects.py" pos="40:2:2" line-data="def check_repo(name: str, contents: str) -&gt; str:" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`check_repo`</SwmToken> function checks a repository for Python version requirements. It checks different files (<SwmToken path="bin/inspect_all_known_projects.py" pos="42:8:10" line-data="    if name == &quot;setup.py&quot;:" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`setup.py`</SwmToken>, <SwmToken path="bin/inspect_all_known_projects.py" pos="53:8:10" line-data="    elif name == &quot;setup.cfg&quot;:" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`setup.cfg`</SwmToken>, and others) for the <SwmToken path="bin/inspect_all_known_projects.py" pos="43:4:4" line-data="        if &quot;python_requires&quot; not in contents:" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`python_requires`</SwmToken> keyword and returns a string indicating whether the keyword was found and what the requirement is.

```python
def check_repo(name: str, contents: str) -> str:
    s = f"  {name}: "
    if name == "setup.py":
        if "python_requires" not in contents:
            s += "❌"
        res = parse(contents)
        if res is None:
            s += "⚠️ "
        elif res:
            s += "✅ " + res
        elif "python_requires" in contents:
            s += "☑️"

    elif name == "setup.cfg":
        s += "✅" if "python_requires" in contents else "❌"
    else:
        s += "✅" if "requires-python" in contents else "❌"

    return s
```

---

</SwmSnippet>

<SwmSnippet path="/bin/inspect_all_known_projects.py" line="108" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==">

---

The <SwmToken path="bin/inspect_all_known_projects.py" pos="108:2:2" line-data="def main(online: str | None) -&gt; None:" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`main`</SwmToken> function orchestrates the script's operations. It loads a list of known projects from a YAML file, creates a <SwmToken path="bin/inspect_all_known_projects.py" pos="114:5:5" line-data="    ghinfo = MaybeRemote(&quot;all_known_setup.yaml&quot;, online=online)" repo-id="Z2l0aHViJTNBJTNBY2lidWlsZHdoZWVsJTNBJTNBZ2lsYWRuYXZvdA==" repo-name="cibuildwheel">`MaybeRemote`</SwmToken> object to handle fetching file contents, and then iterates over each repository, checking each one and printing the results.

```python
def main(online: str | None) -> None:
    with open(DIR / "../docs/data/projects.yml") as f:
        known = yaml.safe_load(f)

    repos = [x["gh"] for x in known]

    ghinfo = MaybeRemote("all_known_setup.yaml", online=online)

    for _, filename, contents in ghinfo.on_each(repos):
        if contents is None:
            print(f"[red]  {filename}: Not found")
        else:
            print(check_repo(filename, contents))

    if online:
        ghinfo.save("all_known_setup.yaml")
```

---

</SwmSnippet>

<SwmMeta version="3.0.0"><sup>Powered by [Swimm](/)</sup></SwmMeta>
