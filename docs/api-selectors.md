---
id: api-selectors
title: Selectors
---

Selectors are defined using the **lib2to3** pattern syntax to search the syntax tree
for elements matching that pattern, and capturing specific nested nodes when matches
are found. Patterns can be nested, with alternate branches, allowing arbitrarily
complex matching.  

## Pattern Syntax

Selector patterns follow a very simple syntax, as defined in the **lib2to3**
[pattern grammar][]. Matching elements of the [Python grammar][] is done by listing
the grammar element, optionally followed by angle brackets containing nested match
expressions. The `any` keyword can be used to match grammar elements, regardless of
their type, while `*` denotes elements that repeat zero or more times. 

Make sure to include _necessary_ string literal tokens when using nested expressions, 
and `any*` to match remaining _grammar_ elements.

```python
# any* does not capture the '=' string literal
expr_stmt<
    attr_name='{name}' attr_value=any*
>
# but declare '(' and ')' string literals to differentiate from '[' and ']'
trailer< '(' function_arguments=any* ')' >
```

Example pattern to match class definitions that contain a function definition (the
"suite" denotes the body of the class definition):

```python
PATTERN = """
    classdef<
        any*
        suite<
            any* funcdef any*
        >
    >
"""
```

The root element of the expression is what will be passed to filters and modifiers by
default when matched. Capturing nested elements is done by preceding elements in the
expressions with the desired name and an equals sign – these elements will then show
up in the `Capture` argument to filters and modifiers. Square brackets can denote
optional (zero or one) elements.

Building on the example above, we can capture the defined classname, any ancestors, and
the function name:

```python
PATTERN = """
    classdef<
        "class" name=NAME ["(" [ancestors=arglist] ")"] ":"
        suite<
            any*
            funcdef< "def" func_name=NAME any* >
            any*
        >
    >
"""
```

To capture multiple possible arrangements of grammar elements, the `|` can be used with
parentheses to combine expressions. You can reuse capture names for each expression,
and you can further nest alternative expressions.

To find both function definitions and function calls in a single selector:

```python
PATTERN = """
    (
        funcdef< "def" name=NAME "(" [args=typedargslist] ")" any* >
    |
        power< name=NAME trailer< "(" [args=arglist] ")" > any* >
    )
"""
```

## Selector Reference

All selectors are accessed as methods on the [`Query`](api-query) class:

<AUTOGENERATED_TABLE_OF_CONTENTS>

> Note: Bowler's API is still in a "provisional" phase.  The core concepts will likely
> stay the same, but individual method signatures may change as the tool matures or
> gains new functionality.

### `.select()`

Supply a custom selector pattern as the given string, and auto-format it with any
supplied keyword arguments. Captures are defined by the supplied pattern.

```python
query.select(pattern: str, **kwargs: str)
```

### `.select_root()`

Select only the root node of the syntax tree, the `file_input` node. No captures.

```python
query.select_root()
```

### `.select_module()`

Select specified modules when imported and referenced.

```python
query.select_module(name: str)
```

Capture | Description | Example | Match
---|---|---|---
module_access | Attributes accessed | `foo.bar()` | `bar`
module_imports | Attributes imported | `from foo import bar` | `[bar]`
module_name | Name of module | `import foo` | `foo`
module_nickname | Nickname at import | `import foo as bar` | `bar`

### `.select_class()`

Select specified classes when imported, defined, subclassed, or instantiated.

```python
query.select_class(name: str)
```

Capture | Description | Example | Match
---|---|---|---
class_arguments | Arguments of instantiation | `Foo(bar)` | `bar`
class_call | Class being instantiated | `Foo()` | `Foo`
class_def | Root of the class definition | `class Foo:` | `class`
class_import | Root of the import | `from foo import Bar` | `from`
class_name | Name of class | `class Bar(Foo):` | `Bar`
class_subclass | Root of the subclass definition | `class Bar(Foo):` | `class`

### `.select_subclass()`

Select subclasses of specified classes when defined.

```python
query.select_subclass(name: str)
```

Capture | Description | Example | Match
---|---|---|---
class_ancestor | Name of ancestor class | `class Bar(Foo):` | `Foo`
class_def | Root of the class definition | `class Bar(Foo):` | `class`
class_name | Name of class | `class Bar(Foo):` | `Bar`

### `.select_attribute()`

Select specified class attributes when defined, assigned, or accessed.

```python
query.select_attribute(name: str)
```

Capture | Description | Example | Match
---|---|---|---
attr_class | Root of class definition | n/a | n/a  
attr_access | Attribute access | `value = self.foo` | `value`
attr_assignment | Attribute assignment | `self.foo = 42` | `self.foo`
attr_name | Name of attribute | `self.foo = 42` | `foo`
attr_value | Value of attribute | `self.foo = 42` | `42`

### `.select_method()`

Select specified class methods when defined or called.

```python
query.select_method(name: str)
```

Capture | Description | Example | Match
---|---|---|---
decorators | Method decorators | `@classmethod` | `@classmethod`
function_arguments | Method arguments | `def foo(self):` | `[self]`
function_call | Method invocation | `foo.bar()` | `foo`
function_def | Method definition | `def foo(self):` | `def`
function_name | Method name | `def foo(self):` | `foo`

### `.select_function()`

Select specified functions when imported, defined, or called.

```python
query.select_function(name: str)
```

Capture | Description | Example | Match
---|---|---|---
decorators | Function decorators | `@wraps` | `@wraps`
function_arguments | Function arguments | `def foo(a, b):` | `[a, b]`
function_call | Function invocation | `foo()` | `foo`
function_def | Function definition | `def foo(self):` | `def`
function_import | Function import | `from foo import bar` | `from`
function_name | Function name | `def foo(self):` | `foo`


### `.select_var()`

Select arbitrary names when assigned or referenced.

```python
query.select_var(name: str)
```

Capture | Description | Example | Match
---|---|---|---
var_assignment | Variable assignment | `foo = 42` | `foo = 42`
var_name | Variable name | `foo = 42` | `foo`
var_value | Assignment value | `foo = 42` | `42`


[pattern grammar]: https://github.com/python/cpython/blob/main/Lib/lib2to3/PatternGrammar.txt
[python grammar]: https://github.com/python/cpython/blob/main/Lib/lib2to3/Grammar.txt
