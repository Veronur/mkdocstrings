# Troubleshooting

## The generated documentation does not look good

Are you using the Material theme?

- "No": We do not support any other theme yet.
  Check the [bugtracker][bugtracker] to see if there is a feature request
  asking to support your theme. If you find one, vote with a thumbs up. If not, you can open a ticket.
- "Yes": Please open an ticket on the [bugtracker][bugtracker] with a detailed
  explanation and screenshots of the bad-looking parts.


## Some objects are not rendered (they do not appear in the generated docs)

- Make sure the configuration options of the handler for both selection and rendering are correct.
  Check the documentation for [Handlers](/handlers/overview) to see the available options for each handler.
- Also make sure your documentation in your source code is formatted correctly.
  For Python code, check the [docstring format](/handlers/python/#docstring-format) page.
- Check the output of the `mkdocs` command, and re-run it with `-v` if necessary.
  Warnings should appear, showing errors that happened during collection.

## Nothing is rendered at all

Python?

- "No": we only support Python right now.
- "Yes": is your package available in the Python path?
  If not, install it in your current virtualenv and try again.
  Make sure you don't have an old version of your package installed,
  shadowing your source code. 
  
## Mkdocs warns me about links to unfound documentation files

A warning like this one:

> WARNING -  Documentation file 'reference/parsers/docstrings.md'
  contains a link to 'reference/parsers/pytkdocs.parsers.docstrings.Section'
  which is not found in the documentation files.

...generally means you used parentheses `()` instead of brackets `[]` for a cross-reference.
Notice the dots in `reference/parsers/pytkdocs.parsers.docstrings.Section`?
It shows that it's probably a cross-reference, not a direct link.
It's probably written like `[Section](pytkdocs.parsers.docstrings.Section)` in the docs,
when it should be `[Section][pytkdocs.parsers.docstrings.Section]`.


## Python specifics

### My docstrings in comments (`#:`) are not picked up

It's because [`pytkdocs`][pytkdocs] does not pick up documentation in comments.
To load documentation for modules, classes, methods and functions, it uses [`inspect`][inspect].
To load documentation for attributes, it uses [`ast`][ast] to parse the source code,
searching for pairs of nodes like `assignment`-`string`, and [`ast`][ast] does not parse comments.

So instead of:

```python
import enum


class MyEnum(enum.Enum):
    v1 = 1  #: The first choice.
    v2 = 2  #: The second choice.
```

Write:

```python
import enum


class MyEnum(enum.Enum):
    v1 = 1
    """The first choice."""

    v2 = 2
    """The second choice."""
```

It does not look better, I know, but this is the price to pay.

### LaTeX in docstrings is not rendered correctly

If you are using a Markdown extension like [`markdown-katex`][markdown-katex] to render LaTeX,
add `r` in front of your docstring to make sure nothing is escaped.
You'll still maybe have to play with escaping to get things right.

Example:

```python
def math_function(x, y):
    r"""
    Look at these formulas:    

    ```math
    f(x) = \int_{-\infty}^\infty
    \hat f(\xi)\,e^{2 \pi i \xi x}
    \,d\xi
    ```
    """
```

### My wrapped function shows documentation/code for its wrapper instead of its own

Use [`functools.wraps()`](https://docs.python.org/3.6/library/functools.html#functools.wraps):

```python
from functools import wraps

def my_decorator(function):
    """The decorator docs."""    

    @wraps(function)
    def wrapped_function(*args, **kwargs):
        print("hello")
        function(*args, **kwargs)
        print("bye")

    return wrapped_function

@my_decorator
def my_function(*args, **kwargs):
    """The function docs."""
    print(*args, **kwargs)
```

[bugtracker]: https://github.com/pawamoy/mkdocstrings
[pytkdocs]: https://github.com/pawamoy/pytkdocs
[inspect]: https://docs.python.org/3/library/inspect.html
[ast]: https://docs.python.org/3/library/ast.html
[markdown-katex]: https://gitlab.com/mbarkhau/markdown-katex
