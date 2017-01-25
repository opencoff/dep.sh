=============
README for gg
=============

:Date: Nov 07, 2016

What is ``gg``
==============
``gg`` is an *augmented* wrapper around 'go' toolchain. The
enhancements are:

#. Painless cross-compiling

#. Easy vendordependency management. ``gg`` does **NOT** checkin the
   vendor code into your local repository. This keeps *your*
   repository small/clean.

When run from a top project directory, it implicitly sets ``GOPATH``
to the project directory and its vendor path.

This gives us the following code organization pattern:

- All vendored code goes in ``./vendor/src``
- All local code goes in sub directories of ``./src``
- All local libraries (by convention) go in ``./src/local/``; and imported in code
  as::

    import "local/module"


Installation & Usage
--------------------
``gg`` is a simple bash shell script. It needs a modern (4.x) bash
and ``git``.

You can install it in one of two places (or both):

#. Your personal bin or scripts directory (usually ``$HOME/bin``)

#. Top level directory of your golang project.

General usage help::

    ./gg --help


``gg`` adds the following commands to the tool-chain vocabulary:

* ``fetch``, ``get`` -- fetch and record a new vendor dependency.

* ``update`` -- update one repository from upstream or *all* repositories from
  upstream and update the manifest.

* ``sync`` -- prepare the local directory with the correct checked out version of
  the vendor dependency. This must be run _once_ when a new directory is setup for
  building the entire daemon.

* ``rebuild`` -- Forcibly rebuild the manifest from the contents in
  the ``vendor/src`` directory. This is useful when you have
  carefully pinned the vendor repo versions after diligent
  test/debugging.

* ``list`` -- Show a list of vendored and pinned repositories.

All other commands, it forwards to the 'go' tool. Thus, 'gg' can be used as a
replacement for 'go' for day-to-day use.

Recipe for a New Project
------------------------
If you are a new project and want to use ``gg`` for your vendor
management, the steps are:

Make sure ``gg`` is accessible either in the current directory or
from ``$PATH``. For this example, we will assume that ``gg`` is
installed in the top level directory of your project. Furthermore,
let us call the new project ``projX``.

Then,::

    mkdir -p projx
    cd projx

Now, copy ``gg`` to the top level of ``projx`` directory. The
following commands assume you have already done that.

Now, we initialize the directory::

    ./gg --init
    git commit -m "Initial commit"


Let's now add a new vendor repo to your project. Usually, one does
that by invoking ``go get /path/to/module``. In our case, we do
something similar. e.g., lets add go-lib/logger as a vendor
dependency::

    # Run gg in verbose mode
    ./gg -v get github.com/opencoff/go-lib/logger

    # Lets see what's in the manifest
    cat vendor/manifest.txt

    # Alright. lets commit this into git
    git commit -m "Added new vendor code: opencoff/go-lib"

The "-v" flag to ``gg`` tells it to run in verbose mode.

Updating vendor modules
-----------------------
After working on your tool ``projx`` for a while, you decide that
you need newer versions of all your vendored modules. To get that::

    ./gg update --all

If you only wanted to update *one* vendor repository to its latest
version, then::

    ./gg update REPO

Where ``REPO`` is the name you would use in your golang program.
e.g.,::

    ./gg update github.com/opencoff/go-lib

If your project needs to update *and* pin one specific vendor lib to a
specific version or tag, do::

    ./gg update REPO TAG

Where ``TAG``  is the git version number (short or long) or a git
tag. e.g.,::

    ./gg update github.com/opencoff/go-lib a645eefad878dfce2638e110d2f6e08f61b9faa6


Working on a project as a new contributor
-----------------------------------------
Let's say that you are joining an existing project as a new
contributor. In such a case, the repository you are working on
likely has vendor code pinned by other developers. But, you need to
ready your workspace for building your golang program; i.e.,
checkout the vendor repositories so ``go build`` can find it::

    git clone /path/to/your/repo/projx
    cd projx
    ./gg sync


This command clones the remote (3rd party) repositories in the
``vendor/src`` directory and pins them to the version in the manifest.
Once complete, you build your golang program.


Cross Compiling
===============
To make cross-compiling easy (including for environments that require CGO), ``gg``
supports a command line option to denote the target CPU and OS. For example,
cross-compiling a binary ``foo`` for Android-ARM64::

    ./gg --arch=android-arm64 build -o bin/android-arm64/foo foo

``gg`` knows which environment variables to set before invoking the Go toolchain.

Vendor Management Internals
===========================
Vendor dependencies are recorded in the file ``vendor/manifest.txt``. Each line is
either a comment (starts with '#') or is a dependency record. Each record is a
3-tuple of import-path, upstream-URL, pinned-version.

``gg get`` and ``gg update`` update the manifest. ``gg sync`` consults the
manifest to checkout the correct version.

The checked out vendor code follows the Golang vendor conventions: the code is put
in ``vendor/src``.

Extras
======
The *extras* folder has a small shell script called ``build``. I use this
script for portably building my go programs. This tool does a few important
things for me:

    * it can build one or more binaries
    * it set git version hash to 'main.Version'
    * it can cross-compile one or more binaries

The script is written to build multiple artifacts in the *src/* directory.
The artifacts are set in the ``progs`` variable at the top of the script. You
generally do not have to modify anything else after that line.

And yes, ``build`` uses ``gg`` for its job.

.. vim: ft=rst:sw=4:ts=4:expandtab:tw=78:
