# poyo
<!-- vim-markdown-toc GFM -->

* [Installation](#installation)
* [Usage](#usage)
* [Example](#example)
    * [Input YAML string](#input-yaml-string)
    * [Output Python dict](#output-python-dict)
* [Logging](#logging)
    * [Disable Logging](#disable-logging)
    * [Example Debug Logging Config](#example-debug-logging-config)
    * [Example Debug Logging Messages](#example-debug-logging-messages)
* [About this project](#about-this-project)
* [Community](#community)
* [License](#license)

<!-- vim-markdown-toc -->

A lightweight YAML Parser for Python. 🐓

**poyo** does not allow deserialization of arbitrary Python objects. Supported
types are `str`, `bool`, `int`, `float`, `NoneType` as well as `dict` and
`list` values.

⚠️ Please note that poyo supports only a chosen subset of the YAML format
that is required to parse [cookiecutter user configuration
files][cookiecutterrc]. poyo does not have support for serializing into YAML
and is not compatible with JSON.

[cookiecutterrc]: https://cookiecutter.readthedocs.io/en/latest/advanced/user_config.html

## Installation

poyo is available on [PyPI][PyPI] for Python versions 2.7 and newer and can
be installed with [pip][pip]:

```text
pip install poyo
```

[PyPI]: https://pypi.org/project/poyo/
[pip]: https://pypi.org/project/pip/

This package does not have any additional requirements. 📦

## Usage

poyo comes with a ``parse_string()`` function, to load utf-8 encoded string
data into a Python dict.

```python
import codecs
import logging

from poyo import parse_string, PoyoException

logging.basicConfig(level=logging.DEBUG)

with codecs.open("tests/foobar.yml", encoding="utf-8") as ymlfile:
    ymlstring = ymlfile.read()

try:
    config = parse_string(ymlstring)
except PoyoException as exc:
    logging.error(exc)
else:
    logging.debug(config)
```

## Example

### Input YAML string

```yaml
---
default_context: # foobar
    greeting: こんにちは
    email: "raphael@hackebrot.de"
    docs: true

    gui: FALSE
    123: 456.789
    # comment
    # allthethings
    'some:int': 1000000
    foo: "hallo #welt" #Inline comment :)
    longtext: >
        This is a multiline string.
        It can contain all manners of characters.

        Single line breaks are ignored,
        but blank linkes cause line breaks.
    trueish: Falseeeeeee
    blog   : raphael.codes
    relative-root: /          # web app root path (default: '')
    lektor: 0.0.0.0:5000      # local build
    doc_tools:
        # docs or didn't happen
        -    mkdocs
        - 'sphinx'

        - null
    # 测试列表中的字典
    list_dict:
        - source: /index.html
            destination: /var/www/html/
        - source: /meetbill.jpg
            destination: /var/www/images/
zZz: True
NullValue: Null

# Block
# Comment

Hello World:
    # See you at EuroPython
    null: This is madness   # yo
    gh: https://github.com/{0}.git
"Yay #python": Cool!
```

### Output Python dict

```python
{   u'Hello World': {   None: u'This is madness',
                        u'gh': u'https://github.com/{0}.git'},
    u'NullValue': None,
    u'Yay #python': u'Cool!',
    u'default_context': {   123: 456.789,
                            u'blog': u'raphael.codes',
                            u'doc_tools': [u'mkdocs', u'sphinx', None],
                            u'docs': True,
                            u'email': u'raphael@hackebrot.de',
                            u'foo': u'hallo #welt',
                            u'greeting': u'\u3053\u3093\u306b\u3061\u306f',
                            u'gui': False,
                            u'lektor': u'0.0.0.0:5000',
                            u'list_dict': [   {   u'destination': u'/var/www/html/',
                                                  u'source': u'/index.html'},
                                              {   u'destination': u'/var/www/images/',
                                                  u'source': u'/meetbill.jpg'}],
                            u'longtext': u'This is a multiline string. It can contain all manners of characters.\nSingle line breaks are ignored, but blank linkes cause line breaks.\n',
                            u'relative-root': u'/',
                            u'some:int': 1000000,
                            u'trueish': u'Falseeeeeee'},
    u'zZz': True}
```

## Logging

poyo follows the recommendations for [logging in a library][logging], which
means it does not configure logging itself. Its root logger is named ``poyo``
and the names of all its children loggers track the package/module hierarchy.
poyo logs to a ``NullHandler`` and solely on ``DEBUG`` level.

If your application configures logging and allows debug messages to be shown,
you will see logging when using poyo. The log messages indicate which parser
method is used for a given string as the parser deseralizes the config. You
can remove all logging from poyo in your application by setting the log level
of the ``poyo`` logger to a value higher than ``DEBUG``.

[logging]: https://docs.python.org/3/howto/logging.html#configuring-logging-for-a-library

### Disable Logging

```python
import logging

logging.getLogger("poyo").setLevel(logging.WARNING)
```

### Example Debug Logging Config

```python
import logging
from poyo import parse_string

logging.basicConfig(level=logging.DEBUG)

CONFIG = """
---
default_context: # foobar
    greeting: こんにちは
    gui: FALSE
    doc_tools:
        # docs or didn't happen
        -    mkdocs
        - 'sphinx'
    123: 456.789
"""

parse_string(CONFIG)
```

### Example Debug Logging Messages

```text
DEBUG:poyo.parser:parse_blankline <- \n
DEBUG:poyo.parser:parse_blankline -> IGNORED
DEBUG:poyo.parser:parse_dashes <- ---\n
DEBUG:poyo.parser:parse_dashes -> IGNORED
DEBUG:poyo.parser:parse_section <- default_context: # foobar\n
DEBUG:poyo.parser:parse_str <- default_context
DEBUG:poyo.parser:parse_str -> default_context
DEBUG:poyo.parser:parse_section -> <Section name: default_context>
DEBUG:poyo.parser:parse_simple <-     greeting: \u3053\u3093\u306b\u3061\u306f\n
DEBUG:poyo.parser:parse_str <- greeting
DEBUG:poyo.parser:parse_str -> greeting
DEBUG:poyo.parser:parse_str <- \u3053\u3093\u306b\u3061\u306f
DEBUG:poyo.parser:parse_str -> \u3053\u3093\u306b\u3061\u306f
DEBUG:poyo.parser:parse_simple -> <Simple name: greeting, value: \u3053\u3093\u306b\u3061\u306f>
DEBUG:poyo.parser:parse_simple <-     gui: FALSE\n
DEBUG:poyo.parser:parse_str <- gui
DEBUG:poyo.parser:parse_str -> gui
DEBUG:poyo.parser:parse_false <- FALSE
DEBUG:poyo.parser:parse_false -> False
DEBUG:poyo.parser:parse_simple -> <Simple name: gui, value: False>
DEBUG:poyo.parser:parse_list <-     doc_tools:\n        # docs or didn't happen\n        -    mkdocs\n        - 'sphinx'\n
DEBUG:poyo.parser:parse_str <- mkdocs
DEBUG:poyo.parser:parse_str -> mkdocs
DEBUG:poyo.parser:parse_str <- 'sphinx'
DEBUG:poyo.parser:parse_str -> sphinx
DEBUG:poyo.parser:parse_str <- doc_tools
DEBUG:poyo.parser:parse_str -> doc_tools
DEBUG:poyo.parser:parse_list -> <Simple name: doc_tools, value: ['mkdocs', 'sphinx']>
DEBUG:poyo.parser:parse_simple <-     123: 456.789\n
DEBUG:poyo.parser:parse_int <- 123
DEBUG:poyo.parser:parse_int -> 123
DEBUG:poyo.parser:parse_float <- 456.789
DEBUG:poyo.parser:parse_float -> 456.789
DEBUG:poyo.parser:parse_simple -> <Simple name: 123, value: 456.789>
DEBUG:poyo.parser:parse_simple <-     docs: true\n
DEBUG:poyo.parser:parse_str <- docs
DEBUG:poyo.parser:parse_str -> docs
DEBUG:poyo.parser:parse_true <- true
DEBUG:poyo.parser:parse_true -> True
DEBUG:poyo.parser:parse_simple -> <Simple name: docs, value: True>
```

## About this project

We created this project to work around installation issues with a
[cookiecutter][cookiecutter] version that depended on existing YAML parsers
for Python. For more information please check out this [GitHub issue][issue].

[issue]: https://github.com/cookiecutter/cookiecutter/pull/621

## Community

Would you like to contribute to **poyo**? You're awesome! 😃

Please check out the [good first issue][good first issue] label for tasks,
that are good candidates for your first contribution to poyo. Your
contributions are greatly appreciated! Every little bit helps and credit will
always be given.

Everyone interacting in the poyo project's codebases, issue trackers, chat
rooms, and mailing lists is expected to follow the [PyPI Code of
Conduct][code of conduct].

Join the poyo [community][community]! 🌍🌏🌎

[code of conduct]: https://www.pypa.io/en/latest/code-of-conduct/
[community]: https://github.com/hackebrot/poyo/blob/master/COMMUNITY.md
[good first issue]: https://github.com/hackebrot/poyo/labels/good%20first%20issue

## License

Distributed under the terms of the [MIT][MIT] license, poyo is free and open source
software.

[MIT]: https://github.com/hackebrot/poyo/blob/master/LICENSE

[cookiecutter]: https://github.com/cookiecutter/cookiecutter
