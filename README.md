# htbuilder — tiny HTML string builder for Python

htbuilder lets you build HTML strings using a purely functional syntax in Python.
Why use templating languages when you can just use functions?

(PS: If you like this, check out [jsbuild](https://github.com/tvst/jsbuild) which
lets you build JavaScript strings by simply annotating Python functions!)

## Installation

Just PIP it!

```py
pip install htbuilder
```

## Usage

Just import tags like `div` with `from htbuilder import div`, then call them:

```py
# Import any tag you want from htbuilder, and it just works!
# (This syntax requires Python 3.7+. See below for an alternate syntax)
from htbuilder import div

dom = div('Hello world!')
```

Then you can get the string output by calling `str()` on it:

```py
str(dom)
# Returns '<div>Hello world!</div>'
```

...which means you can also just `print()` to see it in the terminal:

```py
print(dom)
# Prints '<div>Hello world!</div>'
```

To specify attributes, call the tag builder with keyword args:

```py
print(
  div(id='sidebar', foo='bar')
)
# Prints '<div id="sidebar" foo="bar"></div>'
```

To specify both attributes and children, first specify the attributes using
keyword args, then pass the children afterwards **inside a new
set of parentheses**:

```py
print(
  div(id='sidebar', foo='bar')(
    "Hello world!"
  )
)
# Prints '<div id="sidebar" foo="bar">Hello world!</div>'
```

This is required because Python doesn't allow you to pass keyword arguments
_before_ you pass normal arguments.

## Multiple children

Want to output multiple children? Just pass them all as arguments:

```py
from htbuilder import div, ul, li, img

dom = (
  div(id='container')(
    ul(_class='greetings')(
      li('hello'),
      li('hi'),
      li('whattup'),
    )
  )
)

print(dom)

# Prints this (but without added spacing):
# <div id="container">
#   <ul class="greetings">
#     <li>hello</li>
#     <li>hi</li>
#     <li>whattup</li>
#   </ul>
# </div>
```

## Grouping children without rendering extra elements

It can be useful to group a number of logically related elements, without rendering an additional element to act
as a container (e.g. for more readable indentation of your
element tree in Python). To achieve this, use the `fragment`
helper:

```py
from htbuilder import fragment, div, b, i

dom = (
  div(
    "The following flavours are available: ",
    fragment(
      b("strawberry"),
      ", ",
      b("vanilla"),
      " and ",
      b("pistachio"),
      ". "
    ),
    "The following flavours are unavailable: ",
    fragment(
      i("chocolate"),
      " and ",
      i("raspberry"),
      "."
    )
  )
)

print(dom)
# Prints: <div>The following flavours are available: <b>strawberry</b>, <b>vanilla</b> and <b>pistachio</b>. The following flavours are unavailable: <i>chocolate</i> and <i>raspberry</i>.</div>
```

## Programmatically add children

You can also pass any iterable to specify multiple children, which means you can
simply use things like generator expressions for great awesome:

```py
from htbuilder import div, ul, li, img

image_paths = [
  'http://myimages.com/foo1.jpg',
  'http://myimages.com/foo2.jpg',
  'http://myimages.com/foo3.jpg',
]

dom = (
  div(id='container')(
    ul(_class='image-list')(
      li(img(src=image_path, _class='large-image'))
      for image_path in image_paths
    )
  )
)

print(dom)
# Prints:
# <div id="container">
#   <ul class="image-list">
#     <li><img src="http://myimages.com/foo1.jpg" class="large-image"/></li>
#     <li><img src="http://myimages.com/foo2.jpg" class="large-image"/></li>
#     <li><img src="http://myimages.com/foo3.jpg" class="large-image"/></li>
#   </ul>
# </div>
```

## Conditionally add elements and attributes

And because it's just Python, you can use an if/else expression to conditionally
insert elements:

```py
use_bold = True

dom = (
  div(
      b("bold text")
    if use_bold else
      "normal text"
  )
)

print(dom)
# Prints: <div><b>bold text</b></div>
```

Any child argument that evaluates to `None`, `True`, or `False` will not be rendered:

```py
from htbuilder import div, span

show_this = True

dom = (
  div("Always show me. ", show_this and "Sometimes show me.", bool("Never show me"), bool(""))
)
print(dom)
# Prints: <div>Always show me. Sometimes show me.</div>

show_this = False

dom = (
  div("Always show me. ", show_this and "Sometimes show me.")
)

print(dom)
# Prints: <div>Always show me. </div>
```

Similarly, any attribute which evaluates to `None` or `False` will not be rendered:

```py
maybe_empty_str = ""
dom = (
  div(foo=None, bar="baz", baz=maybe_empty_str or None, buzz=False)
)

print(dom)
# Prints: <div bar="baz"></div>
```

However, attributes set to `True` will be interpreted as so-called Boolean attributes,
and will be rendered with only a key but no value:

```py
from htbuilder import div, script, option

dom = (
  div(
    script(defer=True, src="some.js"),
    option(disabled=True, name="a disabled option"),
    option(disabled=False, name="an enabled option"),
  )
)

print(dom)
# Prints: <div><script defer src="some.js"></script><option disabled name="a disabled option"></option><option name="an enabled option"></option></div>
```

## Styling

We provide helpers to write styles without having to pass huge style strings as
arguments. Instead, just use handy builders like `styles()`, `classes()`,
`fonts()`, along with helpers you can import from the `units` and `funcs`
modules.

```py
# styles, classes, and fonts are special imports to help build attribute strings.
from htbuilder import div, styles, classes, fonts

# You can import anything from .units and .funcs to make it easier to specify
# units like "%" and "px", as well as functions like "rgba()" and "rgba()".
from htbuilder.units import percent, px
from htbuilder.funcs import rgba, rgb

bottom_margin = 10
is_big = True

dom = (
  div(
    _class=classes('btn', big=is_big)
    style=styles(
        color='black',
        font_family=fonts('Comic Sans', 'sans-serif'),
        margin=px(0, 0, bottom_margin, 0),
        padding=(px(10), percent(5))
        box_shadow=[
            (0, 0, px(10), rgba(0, 0, 0, 0.1)),
            (0, 0, '2px', rgb(0, 0, 0)),
        ],
    )
  )
)

# Prints:
# <div
#   class="btn big"
#   style="
#     color: black;
#     font-family: "Comic Sans", "sans-serif";
#     margin: 0 0 10px 0;
#     padding: 10px 5%;
#     box-shadow: 0 0 10px rgba(0, 0, 0, 0.1), 0 0 2px rgb(0, 0, 0);
#   "></div>
```

## Underscores are magic

### Use underscores instead of dashes

Like most popular languages, Python doesn't support dashes in identifiers. So if you want to build
an element that includes dashes in the tag name or attributes, like `<my-element foo-bar="baz">`, you can
do so by using underscores instead:

```py
from htbuilder import my_element

dom = my_element(foo_bar="baz")

print(dom)
# Prints:
# <my-element foo-bar="baz"></my-element>
```

### Prefix with underscore to avoid reserved words

The word `class` is reserved in Python, so if you want to set an element's `class` attribute you
should prepend it with an underscore like this:

```py
dom = div(_class="myclass")

print(dom)
# Prints:
# <div class="myclass"></div>
```

This works because underscores preceding or following any identifier are automatically stripped away
for you.

## Working with Python &lt; 3.7

If using Python &lt; 3.7, the import should look like this instead:

```py
from htbuilder import H

div = H.div
ul = H.ul
li = H.li
img = H.img
# ...etc
```
