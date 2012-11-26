======
Flake8
======

Flake8 is a wrapper around these tools:

- PyFlakes
- pep8
- Ned's McCabe script

Flake8 runs all tools by launching the single 'flake8' script, but ignores pep8
and PyFlakes extended options and just uses defaults. It displays the warnings
in a per-file, merged output.

It also adds a few features:

- files that contains with this header are skipped::

    # flake8: noqa

- lines that contains a "# NOQA" comment at the end will not issue a warning.
- a Mercurial hook.
- a McCabe complexity checker.

QuickStart
==========

To run flake8 just invoke it against any directory or Python module::

    $ flake8 coolproject
    coolproject/mod.py:1027: local variable 'errors' is assigned to but never used
    coolproject/mod.py:97: 'shutil' imported but unused
    coolproject/mod.py:729: redefinition of function 'readlines' from line 723
    coolproject/mod.py:1028: local variable 'errors' is assigned to but never used
    coolproject/mod.py:625:17: E225 missing whitespace around operato

The output of PyFlakes *and* pep8 is merged and returned.

flake8 offers an extra option: --max-complexity, which will emit a warning if the
McCabe complexityu of a function is higher that the value. By default it's
deactivated::

    $ bin/flake8 --max-complexity 12 flake8
    coolproject/mod.py:97: 'shutil' imported but unused
    coolproject/mod.py:729: redefinition of function 'readlines' from line 723
    coolproject/mod.py:1028: local variable 'errors' is assigned to but never used
    coolproject/mod.py:625:17: E225 missing whitespace around operator
    coolproject/mod.py:452:1: 'missing_whitespace_around_operator' is too complex (18)
    coolproject/mod.py:939:1: 'Checker.check_all' is too complex (12)
    coolproject/mod.py:1204:1: 'selftest' is too complex (14)

This feature is quite useful to detect over-complex code. According to McCabe, anything
that goes beyond 10 is too complex.
See https://en.wikipedia.org/wiki/Cyclomatic_complexity.


Mercurial hook
==============

To use the Mercurial hook on any *commit* or *qrefresh*, change your .hg/rc file
like this::

    [hooks]
    commit = python:flake8.run.hg_hook
    qrefresh = python:flake8.run.hg_hook

    [flake8]
    strict = 0
    complexity = 12


If *strict* option is set to **1**, any warning will block the commit. When
*strict* is set to **0**, warnings are just displayed in the standard output.

*complexity* defines the maximum McCabe complexity allowed before a warning
is emited. If you don't specify it it's just ignored. If specified, must
be a positive value. 12 is usually a good value.

Git hook
========

To use the Git hook on any *commit*, add a **pre-commit** file in the
*.git/hooks* directory containing::

    #!/usr/bin/python
    import sys
    from flake8.run import git_hook

    COMPLEXITY = 10
    STRICT = False

    if __name__ == '__main__':
        sys.exit(git_hook(complexity=COMPLEXITY, strict=STRICT, ignore='E501'))


If *strict* option is set to **True**, any warning will block the commit. When
*strict* is set to **False** or omited, warnings are just displayed in the
standard output.

*complexity* defines the maximum McCabe complexity allowed before a warning
is emited. If you don't specify it or set it to **-1**, it's just ignored.
If specified, it must be a positive value. 12 is usually a good value.

*lazy* when set to ``True`` will also take into account files not added to the 
index.

Also, make sure the file is executable and adapt the shebang line so it
point to your python interpreter.


Buildout integration
=====================

In order to use Flake8 inside a buildout, edit your buildout.cfg and add this::

    [buildout]

    parts +=
        ...
        flake8

    [flake8]
    recipe = zc.recipe.egg
    eggs = flake8
           ${buildout:eggs}
    entry-points =
        flake8=flake8.run:main


setuptools integration
======================

If setuptools is available, Flake8 provides a command that checks the
Python files declared by your project. To use it, add flake8 to your
setup_requires::

    setup(
        name="project",
        packages=["project"],

        setup_requires=[
            "flake8"
        ]
    )

Running ``python setup.py flake8`` on the command line will check the
files listed in your ``py_modules`` and ``packages``. If any warnings
are found, the command will exit with an error code::

    $ python setup.py flake8
    


Original projects
=================

Flake8 is just a glue project, all the merits go to the creators of the original
projects:

- pep8: http://github.com/jcrocholl/pep8/
- PyFlakes: http://divmod.org/trac/wiki/DivmodPyflakes
- McCabe: http://nedbatchelder.com/blog/200803/python_code_complexity_microtool.html

Warning / Error codes
=====================

Below are lists of all warning and error codes flake8 will generate, broken
out by component.

pep8:

- E101: indentation contains mixed spaces and tabs
- E111: indentation is not a multiple of four
- E112: expected an indented block
- E113: unexpected indentation
- E201: whitespace after char
- E202: whitespace before char
- E203: whitespace before char
- E211: whitespace before text
- E223: tab / multiple spaces before operator
- E224: tab / multiple spaces after operator
- E225: missing whitespace around operator
- E225: missing whitespace around operator
- E231: missing whitespace after char
- E241: multiple spaces after separator
- E242: tab after separator
- E251: no spaces around keyword / parameter equals
- E262: inline comment should start with '# '
- E301: expected 1 blank line, found 0
- E302: expected 2 blank lines, found <n>
- E303: too many blank lines (<n>)
- E304: blank lines found after function decorator
- E401: multiple imports on one line
- E501: line too long (<n> characters)
- E701: multiple statements on one line (colon)
- E702: multiple statements on one line (semicolon)
- W191: indentation contains tabs
- W291: trailing whitespace
- W292: no newline at end of file
- W293: blank line contains whitespace
- W391: blank line at end of file
- W601: .has_key() is deprecated, use 'in'
- W602: deprecated form of raising exception
- W603: '<>' is deprecated, use '!='
- W604: backticks are deprecated, use 'repr()'

pyflakes:

- W402: <module> imported but unused
- W403: import <module> from line <n> shadowed by loop variable
- W404: 'from <module> import ``*``' used; unable to detect undefined names
- W405: future import(s) <name> after other statements
- W801: redefinition of unused <name> from line <n>
- W802: undefined name <name>
- W803: undefined name <name> in __all__
- W804: local variable <name> (defined in enclosing scope on line <n>) referenced before assignment
- W805: duplicate argument <name> in function definition
- W806: redefinition of function <name> from line <n>
- W806: local variable <name> is assigned to but never used

McCabe:

- W901: '<function_name>' is too complex ('<complexity_level>')

CHANGES
=======

1.6 - 2012-11-16
----------------

- changed the signatures of the ``check_file`` function in flake8/run.py, 
  ``skip_warning`` in flake8/util.py and the ``check``, ``checkPath``
  functions in flake8/pyflakes.py.
- fix ``--exclude`` and ``--ignore`` command flags (#14, #19)
- fix the git hook that wasn't catching files not already added to the index 
  (#29)
- pre-emptively includes the addition to pep8 to ignore certain lines. Add ``# 
  nopep8`` to the end of a line to ignore it. (#37)
- ``check_file`` can now be used without any special prior setup (#21)
- unpacking exceptions will no longer cause an exception (#20)
- fixed crash on non-existant file (#38)



1.5 - 2012-10-13
----------------

- fixed the stdin
- make sure mccabe catches the syntax errors as warnings
- pep8 upgrade
- added max_line_length default value
- added Flake8Command and entry points is setuptools is around
- using the setuptools console wrapper when available


1.4 - 2012-07-12
----------------

- git_hook: Only check staged changes for compliance
- use pep8 1.2


1.3.1 - 2012-05-19
------------------

- fixed support for Python 2.5


1.3 - 2012-03-12
----------------

- fixed false W402 warning on exception blocks.


1.2 - 2012-02-12
----------------

- added a git hook
- now python 3 compatible 
- mccabe and pyflakes have warning codes like pep8 now


1.1 - 2012-02-14
----------------

- fixed the value returned by --version
- allow the flake8: header to be more generic
- fixed the "hg hook raises 'physical lines'" bug
- allow three argument form of raise
- now uses setuptools if available, for 'develop' command

1.0 - 2011-11-29
----------------

- Deactivates by default the complexity checker
- Introduces the complexity option in the HG hook and the command line.


0.9 - 2011-11-09
----------------

- update pep8 version to 0.6.1
- mccabe check: gracefully handle compile failure

0.8 - 2011-02-27
----------------

- fixed hg hook
- discard unexisting files on hook check


0.7 - 2010-02-18
----------------

- Fix pep8 intialization when run through Hg
- Make pep8 short options work when run throug the command line
- skip duplicates when controlling files via Hg


0.6 - 2010-02-15
----------------

- Fix the McCabe metric on some loops


