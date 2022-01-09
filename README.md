# Distributed Package Manager (DPM)

DPM is a light-weight package manager for installing packages and manage their dependencies
from one or more git servers without the need for a central repository.

Install, update, and remove packages and their dependencies across git servers with a single command.
Dependencies are tracked and removed when all packages that dependend on it are removed.

Dependencies a package requires are listed in one text file, `etc/dpm-deps`, in its git repo.
If the package does not have this file and one cannot be added (e.g., you don't have access)
then a meta package can be created that specifies the dependencies of the package.

## Installation

Clone `dpm` then evaluate the output of `dpm setup` from `.bashrc`:

```bash
$ mkdir $HOME/src
$ git -C $HOME/src clone https://github.com/markuskimius/dpm.git
$ echo 'eval "$(bash --norc --noprofile "${HOME}/src/dpm/bin/dpm" setup)"' >> ~/.bashrc
```

Then log out of the terminal session and log back in for the setup script to take effect.

Please note `$HOME/dpm` is a reserved directory (that's where the git packages are cloned)
so do not clone `dpm` in `$HOME`.

Optionally, after the initial install of dpm, dpm can be re-installed using dpm and the original install removed:

```bash
$ dpm install https://github.com/markuskimius/dpm.git
$ echo 'eval "$(bash --norc --noprofile "${HOME}/dpm/dpm/bin/dpm" setup)"' >> ~/.bashrc
$ vim ~/.bashrc    # Remove the old eval line from ~/.bashrc
```

Then log out of the terminal session and log back in for the change to take effect.

## Basic Usage

To install a package from, say, github, use either of these syntaxes:

```bash
$ dpm install https://github.com/markuskimius/common.git
$ dpm install git@github.com:markuskimius/common.git
```

By default, the default branch is installed.
To specify a different branch or a tag, use the `-b` option.

To update:

```bash
$ dpm update common  # Update the package named 'common'
$ dpm update         # Updates all packages
```

To remove:

```bash
$ dpm remove common  # Remove the package named 'common',
                     # and any of its dependencies that aren't used by another package
```

You may need to log out of the terminal session
then log in again after any of the above operations for the change to take effect.

Packages are installed to `$DPM`, set to `$HOME/dpm` by default.
This can be changed by adding the following setting to `$HOME/.dpmrc`:

```bash
PKGDIR=$HOME/path/to/new/package/directory
```

When `dpm setup` is evaluated by `~/.bashrc`, all installed packages' `bin` directory are added to PATH,
`lib` directory added to PYTHONPATH and/or TCLLIBPATH on startup, as appropriate.
Also, if a package provides `etc/bashrc` it is output to be evaluated but only if the package is "active".
More on this later.

To list the installed packages:

```bash
$ dpm list -l
          PKGNAME  NUMREF  HEIGHT  FLAGS  BRANCH           URL
          -------  ------  ------  -----  ------           ---
           common       2       2  AD     main             git@github.com:markuskimius/common.git
              dpm       1       1  D      main             git@github.com:markuskimius/dpm.git
        getopt-sh       2       1         master           git@github.com:markuskimius/getopt-sh.git
       profile-sh       1       3  D      main             git@github.com:markuskimius/profile-sh.git
$
```

Packages with the `D` flag were direct installed by the user.
Those without this flag were installed as a dependency and dependency only.

Packages with the `A` flag are "activated".
An activated package, if it supplies `etc/bashrc`, is sourced at startup
by returning it as a part of `dpm setup` evaluated in `~/.bashrc`.
This activation procedure is a protective measure to allow a user to install a package
and review it before running a script it supplies.
Active packages are also listed in a file, `$DPM/.active`
if any package wants to get the list of packages.

To activate a package, run:
```bash
$ dpm activate common      # common is the package name
```

NUMREF shows the number of packages that depend on this package.
It's possible that a package was direct installed by a user *and* it is a dependency.
A direct installed package with no other package depending on it has the `D` flag and a NUMREF of 1.

HEIGHT shows the *depth* of dependencies a package has.
A package with the height of 1 has no dependencies.
A package with the height of 2 has one or more dependencies whose maximum height is 1.
A package with the height of 3 has one or more dependencies whose maximum height is 2.
This information is used by `dpm setup` to source their startup scripts in order,
so that package A's startup script is sourced after package B's,
if A's startup script requires that B's startup script to have run first.
The height information is also used to generate `$DPM/.active` in the order of lowest to heighest height.

To see the actual dependencies of a package, `dpm deps` can be used.
The following command shows any package that depends on the `getopt-sh` package:

```bash
$ dpm deps getopt-sh
* common
  +-- getopt-sh
* profile-sh
  +-- common
      +-- getopt-sh
```

The number of times the package appears in the output should match its NUMREF.
The maximum depth of dependencies a package has, including itself, should match its HEIGHT.

NUMREF, HEIGHT, and FLAGS of a package are saved in `$DPM/.installed`.
If this running value ever goes out of sync
(which can occur if the package dependencies are manually edited locally)
the file can be regenerated as follows:

```bash
$ dpm recover
```

If this or any other command, except `dpm remove`, detects that a package can be removed
it won't actually remove it but simply show a NUMREF of 0.
To remove all packages with NUMREF of 0, use the following command:

```bash
$ dpm cleanup
```

To keep a package with NUMREF of 0 from being removed, direct-install it:

```
$ dpm install common   # no need to specify URL if the package exists locally
```

## Dependency

A package may include its package dependency information in `etc/dpm-deps`.
This is a pipe-separated text file in the following format:

```
DEPNAME|URL1 BRANCH1 [URL2 BRANCH2] [URL3 BRANCH3] [...]
```

`DEPNAME` is the package name of the dependency.  This name also serves as the
name of the subdirectory under `$DPM` which the package is installed.

`DEPNAME` is installed from `URL1` branch `BRANCH1`.
`BRANCH1` may be `*` to indicate the default branch,
or may be omitted if it is the last item in the field.

If `DEPNAME` is already installed,
the URL and BRANCH of the installed package is tested against all URL and BRANCH listed on the field.
If the installed package's URL and BRANCH is listed,
it is assumed to be a compatible package.
Otherwise it is assumed to be an incompatible package whose name happens to be identical
as the dependency required by the package and will result in an error.

## Meta Package

If a package does not provide the dpm dependency file and one cannot be added,
a meta package can be created.

Simply create an otherwise empty package with one file, `etc/dpm-deps`,
that lists the package and its dependencies.
Installing this meta package will install all required packages.

## License

[GPLv2]


[GPLv2]: <https://github.com/markuskimius/dpm/blob/main/LICENSE>

