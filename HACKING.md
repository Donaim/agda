Code of conduct
===============

* Follow the Haskell style guide at
  https://github.com/andreasabel/haskell-style-guide/blob/master/haskell-style.md .

* Familiarize yourself with our local toolbox at `src/full/Agda/Utils/*`.

* Write code with verification in mind, documenting invariants and
  pre- and post-conditions.  Testable invariants and algebraic properties
  go to the internal testsuite at `test/Internal`.

* Document (in `haddock` style) the purpose of functions and data structures,
  down to the individual constructor and field.
  An overview over a component or algorithm should be given in the
  `haddock` module comment.

* Remember to document your new feature in `doc/user-manual` and briefly in
  `CHANGELOG.md`. See [Testing and documentation](#testing-and-documentation).

  Excluded are simple bug fixes with a milestone on github and without
  the tag `not-in-changelog`.  These are added upon release by the
  release manager via the tool `src/release-tools/closed-issues-for-milestone`.
  See [Closing issues](#closing-issues).

* Run the full testsuite before pushing patches!

  The recommended way is to run the testsuite on `travis-ci.org`.
  It is automatically started when submitting a pull request.

Note: Some instructions in this document are likely outdated,
so take everything with a grain of salt.
Fixes to outdated instructions welcome!

Working with git
================

Since: 2013-06-15.

Branches
---------

The two main branches of the repository are `master` and `future`. The
master branch contains everything slated for the next release, and the
future branch should be used for more experimental features that need
longer to mature. Any changes on master must be merged into the future
branch (NOTE: currently there are no additional features on future, so
merging is not required or encouraged).

Feature branches should be used generously when fixing bugs and adding
features. Whenever possible, branches should be based on the master
branch.  Even when working on features in the future branch, try to do
as much of the work as possible on master to reduce the number and
severity of merge conflicts.

For instance, fixing issue 1234 would work as follows:

    git checkout master
    git checkout -b issue1234 # create a new branch based on master
    ... work on issue 1234 ...
    git commit -p             # record some patches

    ... working for a long time on issue 1234 ...
    git rebase master          # get fresh upstream patches, keep own work on top
    git commit -p              # record some more patches

    make install-bin test      # ensure compilation and tests

    # Done!  If you have commit rights:

    ## Merge into master
    git checkout master
    git merge issue1234        # merge into master
    make install-bin test      # ensure compilation and tests
    git push

    ## Merge into future  (SKIP THIS STEP FOR NOW)
    git checkout future
    git merge master           # merge master into future
    make install-bin test      # ensure compilation and tests
    git push
    git branch -d issue1234    # delete the branch

    # Otherwise, push branch to your GitHub fork of Agda and create a pull
    # request.
    git push -u myfork issue1234
    Go to https://github.com/agda/agda and click the "New pull request" button
    next to the branch dropdown.

The above procedure has the drawback that with each checkout, many
source files are touched and recompilation is slow.  Here is an
alternative workflow, if you have commit rights and two local
repositories, one on master and one on future (both up-to-date).

    master$  git checkout -b issue1234
    master$  git commit ...
    master$  git checkout master
    master$  git merge issue1234
    master$  make install-bin test
    master$  git push
    master$  git branch -d issue1234

    # Now fast-forward master branch without checking it out.
    # Merge it into future.

    future$ git fetch origin master:master
    future$ git pull
    future$ git merge master
    future$ make install-bin test
    future$ git push

Finding the commit that introduced a regression
-----------------------------------------------

If you want to find the commit that introduced a regression that
caused Module-that-should-be-accepted to be rejected, then you can try
the following recipe:
  ```sh
    git clone <agda repository> agda-bug
    cd agda-bug
    git checkout <suitable branch>
    cabal sandbox init
    git bisect start <bad commit> <good commit>
    cp <some path>/Module-that-should-be-accepted.agda .
    git bisect run sh -c \
      "cabal install --force-reinstalls \
                     --disable-library-profiling \
                     --disable-documentation || exit 125; \
       .cabal-sandbox/bin/agda --ignore-interfaces \
         Module-that-should-be-accepted.agda"
  ```
An alternative is to use the program agda-bisect from
`src/agda-bisect`:
  ```sh
    git clone <agda repository> agda-bug
    cd agda-bug
    cp <some path>/Module-that-should-be-accepted.agda .
    agda-bisect --bad <bad commit> --good <good commit> \
      Module-that-should-be-accepted.agda
  ```
See `agda-bisect --help` for usage information.

The following command temporarily enables Bash completion for
`agda-bisect`:
  ```sh
    source < (agda-bisect --bash-completion-script `which agda-bisect`)
  ```
Bash completion can perhaps be enabled more permanently by storing the
output of the command above in a file in a suitable directory (like
`/etc/bash_completion.d/`).

Standard library submodule
--------------------------

* A large part of the test suite involves the standard library.
  Each version of Agda is deemed compatible with a corresponding version of the
  standard library.

* Each commit in the main Agda repository has a reference to a branch and a
  commit in the standard library repository. The tests are run using this
  referenced version of the standard library.

  + The file `/.gitmodules` contains the URL of the standard library
    repository and the name of the branch.

  + The path `/std-lib` is treated by git as a file containing the hash of the
    referenced commit.

* To obtain the referenced version of the standard library, run `make
  std-lib`.

* To obtain and install the referenced version of the standard
  library, run `make up-to-date-std-lib`.

* To obtain and install  the newest version of the standard library for the
  referenced branch, run `make fast-forward-std-lib`.

  If the new version of the standard library also passes all tests, you can
  have the repository point to it:
  ```sh
      git add std-lib
      git commit
  ```

* The standard library is tracked as a git submodule, which means that the
  `/std-lib` subdirectory will appear as a git repository in a detached-HEAD
  state.

  To avoid this, you may run, inside the submodule directory `git checkout <branch name>`

  and then, from the root directory `git submodule update --remote [--merge|--rebase]`.

  See: https://www.git-scm.com/book/en/v2/Git-Tools-Submodules

Testing and documentation
=========================

* When you implement a new feature it needs to be documented in
  `doc/user-manual/` and `doc/release-notes/<next-version>.txt`.  When
  you fix a bug, drop a note in `CHANGELOG.md`.

* In both cases, you need to add regression tests under `test/Succeed`
  and `test/Fail`, and maybe also `test/interaction`. When adding test
  cases under `test/Fail`, remember to record the error messages
  (`.err` files) after running make test.
  Same for `.warn` files in `test/Succeed` and `.out` files in `test/interaction`.

* Run the test-suite, using `make test`.
  Maybe you want to build Agda first, using `make` or `make install-bin`.

* You can run a single interaction test by going into the
  `test/interaction` directory and typing `make <test name>.cmp`.

* Additional options for the tests using the Haskell/tasty test runner
  can be given using `AGDA_TESTS_OPTIONS`. By default, the interactive
  mode (`-i`) is used and the number of parallel tests to run (`-j`)
  is set to the number of CPU cores.

  You can select certain tests to run by using the `-p` pattern option.
  For example, to only run the simple MAlonzo compiler tests, you
  can use the following command:
  ```sh
    make AGDA_TESTS_OPTIONS="-i -j8 -p MAlonzo.simple" compiler-test
  ```
  You can use the `AGDA_ARGS` environment variable to pass additional
  arguments to Agda when executing the Succeed/Fail/Compiler tests.

* Tests under `test/Fail` can fail if an error message has changed.
  You will be asked whether to accept the new error message.
  Alternatively, you can touch the corresponding source file, since,
  when the test case changes, it is assumed that the error message
  changes as well.

* Tests under `test/Succeed` will also be tested for expected warning
  messages if there is a corresponding `.warn` file. If you want to
  record a new warning, touch the `.warn` file, run `make succeed` and
  accept the new golden value.

* Make sure you do not introduce performance regression.  If you
  ```sh
    make library-test
  ```
  you get a small table with benchmarks at the end. (Due to garbage
  collection, these benchmarks are not 100% stable.)  Compare this
  with benchmarks before the new feature/bug fix.

* You can obtain a simple profile by using `-vprofile:7`. This works
  also in the Emacs mode, output goes to the `*Agda debug*`
  buffer. Note that the `-vprofile:7` option is *not* supposed to be
  given in an OPTIONS pragma, use `agda2-program-args`.

* To avoid problems with the whitespace test failing we suggest add the
  following lines to `.git/hooks/pre-commit`:
  ```sh
      echo "Starting pre-commit"
      make check-whitespace
      if [ $? -ne 0 ]; then
        exit 1
      fi
      echo "Ending pre-commit"
  ```
  You can fix the whitespace issues running
  ```sh
      make fix-whitespace
  ```

* To build the user manual locally, you need to install
  the following dependencies:

    + Python >=3.4.6 from the Travis test.

    + Sphinx and sphinx-rtd-theme

          pip3 install --user -r doc/user-manual/requirements.txt

      Note that the `--user` option puts the Sphinx binaries in
      `$HOME/.local/bin`.

    + LaTeX

  To see the list of available targets, execute `make help`
  in doc/user-manual. E.g., call `make html` to build the
  documentation in html format.

* Running the test-suite using Cabal sandboxes

  If the sandbox uses for example the directory
  `dist/dist-sandbox-12345` you can run the test-suite using the
  following commands:
  ```sh
      export AGDA_BIN=dist/dist-sandbox-12345/build/agda/agda
      export AGDA_TESTS_BIN=dist/dist-sandbox-12345/build/agda-tests/agda-tests
      make test
  ```

* Internal test-suite

  The internal test-suite `test/Internal` is used for testing the Agda library
  (which after closing Issue #2083 doesn't use the QuickCheck library).

  The test-suite uses the same directory structure as the Agda library.

  Internal tests for a module `Agda.Foo.Bar` should reside in module
  `InternalTests.Foo.Bar`.  Same for `Arbitrary` and `CoArbitrary` instances.

Testing with Travis CI and GitHub Actions
=========================================
Since Dec 2019.

Instead of running all test suites locally, it is encouraged that you compile
Agda and run test suites by GitHub Actions and Travis on your own GitHub fork
when hacking Agda.

Different tool chains, compilation flags, and platforms are tested. These tests
are executed in parallel when possible for efficiency, so ideally it also saves
you some time. One caveat:

* *Travis CI* is not active by default. See:
  https://docs.travis-ci.com/user/tutorial/

  for signing up.

You should see the status in your GitHub Actions page and
the Travis dashboard page, if successful.

Some Agda Hacking Lore
======================

* Whenever you change the interface file format you should update
  `Agda.TypeChecking.Serialise.currentInterfaceVersion`.

* Use `__IMPOSSIBLE__` instead of calls to error. `__IMPOSSIBLE__`
  generates errors of the following form:

      An internal error has occurred. Please report this as a bug.
      Location of the error: ...

  Calls to error can make Agda fail with an error message in the
  `*ghci*` buffer.

* GHC warnings are turned on globally in `Agda.cabal`. If you want to
  turn on or off an individual warning in a specific file, use an
  `OPTIONS_GHC` pragma. Don't use `-Wall`, because the meaning of this
  flag can vary between different versions of GHC.

* The GHC documentation (7.10.1) contains the following information
  about orphan instances:

  > GHC identifies orphan modules, and visits the interface file of
  every orphan module below the module being compiled. This is usually
  wasted work, but there is no avoiding it. You should therefore do
  your best to have as few orphan modules as possible.

  See:
  https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/separate-compilation.html#orphan-modules
  .

  In order to avoid *unnecessary* orphan instances the flag
  `-fwarn-orphans` is turned on. If you feel that you really want to use
  an orphan instance, place
  ```haskell
      {-# OPTIONS_GHC -fno-warn-orphans #-}
  ```
  at the top of the module containing the instance.

Haskell-mode and the Agda codebase
==================================

* If you're using a recent haskell-mode (use `M-x package-install
  haskell-mode` to be sure, what's packaged by Debian is not enough),
  and you're editing an Haskell file, you can load it up in by tapping
  `C-c C-l`, and agreeing to Emacs proposals about paths and whatsnot.

* You can toggle from `:load` to `:reload` with `C-u C-c C-l`, which
  you probably want since otherwise you'll load up the world each
  time.

* You have semantic jumps with `M-.`. No more pesky T.A.G.S.!

* You can jump to errors and warnings with `C-x`.  You can probably do
  many other things, Emacs is your oyster.

* One little caveat: GHCi needs some generated files to work.  To make
  sure you have them, you can issue `cabal build` and kill it when it
  starts compiling modules. There doesn't seem to be a programmatic
  way to instruct cabal to do so. They're pretty stable so you don't
  have to do that often.

Emacs mode
==========

* Load times (wall-clock time) can be measured using
  `agda2-measure-load-time`.

* If you fix a bug related to syntax highlighting, please add a test
  case under `test/interaction`. Example `.in` file command:

      IOTCM "Foo.agda" NonInteractive Direct (Cmd_load "Foo.agda" [])

  If you want to include interactive highlighting directives, replace
  `NonInteractive` with `Interactive`.

* The following Elisp code by Nils Anders Danielsson fixes whitespace
  issues upon save.  Add to your `.emacs`.
  ```elisp
      (defvar fix-whitespace-modes
        '(text-mode agda2-mode haskell-mode emacs-lisp-mode LaTeX-mode TeX-mode)
        "*Whitespace issues should be fixed when these modes are used.")

      (add-hook 'before-save-hook
        (lambda nil
          (when (and (member major-mode fix-whitespace-modes)
                     (not buffer-read-only))
            ;; Delete trailing whitespace.
            (delete-trailing-whitespace)
            ;; Insert a final newline character, if necessary.
            (save-excursion
              (save-restriction
                (widen)
                (unless (equal ?\n (char-before (point-max)))
                  (goto-char (point-max))
                  (insert "\n")))))))
  ```

Faster compilation of Agda
==========================

Since: July 2019.

* `make quicker-install-bin` compiles Agda will all optimizations turned off (`-O0`).
  This could be e.g. 5 times as fast (5min instead of 25min).

* Recommended during the development process of a refactoring, new feature or bug fix.
  Not recommended when building Agda for Agda development.
  Unoptimized Agda is slooooow.

* The generated executables have the suffix `-quicker`, e.g., `agda-quicker`.

* In Emacs, activate this version of Agda via
  `M-x agda2-set-program-version RET quicker RET`.

* Running the testsuite requires some tinkering.  E.g., the interactive testsuite
  can be run via `make -C test/interaction AGDA-BIN=agda-quicker`.


Cabal stuff
===========

* For running `cabal repl` use the following command (see
  https://code.google.com/p/agda/issues/detail?id=1196):

      cabal repl --ghc-options=-Wwarn


Developing with Stack
=====================

At the time of writing, the whole dev stack of Agda is still centered around
tools like `Cabal` and `Makefile`.

To develop Agda with `Stack`, copy one of the stack-x.x.x.yaml files of your
choice, and rename it to `stack.yaml`. For example:

    cp stack-8.4.4.yaml stack.yaml

And you are good to go!

You can proceed to build the project and run tests like you would
before:

    make install-bin test

To run `Ghci`:

    stack repl

Closing issues by milestone program
===================================

The `closed-issues-by-milestone` program requires a GitHub personal
access token in the `GITHUBTOKEN` environment variable, i.e,

    export GITHUBTOKEN=your-personal-access-token

The personal access token can be generated from your GitHub user:

    Settings -> Developer settings -> Personal access tokens


Closing issues reported in the CHANGELOG
========================================

Before releasing for example Agda 1.2.3 we add to the `CHANGELOG`
*all* the closed issues with milestone 1.2.3 except those issues
tagged with the labels listed in `labelsNotInChangelog` in the
`src/release-tools/closed-issues-for-milestone/Main.hs` file.

Documentation
=============

See http://agda.readthedocs.io/en/latest/contribute/documentation.html .
