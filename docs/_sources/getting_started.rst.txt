Getting started
===============

This section is intended to help you through the process of obtaining a working binary by building the source code.
If you just want to launch the game, try the `online version <https://planetincremental.github.io>`_, which is
automatically deployed with every major release.


What you need
-------------

1. A working computer, tablet, phone, or whatever device you want, as long as it can satisfy all the following
   requirements.
2. Git client.
3. C++23 compatible compiler.
4. CMake.
5. A terminal. Not necessarily needed, but this guide assumes that you use one (and are capable of basic tasks
   such as navigating the directory structure, etc.)
6. Ninja build system. Not necessarily needed, but highly recommended. If you do not wish to use Ninja, be
   sure to remove all occurrences of ``-GNinja``, and replace ``ninja`` command with ``make`` in the following
   examples.


Getting the code
----------------

The proper way to obtain the code is to clone the Git repository::

    $ git clone https://github.com/pawel-szajna/planet-incremental.git

After cloning the code, enter the repository directory, and initialize the submodules (which contain most of the
dependencies)::

    $ cd planet-incremental
    $ git submodule update --init --recursive

When this process finishes, you will have a complete copy of the most recent code on your computer. You may use
the command::

    $ git checkout release

to check out the currently released game version code.


Building the code
-----------------

CMake is used in this project to manage the build system. In order to prepare the CMake configuration, and create
the build system scripts, you need to run the CMake program in a designated build directory. It is suggested to use
a directory such as ``build``, and not run CMake in the root project directory.


Launching CMake
...............

In the repository, run::

    $ mkdir build && cd build
    $ cmake .. -GNinja -DCMAKE_BUILD_TYPE=Release -DENABLE_UT=OFF

This will check if your system has all the needed dependencies, and generate the build scripts.

If an error is displayed during this phase, it is most probably related to some missing dependencies. Usually, when
a missing package is reported, googling its name along with the operating system name shows a system package containing
it. For Windows systems, using a C++ package manager like ``vcpkg`` might be the easiest way to install them.


Available CMake options
```````````````````````

If you are not interested in available options and just want to build a working application, feel free to skip this
section.

The ``-GNinja`` option is a CMake instruction to use the Ninja build system generator. By default, on Unix platforms,
Unix Makefiles is used (and the build is then invoked with the ``make`` command, instead of ``ninja``, which is used
in the examples), and on Windows platform, some kind of scary magic such as MSBuild or Visual C++ project files.

The other options are more related to the project:

+---------------------------+------------------------+------------------------------------------------------+
| Option                    | Values                 | Description                                          |
+===========================+========================+======================================================+
| ``-DCMAKE_BUILD_TYPE``    | ``Debug``, ``Release`` | Release builds enable compiler optimizations, while  |
|                           |                        | debug builds include debug symbols, which, as the    |
|                           |                        | name suggests, make debugging easier.                |
+---------------------------+------------------------+------------------------------------------------------+
| ``-DENABLE_UT``           | ``ON``, ``OFF``        | Enables unit tests.                                  |
+---------------------------+------------------------+------------------------------------------------------+
| ``-DENABLE_DOCS``         | ``ON``, ``OFF``        | Enables documentation generation.                    |
+---------------------------+------------------------+------------------------------------------------------+
| ``-DPLATFORM``            | ``Web``, ``Desktop``   | Selects the target platform for use in raylib.       |
+---------------------------+------------------------+------------------------------------------------------+


Building the executable
.......................

After the CMake generation finishes, you can invoke::

    $ ninja

in the build directory, and the building process will start. Ninja automatically detects how many CPU cores are
available, and attempts to utilize them all for compilation. After a while, the process should finish.

If you want to build a specific target, you can specify it with the command::

    $ ninja incremental

The ``incremental`` target is the main game target. Some other targets are mentioned in the sections below.


Running the game
----------------

After the build process finishes, the game file, called ``incremental`` (or ``incremental.exe`` on Windows) in the
``bin`` subdirectory within the build directory. So, after the build finishes, you can type::

    $ ./bin/incremental

to start the freshly built binary file.


Building and executing unit tests
---------------------------------

To enable unit tests, you first need to specify the ``-DENABLE_UT=ON`` option to CMake, then execute ``ninja`` to
build all targets.


Running all tests
.................

The ``test`` target can be used to run all the unit tests::

    $ ninja test

A summary of launched tests will be displayed after the execution finishes.


Running single module tests
...........................

For every module which has unit tests, a binary executable file is created. For instance, one can run::

    $ ./bin/ut/StatisticsUT

to launch all the unit tests from the ``Statistics`` module. The standard Google Test options may be applied, so
for example, to only run the tests from the ``TrackerTests`` test suite, the following command may be used::

    $ ./bin/ut/StatisticsUT --gtest_filter='TrackerTests.*'

Every module UT executable is, of course, a target available, so it may be built without other targets, for instance::

    $ ninja StatisticsUT

builds the executable from the aforementioned examples.


WebAssembly platform
--------------------

The web version of the game needs Emscripten SDK to be built. In order to set up the environment, please refer to the
`Emscripten docs <https://emscripten.org/docs/getting_started/downloads.html>`_. Afterwards, a new build directory
is needed, other than for the desktop target (as the Emscripten SDK wraps CMake and replaces the C++ compiler with
its own C++–WASM, LLVM-based compiler).


Building the WASM binary
........................

In the repository, execute the following commands::

    $ mkdir build_web && cd build_web
    $ emcmake cmake .. -GNinja -DCMAKE_BUILD_TYPE=Release -DENABLE_UT=OFF -DPLATFORM=Web

Of course, you can use any directory instead of ``build_web``, just make sure that you use a different directory
than for the desktop build. Please note that the command ``emcmake cmake`` is not a typo: the ``emcmake`` wrapper
needs the full CMake invocation provided.

.. note::
   Although disabling unit tests build for web platform is recommended, they actually may be compiled and should
   work perfectly well. However, running GTest in a web browser through a ``.html`` file seems... odd.

   If you want, though, there is no one to stop you! (Maybe apart from some symbol export options provided to
   the linker, but if you *really* want to run the unit tests this way...)

Afterward, the compilation itself is not really different::

    $ ninja

The compiler will provide ``.html``, ``.js``, and ``.wasm`` files instead of traditional binaries.


Executing the WASM binary
.........................

Even though it may initially seem obvious, running the compiled web binary may raise some problems. If you just
open the ``.html`` file in your browser, chances are, you will be greeted by an empty page. How come? The safety
mechanisms in the browser may try to prevent you from running some kinds of code.

There are many ways to overcome this problem. The quickest one, at least on the Unix platforms (assuming you have
a Python interpreter at hand), is setting up a simple HTTP server::

    $ cd bin
    $ python3 -m http.server 8080

then navigating to `localhost:8080 <http://localhost:8080>`_ should take you to the, finally, working web version
of the game.


Generating documentation
------------------------

The documentation you are currently reading is generated from RST source files, and Doxygen output from scanning
project C++ headers. The targets ``docs_html``, ``docs_pdf``, and ``docs`` (the last one includes both available
formats) use Doxygen, Sphinx, and Breathe to generate the documentation.

Therefore, to generate the documentation, you will need Doxygen, Sphinx, and Breathe available on your system.
For PDF output, you will need a LaTeX distribution as well.

To enable the documentation targets, use the ``ENABLE_DOCS`` option when invoking CMake::

    $ cmake .. -DENABLE_DOCS=ON -GNinja

Afterward, just invoke the build system with one of the documentations target::

    $ ninja docs_html

The generated documentation will be available in the ``docs`` subdirectory within the build directory.
