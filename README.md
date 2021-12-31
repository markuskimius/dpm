# Distributed Package Manager (DPM)

## What does it do?

- Install packages directly from a git server (such as github).
- Install dependencies automatically, if [specified](#dependency).
- Track which package was installed directly vs. as a dependency,
  and remove the dependencies automatically when the directly installed package
  is uninstalled.


## Installation

Clone dpm then evaluate the output of `dpm setup` from `.bashrc`.
For example:

```bash
$ mkdir $HOME/src
$ git -C $HOME/src clone https://github.com/markuskimius/dpm.git
$ echo 'eval "$(bash --norc --noprofile "${HOME}/src/dpm/bin/dpm" setup)"' >> ~/.bashrc
```

Note that `$HOME/dpm` is a reserved directory; do not clone `dpm` as it.


## Usage

To install a package, use any of these syntaxes:

```bash
$ dpm install https://github.com/markuskimius/common.git
$ dpm install git@github.com:markuskimius/common.git
$ dpm install user@hostname:/path/to/git/repo/.git
```

To update (`git pull`) the package:

```bash
$ dpm update common  # Update the package named 'common'
$ dpm update         # Updates all packages
```

To remove the package:

```bash
$ dpm remove common
```

You may need to exit then re-login to the terminal for the package changes to
take effect in the environment (e.g., update the `$PATH`).


To list what packages are installed:

```bash
$ dpm list -l
         PACKAGE  NUMREF  HEIGHT  FLAGS  BRANCH  URL
  --------------  ------  ------  -----  ------  ----------------------------------------------
          common       1       2  DA     main    https://github.com/markuskimius/common.git
       getopt-sh       1       1         master  https://github.com/markuskimius/getopt-sh.git
$
```

- `PACKAGE` is the name of the local repository.
  It is the name of the physical directory underneath `$DPM`.
- `NUMREF` is the number of times the package is referended by a direct
  installed package.
  This number may be decremented by the `remove` operation.
  The package is not physically removed until this number hits 0.
- `HEIGHT` is how many "levels" of dependencies the package has underneath it,
  plus one.
  If the package has no dependency, the height is 1.
  If the package has 1 dependency, the height of the package is the height of
  its dependency plus 1.
  If the package has many dependencies, the height of the package is the
  maximum height of its dependencies plus one.
  The height information is used to initialize packages in the order of
  dependency, by initializing them from the lowest height to the highest.
- `FLAGS` may be one of:
  - `D` for direct-installed package.
    That is, this package was installed explicitly by the user.
    Note that it is possible for a package to be both directly installed and is
    also installed as a dependency of another package;
    `NUMREF` of 1 means the package is only directly installed;
    `NUMREF` greater than 1 means the package is also a dependency of another
    package.
  - `A` for "activated" package.
    When a package is activated, its `etc/bashrc`, if it exists, is sourced
    upon startup (it is included in the output of `dpm sourceme`).
    Furthermore, the name of the package is added to the list of package names
    in `$DPM/.active`, in the order of lowest height to the highest, that
    may be read by an application to take an action only on activated packages.
    To activate and deactive a package, use the `dpm activate` and `dpm
    deactivate` commands, respectively.
- `BRANCH` is the git branch of the package.
- `URL` is the remote location of the package.

`URL` and `BRANCH` are read directly from the local repo.
.
`NUMREF`, `HEIGHT`, and `FLAGS` are read from `$DPM/.installed`.
This file is updated when packages are installed, updated, or removed.
If a package dependency changes locally, however, this file needs to be
regenerated:

```bash
$ dpm recover
```

After the installation file is regenerated, some packages may have a `NUMREF` of 0.
After confirming, the packages can be removed:

```bash
$ dpm cleanup
```


## Dependency

A package may include its package dependency information in `etc/dpm-deps`.
This is a pipe-separated text file in the following format:

```
DEPNAME|URL1 BRANCH1 [URL2 BRANCH2] [URL3 BRANCH3] [...]
```

`DEPNAME` is the package name of the dependency.  This name also serves as the
name of the subdirectory under `$DPM` which the package is installed.

When `DEPNAME` is installed, it is installed from `URL1` branch `BRANCH1`.  The
remaining URL and branch names, if supplied, are other acceptable sources from
where `DEPNAME` could also be installed.  If any branch from the URL is
acceptable, it may be ommitted (if it is the last branch) or `*` may be used as
its placeholder.  This information is used when installing `DEPNAME` to test,
if the system already has a package named `DEPNAME` installed, whether the
already installed copy is a compatible package or another package that happens
to be named `DEPNAME`.

If you don't have a write access to PKG1 that depends on PKG2, you can create a
metapackage PKG0 that depends on PKG1 and PKG2, and install PKG0.


## Configuration

The value of `$DPM` can be changed by setting `PKGDIR` in one of the following
files:

```
$HOME/.dpmrc
/usr/local/etc/dpmrc
/usr/etc/dpmrc
/etc/dpmrc
```
Only the first file found is loaded.

Example:

```bash
PKGDIR=$HOME/opt
```

The syntax is in BASH, so there can be no whitespace around `=`.


## License

[GPLv2]


[GPLv2]: <https://github.com/markuskimius/dpm/blob/main/LICENSE>

