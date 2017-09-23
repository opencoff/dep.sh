=================
README for dep.sh
=================

:Date: Nov 07, 2016

What is ``dep.sh``
==================
``dep.sh`` is a simple vendor management tool for go. It does **NOT** checkin
the vendor code into your repository. This keeps your repository small & clean.

When run from a top project directory, it implicitly sets ``GOPATH``
to the project directory and its vendor path.

``dep.sh`` uses the following code organization pattern:

- All vendored code goes in ``./vendor/src``
- All local code goes in sub directories of ``./src``


Installation & Usage
--------------------
``dep.sh`` is a simple bash shell script. It needs a modern (4.x) bash
and ``git``.

You can install it in one of two places (or both):

#. Your personal bin or scripts directory (usually ``$HOME/bin``)

#. Top level directory of your golang project.

General usage help::

    dep.sh --help


``dep.sh`` adds the following commands to the tool-chain vocabulary:

* ``init`` -- initialize the current directory for vendor dependency
  management. It does NOT commit anything to git.

* ``fetch``, ``get`` -- fetch and record a new vendor dependency.

* ``update`` -- update one repository (optionally to a specific version or
  commit-hash) from upstream or *all* repositories from
  upstream and update the manifest.

* ``ensure`` -- prepare the local directory with the correct checked out version of
  the vendor dependency. This must be run _once_ when a new directory is setup for
  building the entire daemon.

* ``rebuild`` -- Forcibly rebuild the manifest from the contents in
  the ``vendor/src`` directory. This is useful when you have
  carefully pinned the vendor repo versions after diligent
  test/debugging.

* ``status`` -- Show a list of vendored and pinned repositories.

Recipe for a New Project
------------------------
If you are a new project and want to use ``dep.sh`` for your vendor
management, the steps are:

Make sure ``dep.sh`` is accessible either in the current directory or
from ``$PATH``. For this example, we will assume that ``dep.sh`` is
installed in the top level directory of your project. Furthermore,
let us call the new project ``projX``.

Then,::

    mkdir -p projx
    cd projx
    git init

Now, copy ``dep.sh`` to the top level of ``projx`` directory. The
following commands assume you have already done that.

Now, we initialize the directory::

    dep.sh init
    git commit -m "Initial commit"


Let's now add a new vendor repo to your project. Usually, one does
that by invoking ``go get /path/to/module``. In our case, we do
something similar. e.g., lets add ``go-lib/logger`` as a vendor
dependency::

    # Run dep.sh in verbose mode
    dep.sh -v get github.com/opencoff/go-lib/lodep.sher

    # Lets see what's in the manifest
    cat vendor/manifest.txt

    # Alright. lets commit this into git
    git commit -m "Added new vendor code: opencoff/go-lib"

The "-v" flag to ``dep.sh`` tells it to run in verbose mode.

Updating vendor modules
-----------------------
After working on your tool ``projx`` for a while, you decide that
you need newer versions of all your vendored modules. To get that::

    dep.sh update --all

Note that in practice, this is **NOT** a good idea at all. You should always
try each dependency carefully and then commit.

If you only wanted to update *one* vendor repository to its latest
version, then::

    dep.sh update REPO

Where ``REPO`` is the name you would use in your golang program.
e.g.,::

    dep.sh update github.com/opencoff/go-lib

If your project needs to update *and* pin one specific vendor lib to a
specific version or tag, do::

    dep.sh update REPO TAG

Where ``TAG``  is the git version number (short or long) or a git
tag. e.g.,::

    dep.sh update github.com/opencoff/go-lib a645eefad878dfce2638e110d2f6e08f61b9faa6


Working on a project as a new contributor
-----------------------------------------
Let's say that you are joining an existing project as a new
contributor. In this case, the repository you are working on
likely has vendor code pinned by other developers. But, you need to
get your workspace ready for building your golang program; i.e.,
checkout the vendor repositories so ``go build`` can find it::

    git clone /path/to/your/repo/projx
    cd projx
    dep.sh ensure


This command clones the remote (3rd party) repositories in the
``vendor/src`` directory and pins them to the version in the manifest.
Once complete, you build your golang program.


Vendor Management Internals
===========================
Vendor dependencies are recorded in the file ``vendor/manifest.txt``. Each line is
either a comment (starts with '#') or is a dependency record. Each record is a
3-tuple of import-path, pinned-version, upstream-URL.

* ``dep.sh get`` and ``dep.sh update`` update the manifest.

* ``dep.sh ensure`` consults the manifest to checkout the correct version.

The checked out vendor code follows the Golang vendor conventions: the code is put
in ``vendor/src``.

Extras
======
The *extras* folder has a small shell script called ``build``. I use this
script for building my go programs. This tool does a few important
things for me:

    * it can build one or more binaries and sets some variables from the
      environment:

        - main.RepoVersion is set to git repository hash
        - main.ProductVersion is set from build command line option (``--version=``)
        - main.Buildtime is set to UTC build timestamp
    * it can cross-compile one or more binaries
    * it can statically link binaries - as long as CGO is not needed for that
      HOST/OS combination.
    * it can invoke protobuf compiler for generating go source before building
      the programs [only invokes if .proto file is newer than the generated
      file]

The script is written to build multiple artifacts in the *src/* directory.
The artifacts are set in the ``progs`` variable at the top of the script. You
generally do not have to modify anything else after that line.

.. vim: ft=rst:sw=4:ts=4:expandtab:tw=78:
