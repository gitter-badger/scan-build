[![Build Status](https://travis-ci.org/rizsotto/scan-build.svg?branch=master)](https://travis-ci.org/rizsotto/scan-build)

scan-build
==========

A package designed to wrap a build so that all calls to gcc/clang are
intercepted and logged into a [compilation database][1] and/or piped to
the clang static analyzer. Includes intercept-build tool, which logs
the build, as well as scan-build tool, which logs the build and runs
the clang static analyzer on it.

How to get
----------

It's available from [the Python Package Index][2].

    $ pip install scan-build

Or check out the sources and add the directory `bin` to your `PATH` environment.


Portability
-----------

Should be working on UNIX operating systems.

- It has been tested on FreeBSD, GNU/Linux and OS X.
- Prepared to work on windows, but need help to make it.


Prerequisites
-------------

1. **clang compiler**, to compile the sources and have the static analyzer.
2. **python** interpreter (version 2.7, 3.2, 3.3, 3.4, 3.5).


How to use
----------

To run the Clang static analyzer against a project goes like this:

    $ scan-build <your build command>

To generate a compilation database file goes like this:

    $ intercept-build <your build command>

To run the Clang static analyzer against a project with compilation database
goes like this:

    $ analyze-build

Use `--help` to know more about the commands.


Limitations
-----------

Generally speaking, the `intercept-build` and `analyze-build` tools together
does the same job as `scan-build` does. So, you can expect the same output
from this line as simple `scan-build` would do:

    $ intercept-build <your build command> && analyze-build

The major difference is how and when the analyzer is run. The `scan-build`
tool has three distinct model to run the analyzer:

1.  Use compiler wrappers to make actions.
    The compiler wrappers does run the real compiler and the analyzer.
    This is the default behaviour, can be enforced with `--override-compiler`
    flag.

2.  Use special library to intercept compiler calls durring the build process.
    The analyzer run against each modules after the build finished.
    Use `--intercept-first` flag to get this model.

3.  Use compiler wrappers to intercept compiler calls durring the build process.
    The analyzer run against each modules after the build finished.
    Use `--intercept-first` and `--override-compiler` flags together to get
    this model.

The 1. and 3. are using compiler wrappers, which works only if the build
process respects the `CC` and `CXX` environment variables. (Some build
process can override these variable as command line parameter only. This case
you need to pass the compiler wrappers manually. eg.: `intercept-build
--override-compiler make CC=intercept-cc CXX=intercept-c++ all` where the
original build command would have been `make all` only.)

The 1. runs the analyzer right after the real compilation. So, if the build
process removes removes intermediate modules (generated sources) the analyzer
output still kept.

The 2. and 3. generate the compilation database first, and filters out those
modules which are not exists. So, it's suitable for incremental analysis durring
the development.

The 2. mode is available only on FreeBSD, Linux and OSX. Where library preload
is available from the dynamic loader. Security extension/modes on different
operating systems might disable library preloads. This case the build behaves
normaly, but the result compilation database will be empty. (Notable examples
for enabled security modes are: SIP on OS X Captain and SELinux on Fedora,
RHEL and CentOS.) The program checks the security modes for SIP and SELinux,
and falls back to 3. mode.

`intercept-build` command uses only the 2. and 3. mode to generate the
compilation database. `analyze-build` does only run the analyzer against the
captured compiler calls.


Known problems
--------------

Because it uses `LD_PRELOAD` or `DYLD_INSERT_LIBRARIES` environment variables,
it does not append to it, but overrides it. So builds which are using these
variables might not work. (I don't know any build tool which does that, but
please let me know if you do.)


Problem reports
---------------

If you find a bug in this documentation or elsewhere in the program or would
like to propose an improvement, please use the project's [issue tracker][3].
Please describing the bug and where you found it. If you have a suggestion
how to fix it, include that as well. Patches are also welcome.


License
-------

The project is licensed under University of Illinois/NCSA Open Source License.
See LICENSE.TXT for details.

  [1]: http://clang.llvm.org/docs/JSONCompilationDatabase.html
  [2]: https://pypi.python.org/pypi/scan-build
  [3]: https://github.com/rizsotto/scan-build/issues
