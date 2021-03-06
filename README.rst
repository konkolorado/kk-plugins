kk-core + kk-plugins
====================

This repository serves as a template for creating extensible, loosely coupled 
CLI programs (based on `Typer`_) leveraging Python `package namespaces`_ (via 
`poetry`_).

What this means is that you can create a base, pip-installable CLI program (in 
this example, the code for the base CLI program is under ``kk-core``) to 
provide the core functionality of the CLI (the CLI created in this example is 
called ``kk``). You can then augment the functionality of the base CLI program 
by pip-installing additional packages (which are designated ``plugins`` in 
this example and the code for the example plugin is in ``kk-plugin1``). The 
core CLI program will dynamically find ``plugins`` in the ``kk`` namespace and 
load them onto itself.

Why is this cool? This lets you, a project, or a company split its CLI tools 
and/or libraries amongst different code repositories, allowing for different 
release cadences. It is also a form of bloat control; clients who require 
functionality beyond the base CLI or library can simply install the additional 
plugin(s) and use augmented features through a familiar interface. Clients who 
only require the base functionality need not concern themselves about the 
additional plugins.


What follows is a motivating example of how this setup behaves in practice. 
The code and libraries used in this repository are live on pypi and can be 
installed to interactively follow along with the examples.

.. note::
   In the code snippets below, lines that begin with ``>>`` indicate an 
   operation's output. 

.. contents:: Table of Contents
   :depth: 2


Example
-------

The Core Functionality
^^^^^^^^^^^^^^^^^^^^^^

First, install the ``kk-core`` library from pypi:

.. code-block:: bash

   pip3 install kk-core

The installation adds a CLI tool called ``kk``. Depending on your shell 
configuration, your shell may not immediately find the command. Start a new 
session or source your shell's ``rc`` file to continue.

.. code-block:: bash

   kk 

   >> Usage: kk [OPTIONS] COMMAND [ARGS]...
   >>
   >> Options:
   >>   --install-completion  Install completion for the current shell.
   >>   --show-completion     Show completion for the current shell, to copy it or
   >>                         customize the installation.
   >>
   >>   --help                Show this message and exit.
   >>
   >> Commands:
   >>   core  This is the core command

As shown in the help text, the only command available from the ``kk-core`` 
library is the ``core`` command. Go ahead and run it:

.. code-block:: bash

   kk core

   >> Usage: kk core [OPTIONS] COMMAND [ARGS]...
   >> 
   >> Options:
   >> --help  Show this message and exit.
   >> 
   >> Commands:
   >> core-command1
   >> core-command2

Again, the help text indicates that the ``kk core`` operation has two 
subcommands, ``core-command1`` and ``core-command2``. Run one of those 
commands:

.. code-block:: bash

   kk core core-command1

   >> core_command1

In this example, the command simply prints its name to stdout. In practice, 
the command would perform some logic.

This module could also be used in a python script:

.. code-block:: python

   from kk import core

   core.app()

   >> Usage:  [OPTIONS] COMMAND [ARGS]...
   >> 
   >> Options:
   >>   --install-completion  Install completion for the current shell.
   >>   --show-completion     Show completion for the current shell, to copy it or
   >>                         customize the installation.
   >> 
   >>   --help                Show this message and exit.
   >> 
   >> Commands:
   >>   core-command1
   >>   core-command2

.. note::
   The ``kk.core`` module in these examples doesn't really provide any 
   interesting functionality. But this doesn't need to be the case for your 
   module.

At this point, we have our base CLI library installed, which gives us access 
to the ``core`` command and the ``core-command1`` and ``core-command2`` 
subcommands. Let's install the command line completion for our shell:

.. code-block:: bash
   
   kk --install-completion

Now, we have the convenience of being able to TAB through the CLI's interface:

.. code-block:: bash
   
   kk [TAB][TAB]

   >> kk core

   kk core [TAB][TAB]
   >> core-command1  core-command2

Depending on your shell configuration, your shell may not find the command 
completion. Start a new session or source your shell's ``rc`` file to 
continue.

The Plugin Functionality
^^^^^^^^^^^^^^^^^^^^^^^^

Suppose the client now needs to augment the functionality of their ``kk`` 
tool. They can do so by installing a plugin:

.. code-block:: bash
   
    pip3 install kk-plugin1

By simply installing the plugin, the base ``kk`` CLI and library has been 
enhanced:

.. code-block:: bash
   
   kk --help

   >> Usage: kk [OPTIONS] COMMAND [ARGS]...
   >> 
   >> Options:
   >>   --install-completion  Install completion for the current shell.
   >>   --show-completion     Show completion for the current shell, to copy it or
   >>                         customize the installation.
   >> 
   >>   --help                Show this message and exit.
   >> 
   >> Commands:
   >>   core     This is the core command
   >>   plugin1  This is plugin1

``kk`` now has access to the ``plugin1`` command, and running that command 
informs us that ``plugin1`` supplies various subcommands:

.. code-block:: bash
   
   kk plugin1

   >> Usage: kk plugin1 [OPTIONS] COMMAND [ARGS]...
   >> 
   >> Options:
   >>   --help  Show this message and exit.
   >> 
   >> Commands:
   >>   plugin1-command1
   >>   plugin1-command2

Perhaps most impressively, after installing ``plugin1``, our TAB completion 
can pick up the completion for the plugin, with no extra effort:

.. code-block:: bash
   
   kk [TAB][TAB]

   >> core     -- This is the core command
   >> plugin1  -- This is plugin1

   kk plugin1 [TAB][TAB]
   >> plugin1-command1  plugin1-command2

When running in a python process, the plugin is available for import:

.. code-block:: python

   from kk import plugin1

   plugin1.app()

   >> Usage:  [OPTIONS] COMMAND [ARGS]...
   >> 
   >> Options:
   >>   --install-completion  Install completion for the current shell.
   >>   --show-completion     Show completion for the current shell, to copy it or
   >>                         customize the installation.
   >> 
   >>   --help                Show this message and exit.
   >> 
   >> Commands:
   >>   plugin1-command1
   >>   plugin1-command2

.. note::
   Again, the ``kk.plugin1`` module in these examples doesn't really provide 
   any interesting functionality. What is interesting is that in scripts, the 
   user can access the augmented functionality through the familiar 
   ``from kk import _`` interface.

Removing Plugins
^^^^^^^^^^^^^^^^

If a plugin's functionality is no longer required, it can be removed and leave 
the base functionality intact:

.. code-block:: bash
   
   pip3 uninstall -y kk-plugin1

   kk --help

   >> Usage: kk [OPTIONS] COMMAND [ARGS]...
   >> 
   >> Options:
   >>   --install-completion  Install completion for the current shell.
   >>   --show-completion     Show completion for the current shell, to copy it or
   >>                         customize the installation.
   >> 
   >>   --help                Show this message and exit.
   >> 
   >> Commands:
   >>   core     This is the core command

And here, impressively again, the CLI's tab completion dynamically picks up on 
the available commands:
 
.. code-block:: bash
   
   kk [TAB][TAB]

   >> core     -- This is the core command

How It Works
------------

There are two things at play that make the example above all work. One may not 
be required, depending on your actual use case. The first is using `poetry`_ 
to create Python `package_namespaces`_ which let us create different Python 
modules that can be imported from the same base package (``kk``). The second 
was using a dynamic python module discovery and loading system to allow us to 
discover plugins and register them into our main package's contents (in this 
example, a `Typer`_ based CLI program).

Package Namespaces
^^^^^^^^^^^^^^^^^^

Read the official documenation on `package namespaces`_ for complete details 
on what they are and how to create them. In the context of this example, we 
create one by:

- Adding the line ``packages = [{include = "kk"}]`` to the ``[tool.poetry]`` 
  section of our core and plugin projects' `pyproject.toml`.
- Following a project structure that tells Python that this package is a 
  namespace package:
 
  core-module-name/
      namespace-name/
          module-name/
              __init__.py
              main.py
      pyproject.toml
  
  plugin-module-name/
      namespace-name/
          module-name/
              __init__.py
              main.py
      pyproject.toml

.. note::

    The salient points of the project structure is that the module's 
    functionality lives in a subdirectory of a directory named after the 
    project's namespace (in our example, the namespace is ``kk``)


Dynamic Package Namespaces Module Discovery
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This point is relevant for our CLI implementation. `Typer`_ expects 
subcommands to be instances of Typer applications, therefore, it is important 
that each of our plugins expose a Typer app instance (which gets registered 
onto the core Typer application) and a name (under which to register the 
plugin's Typer app, effectively becoming the subcommand name). The plugin 
exposes this in its `__init__.py` file.

The core module implements a plugin loading helper module, aptly named 
``plugin_utils.py``. It provides a series of functions that can find modules 
in a given namespace (in our example, the namespace is ``kk``) and import 
them. ``main.py`` uses these functions to identify plugins and pull the 
required Typer attributes (an ``app`` instance and its ``name``) to load the 
plugin into the main CLI application. Finally, the core project defines a CLI 
script (via `poetry`_) that runs ``main.py`` to create the CLI with all 
available plugins loaded. 

.. _Typer: https://typer.tiangolo.com/
.. _poetry: https://python-poetry.org/
.. _package namespaces: https://packaging.python.org/guides/packaging-namespace-packages/
