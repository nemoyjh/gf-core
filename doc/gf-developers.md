
## Before you start 

This guide is intended for people who want to contribute to
the development of the GF compiler or the Resource Grammar Library. If
you are a GF user who just wants to download and install GF
(e.g to develop your own grammars), the simpler guide on
[the GF download page](../download/index.html) should be sufficient.

## Setting up your system for building GF 

To build GF from source you need to install some tools on your
system: the Haskell build tool *Stack*, the version control software *Git* and the *Haskeline* library.

### Stack 

The primary installation method is via *Stack*.
(You can also use Cabal, but we recommend Stack to those who are new to Haskell.)

To install Stack:

 * **On Linux and Mac OS**, do either

`$ curl -sSL https://get.haskellstack.org/ | sh`

or

`$ wget -qO- https://get.haskellstack.org/ | sh`

 * **On other operating systems**, see the [installation guide](https://docs.haskellstack.org/en/stable/install_and_upgrade).

### Git 

To get the GF source code, you also need *Git*, a distributed version control system.

 * **On Linux**, the best option is to install the tools via the standard
software distribution channels:

  * On Ubuntu: `sudo apt-get install git-all`
  * On Fedora: `sudo dnf install git-all`

 * **On other operating systems**, see
https://git-scm.com/book/en/v2/Getting-Started-Installing-Git for installation.

### Haskeline 

GF uses *haskeline* to enable command line editing in the GF shell.

 * **On Mac OS and Windows**, this should work automatically.

 * **On Linux**, an extra step is needed to make sure the C libraries (terminfo)
required by *haskeline* are installed:

  * On Ubuntu: `sudo apt-get install libghc-haskeline-dev`
  * On Fedora: `sudo dnf install ghc-haskeline-devel`

## Getting the source 

Once you have all tools in place you can get the GF source code from
[GitHub](https://github.com/GrammaticalFramework/):

 * https://github.com/GrammaticalFramework/gf-core for the GF compiler
 * https://github.com/GrammaticalFramework/gf-rgl for the Resource Grammar Library

### Read-only access: clone the main repository 

If you only want to compile and use GF, you can just clone the repositories as follows:

      $ git clone https://github.com/GrammaticalFramework/gf-core.git
      $ git clone https://github.com/GrammaticalFramework/gf-rgl.git

To get new updates, run the following anywhere in your local copy of the repository:

      $ git pull

### Contribute your changes: fork the main repository 

If you want the possibility to contribute your changes,
you should create your own fork, do your changes there,
and then send a pull request to the main repository.

1. **Creating and cloning a fork  —**
See GitHub documentation for instructions how to [create your own fork](https://docs.github.com/en/get-started/quickstart/fork-a-repo)
of the repository. Once you've done it, clone the fork to your local computer.

      $ git clone https://github.com/<YOUR_USERNAME>/gf-core.git

1. **Updating your copy —**
Once you have cloned your fork, you need to set up the main repository as a remote:

      $ git remote add upstream https://github.com/GrammaticalFramework/gf-core.git

Then you can get the latest updates by running the following:

      $ git pull upstream master

1. **Recording local changes —**
See Git tutorial on how to [record and push your changes](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository) to your fork.

1. **Pull request —**
When you want to contribute your changes to the main gf-core repository,
[create a pull request](https://docs.github.com/en/github/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request)
from your fork.

If you want to contribute to the RGL as well, do the same process for the RGL repository.

## Compilation from source 

By now you should have installed Stack and Haskeline, and cloned the Git repository on your own computer, in a directory called `gf-core`.

### Primary recommendation: use Stack 

Open a terminal, go to the top directory (`gf-core`), and type the following command.

    $ stack install

It will install GF and all necessary tools and libraries to do that.

### Alternative: use Cabal 

You can also install GF using Cabal, if you prefer Cabal to Stack. In that case, you may need to install some prerequisites yourself.

The actual installation process is similar to Stack: open a terminal, go to the top directory (`gf-core`), and type the following command.

    $ cabal install

*The old (potentially outdated) instructions for Cabal are moved to a [separate page](../doc/gf-developers-old-cabal.html). If you run into trouble with `cabal install`, you may want to take a look.*

## Compiling GF with C runtime system support 

The C runtime system is a separate implementation of the PGF runtime services.
It makes it possible to work with very large, ambiguous grammars, using
probabilistic models to obtain probable parses. The C runtime system might
also be easier to use than the Haskell runtime system on certain platforms,
e.g. Android and iOS.

To install the C runtime system, go to the `src/runtime/c` directory.

 * **On Linux and Mac OS —**
You should have autoconf, automake, libtool and make.
If you are missing some of them, follow the
instructions in the [INSTALL](https://github.com/GrammaticalFramework/gf-core/blob/master/src/runtime/c/INSTALL) file.

Once you have the required libraries, the easiest way to install the C runtime is to use the `install.sh` script. Just type

`$ bash install.sh`

This will install the C header files and libraries need to write C programs
that use PGF grammars.

 * **On other operating systems —** Follow the instructions in the
[INSTALL](https://github.com/GrammaticalFramework/gf-core/blob/master/src/runtime/c/INSTALL) file under your operating system.

Depending on what you want to do with the C runtime, you can follow one or more of the following steps.

### Use the C runtime from another programming language 

 * **What —**
This is the most common use case for the C runtime: compile
your GF grammars into PGF with the standard GF executable,
and manipulate the PGFs from another programming language,
using the bindings to the C runtime.

 * **How —**
The Python, Java and Haskell bindings are found in the
`src/runtime/{python,java,haskell-bind}` directories,
respecively. Compile them by following the instructions
in the `INSTALL` or `README` files in those directories.

The Python library can also be installed from PyPI using `pip install pgf`.

*If you are on Mac and get an error about `clang` version, you can try some of [these solutions](https://stackoverflow.com/questions/63972113/big-sur-clang-invalid-version-error-due-to-macosx-deployment-target)—but be careful before removing any existing installations.*

### Use GF shell with C runtime support 

 * **What —**
If you want to use the GF shell with C runtime functionalities, then you need to (re)compile GF with special flags.

The GF shell can be started with `gf -cshell` or `gf -crun` to use
the C run-time system instead of the Haskell run-time system.
Only limited functionality is available when running the shell in these
modes (use the `help` command in the shell for details).

(Re)compiling your GF with these flags will also give you
Haskell bindings to the C runtime, as a library called `PGF2`,
but if you want Python or Java bindings, you need to do [the previous step](#bindings).

 * **How —**
If you use cabal, run the following command:

    cabal install -fc-runtime

from the top directory (`gf-core`).

If you use stack, uncomment the following lines in the `stack.yaml` file:

    flags:
       gf:
         c-runtime: true
    extra-lib-dirs:
       - /usr/local/lib
and then run `stack install` from the top directory (`gf-core`).

*If you get an "`error while loading shared libraries`" when trying to run GF with C runtime, remember to declare your `LD_LIBRARY_PATH`.*
*Add `export LD_LIBRARY_PATH="/usr/local/lib"` to either your `.bashrc` or `.profile`. You should now be able to start GF with C runtime.*

### Use GF server mode with C runtime 

 * **What —**
With this feature, `gf -server` mode is extended with new requests to call the C run-time
system, e.g. `c-parse`, `c-linearize` and `c-translate`.

 * **How —**
If you use cabal, run the following command:

    cabal install -fc-runtime -fserver
from the top directory.

If you use stack, add the following lines in the `stack.yaml` file:

    flags:
       gf:
         c-runtime: true
         server: true
    extra-lib-dirs:
       - /usr/local/lib

and then run `stack install`, also from the top directory.

## Compilation of RGL 

As of 2018-07-26, the RGL is distributed separately from the GF compiler and runtimes.

To get the source, follow the previous instructions on [how to clone a repository with Git](#getting-source).

After cloning the RGL, you should have a directory named `gf-rgl` on your computer.

### Simple 

To install the RGL, you can use the following commands from within the `gf-rgl` repository:

    $ make install

There is also `make build`, `make copy` and `make clean` which do what you might expect.

### Advanced 

For advanced build options, call the Haskell build script directly:

    $ runghc Setup.hs ...

For more details see the [README](https://github.com/GrammaticalFramework/gf-rgl/blob/master/README.md).

### Haskell-free 

If you do not have Haskell installed, you can use the simple build script `Setup.sh`
(or `Setup.bat` for Windows).

## Creating binary distribution packages 

The binaries are generated with Github Actions. More details can be viewed here:

https://github.com/GrammaticalFramework/gf-core/actions/workflows/build-binary-packages.yml

## Running the test suite 

The GF test suite is run with one of the following commands from the top directory:

      $ cabal test

or

      $ stack test

The testsuite architecture for GF is very simple but still very flexible.
GF by itself is an interpreter and could execute commands in batch mode.
This is everything that we need to organize a testsuite. The root of the
testsuite is the `testsuite/` directory. It contains subdirectories
which themselves contain GF batch files (with extension `.gfs`).
The above command searches the subdirectories of the `testsuite/` directory
for files with extension `.gfs` and when it finds one, it is executed with
the GF interpreter. The output of the script is stored in file with extension `.out`
and is compared with the content of the corresponding file with extension `.gold`, if there is one.

Every time when you make some changes to GF that have to be tested,
instead of writing the commands by hand in the GF shell, add them to one `.gfs`
file in the testsuite subdirectory where its `.gf` file resides and run the test.
In this way you can use the same test later and we will be sure that we will not
accidentally break your code later.

**Test Outcome - Passed:** If the contents of the files with the `.out` extension
are identical to their correspondingly-named files with the extension `.gold`,
the command will report that the tests passed successfully, e.g.

     Running 1 test suites...
     Test suite gf-tests: RUNNING...
     Test suite gf-tests: PASS
     1 of 1 test suites (1 of 1 test cases) passed.

**Test Outcome - Failed:**  If there is a contents mismatch between the files
with the `.out` extension and their corresponding files with the extension `.gold`,
the test diagnostics will show a fail and the areas that failed. e.g.

      testsuite/compiler/compute/Records.gfs: OK
      testsuite/compiler/compute/Variants.gfs: FAIL
      testsuite/compiler/params/params.gfs: OK
      Test suite gf-tests: FAIL
      0 of 1 test suites (0 of 1 test cases) passed.

The fail results overview is available in gf-tests.html which shows 4 columns:

1. Results - only areas that fail will appear. (Note: There are 3 failures in the gf-tests.html which are labelled as (expected). These failures should be ignored.)
1. Input - which is the test written in the .gfs file
1. Gold - the expected output from running the test set out in the .gfs file. This column refers to the contents from the .gold extension files.
1. Output - This column refers to the contents from the .out extension files which are generated as test output.
After fixing the areas which fail, rerun the test command. Repeat the entire process of fix-and-test until the test suite passes before  submitting a pull request to include your changes.

